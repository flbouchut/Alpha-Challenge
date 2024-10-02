# Vault - Tier 1

You are looking through an old version of the OpenZeppelin implementation of ERC-4626 and notice a vulnerability that requires frontrunning an innocent user. You have been granted a large amount of ETH (say e.g. 1k ETH, but you are free to choose the amount :) ) and want to set up a whitehat bot to execute this exploit and return the funds to the user.

-   a) Describe the vulnerability and the payoffs for an attacker.
-   b) Produce code that can check if this vulnerability has occurred in the past and determine how much value was lost, if any.
-   c) Write code for the bot that can carry out the exploit (don’t worry about returning user funds).

### Answer a)

This vulnerability is well know in the crypto security ecosystem and is refered as the **Vault Inflation Attack**. It was first identified on November 15th, 2022, during a [security assessment conducted by OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/audits/2022-10-ERC4626.pdf). Since then, [several mitigation strategies](https://blog.openzeppelin.com/a-novel-defense-against-erc4626-inflation-attacks) have been introduced to protect against this flaw in the ERC-4626 tokenized vault standard.

The vault inflation attack targets ERC-4626 contracts, which are designed to represent **yield-bearing vaults**. In these contracts, shares are minted when users deposit assets, granting them rights to the vault's underlyig assets. As these assets are reinvested in other DeFi protocols, they generate yield, which subsequently increases the value of each share. Users can later redeem their shares when withdrawing from the vault, collecting the accrued yield.

The attack specifically targets **newly deployed vault contracts**, allowing malicious actors to fully or partially steal the initial deposits into the contracts due to a rounding issue in the share calculation.

To execute the attack, an attacker needs to perform two transactions. First, when the vault is deployed, the attacker deposits a small amount into it to ensure he gets **at least 1 share** from the vault. Once the attacker detects a large initial deposit, he frontruns this transaction by depositing the same amount just before the victim's deposit.

The `deposit` function uses the formula `mintedShares = totalSupply() * assets / totalAssets()`. Due to Solidity's integer division rounding down, the victim receives 0 shares. Meanwhile, the attacker can then redeem his share and steal the victim's entire deposit.

A clear example of this attack is documented in the [MixBytes blog post on ERC 4626 inflation attack](https://mixbytes.io/blog/overview-of-the-inflation-attack):

> Attack scenario:
>
> 1. A hacker back-runs the transaction of an ERC4626 pool creation.
> 2. The hacker mints for themself one share: deposit(1). Thus, totalAsset()==1, totalSupply()==1.
> 3. The hacker front-runs the deposit of the victim who wants to deposit 20,000 USDT (20,000.000000).
> 4. The hacker inflates the denominator right in front of the victim: asset.transfer(20_000e6). Now totalAsset()==20_000e6 + 1, totalSupply()==1.
> 5. Next, the victim's tx takes place. The victim gets 1 \* 20_000e6 / (20_000e6 + 1) == 0 shares. The victim gets zero shares.
> 6. The hacker burns their share and gets all the money.

Additionally, there are variations of this attack that enable the attacker to steal only a portion of the victim's deposit or even cause the deposit to be burned.

_Sources_

-   [MixBytes overview of the inflation attack](https://mixbytes.io/blog/overview-of-the-inflation-attack)
-   [OpenZeppelin ERC-4626 Tokenized Vault Security Assessment](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/audits/2022-10-ERC4626.pdf)
-   [OpenZeppelin blog post on mitigation strategies for ERC-4626 inflation attacks](https://blog.openzeppelin.com/a-novel-defense-against-erc4626-inflation-attacks)
-   [OpenZeppelin ERC-4626 docs](https://docs.openzeppelin.com/contracts/5.x/erc4626)
-   [Solidity by Example - Vault inflation](https://solidity-by-example.org/hacks/vault-inflation/)
-   [ERC-4626 Alliance](https://erc4626.info/)
