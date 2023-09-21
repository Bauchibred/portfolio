# Introduction

A time-boxed security review of the **Escrow** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Escrow](#About-Escrow)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://www.codehawks.com/contests/cljyfxlc40003jq082s0wemya). The competition spanned **12** days, from **_July 24th, 2023_** to **_August 5th, 2023_**, and was hosted on [CodeHawks](https://www.codehawks.com). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/Cyfrin/2023-07-escrow/).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/Cyfrin/2023-07-escrow/commit/65a60eb0773803fa0be4ba72defaec7d8567bccc) and primarily covered the following contracts:

- [Escrow.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol)
- [EscrowFactory.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/EscrowFactory.sol)
- [IEscrow.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/IEscrow.sol)
- [IEscrowFactory.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/IEscrowFactory.sol)

~182 NSLOC in 4 files

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Escrow

The Escrow project is meant to enable smart contract auditors (sellers) and smart contract protocols looking for audits (buyers) to connect using a credibly neutral option, with optional arbitration.

# Summary

This report contains **1 high severity** issue, **1 medium** severity issue and **1 minor** issue found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                                     | Severity                                         | Status                                                  |
| ------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [H-01] | Funds could get stuck in escrow for some tokens                                           | ![](https://img.shields.io/badge/-High-red)      | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01] | Current implementation of the escrow contract means a side step could be done to disputes | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01] | Arbitor fee could be excessively set                                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Disputed-grey)        |

# Findings

## [H-01] Funds could get stuck in escrow for some tokens

### Summary

Current implementation causes funds to be stuck in escrow if any of the three parties get blocklisted

### Vulnerability Details

Note that in the public discord channel for this contest, different discussions have been had on what type of tokens the protocol supports, with sponsor Patrick Collins specifically noting

> Tokens vetted by the team that decides to use them, like USDC, DAI, WETH, etc

Now also note that https://github.com/d-xo/weird-erc20#tokens-with-blocklists shows that:

> Some tokens (e.g. USDC, USDT) have a contract level admin controlled address blocklist. If an address is blocked, then transfers to and from that address are forbidden.

Back to the [Escrow.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol) contract, we can see that the transfer of tokens is been processed in a "send" method and not a withdrawal method, i.e none of the three parties can withdraw by specifying their addresses but rather contract sends them to the hardcoded addresses that was used to deploy the contract.
Whereas the chances are low this means that in a case where gets added to a blocklist by the `tokenContract` used, say `USDC` in our case, this means that even if the buyer calls the [confirmReceipt()](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L94)
function, this [line](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L98) would cause the transaction to revert since an attempt to withdraw the tokens would fail due to it being in the blocklist.

Additionally if either one or more of buyer, seller & arbitor get added into the blocklist, this could even be worse in a case where the `initiateDispute()` function is called, since if there exist an arbitor fee (_which would most likely be the case always_) then this [line](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L123) would cause the transaction to revert, alternatively if the arbitor decides to set the `buyerAward > 0` then this [line](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L123) could easily cause the transaction to revert in the case where buyer is in the blocklist, now note that even if the `buyer` is of good intention and wants to change their mind to confirm receipt instead, the function call wont workdue to the changed state at this [line](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L104)

### Impact

Funds get stucked in the contract

### Recommend Mitigation

Consider using a pull over push pattern, i.e in the case of a withdrawal to seller, a new modifier `onlySeller` should be created and the `confirmReceipt()` should only change the state of the to confirmed then introduce a new function that allows seller to claim the funds only when the state is confirmed and this function should allow sender to specify the address they would like their funds to be sent to.
A similar methodology should be applied while resolving disputes too in the case any other of the three parties get blocklisted.

## [M-01] Current implementation of the escrow contract means a side step could be done to disputes

### Summary

The current Escrow contract implementation allows anyone to be the arbitor even the `0x0` address this in turn permits a scenario where the `buyer` also acts as the `arbiter`. This presents a potential vulnerability where a buyer, acting in bad faith, can initiate a dispute and manipulate its resolution in their favor, undermining the escrow's purpose of ensuring fair transactions.

### Vulnerability Details

In the [Escrow.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol) contract, it is possible for a buyer to set themselves as the arbitor during contract initialization, as suggesterd by this lines [34](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/EscrowFactory.sol#L34) & [62](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/EscrowFactory.sol#L62) in the [EscrowFactory.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/EscrowFactory.sol) contract

As seen the contract does not have an explicit check to prevent the buyer from being the arbiter, _infact the arbitor could be set as the arbitor but thats a different topic_ . Note that this lack of restriction is critical during dispute resolution in the [`resolveDispute(uint256 buyerAward)` ](https://github.com/Cyfrin/2023-07-escrow/blob/65a60eb0773803fa0be4ba72defaec7d8567bccc/src/Escrow.sol#L109) function. The buyer, acting as the arbiter, can award the entire balance to themselves regardless of whether the seller fulfilled their obligations.

So even if a seller did a good job and calls initiateDispute() due to not receiving funds, the arbitor is not a third party so it could aswell mean that the buyer has the overall final decision on how it turns out, which breaks contract's intended logic since the arbitor is suppsoed to be a third party.

### Impact

In the event of a dispute, a buyer acting as an arbitor can award themselves the entire escrow balance, leading to potential financial loss for the seller even if they have every right to a dispute due to doing a perfect job. This vulnerability defeats the purpose of an escrow service and undermines the trustworthiness of the platform.

### Recommended Mitigation

Add an explicit check during contract initialization to prevent the buyer from being the arbiter. Ensure that the buyer and arbitor addresses cannot be the same to prevent potential misuse of dispute resolution. I believe same should be done for the seller address too, i.e `buyer != seller != arbiter`

## [L-01] Arbitor fee could be excessively set

### Summary

The current implementation of the Escrow contract allows an arbiter's fee to be set to even as high as 99% of the price, which seems to be an unnecessarily high percentage allowe.

### Vulnerability Details

In the provided [Escrow.sol](https://github.com/Cyfrin/2023-07-escrow/blob/main/src/Escrow.sol) contract, the `Escrow` constructor includes a check to prevent the arbiter's fee from exceeding the transaction price:

```solidity
if (arbiterFee >= price) revert Escrow__FeeExceedsPrice(price, arbiterFee);
```

However, the problem with this implementation is that it only checks if the arbiter's fee is equal to or greater than the price, but it does not limit the proportion of the price that the arbiter's fee can be. This means that the arbiter's fee can be set to 99% of the price, which could be seen as unreasonably high for an escrow service.

As indicated in the comment with the "@audit" tag:

```solidity
    constructor(
        uint256 price,
        IERC20 tokenContract,
        address buyer,
        address seller,
        address arbiter,
        uint256 arbiterFee
    ) {
// @audit   if (arbiterFee >= price) revert Escrow__FeeExceedsPrice(price, arbiterFee) doesn't make sense if fee is 99% of price, instead a particular amount of fee genereally should be set vor arbiters, say it can't  be more than 50% cause as at present implementation fee can get set to as high as 99%
        if (address(tokenContract) == address(0)) revert Escrow__TokenZeroAddress();
        if (buyer == address(0)) revert Escrow__BuyerZeroAddress();
        if (seller == address(0)) revert Escrow__SellerZeroAddress();
        if (arbiterFee >= price) revert Escrow__FeeExceedsPrice(price, arbiterFee);
        if (tokenContract.balanceOf(address(this)) < price) revert Escrow__MustDeployWithTokenBalance();
        i_price = price;
        i_tokenContract = tokenContract;
        i_buyer = buyer;
        i_seller = seller;
        i_arbiter = arbiter;
        i_arbiterFee = arbiterFee;
    }
```

### Impact

Allowing an excessively high arbiter's fee seems like the wrong logic to use

### Recommended Mitigation

Consider implementing a restriction on the maximum allowable proportion of the price that can be set as the arbiter's fee. For example, a limitation where the arbiter's fee cannot be more than 50% of the price could be deemed more reasonable.
