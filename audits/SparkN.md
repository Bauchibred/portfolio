# Introduction

A time-boxed security review of the **Sparkn** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Sparkn](#About-Sparkn)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://www.codehawks.com/contests/cllcnja1h0001lc08z7w0orxx). The competition spanned **8** days, from **_Aug 21st, 2023_** to **_Aug 29th, 2023_**, and was hosted on [CodeHawks](https://www.codehawks.com). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/Cyfrin/2023-08-sparkn/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash]](https://github.com/Cyfrin/2023-08-sparkn/commit/0f139b2dc53905700dd29a01451b330f829653e9) and primarily covered the following contracts:

- [Distributor.sol](https://github.com/Cyfrin/2023-08-sparkn/blob/main/src/Distributor.sol)
- [Proxy.sol](https://github.com/Cyfrin/2023-08-sparkn/blob/main/src/Proxy.sol)
- [Factory.sol](https://github.com/Cyfrin/2023-08-sparkn/blob/main/src/Factory.sol)

~ 194 NSLOC in 3 contracts

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Sparkn

**Sparkn** is a project that aims to build a marketplace for anyone who wants to solve their problems or anyone who wants to help solve problems, and the intention for now is to get this to be fully on chain, but for the baby stages of the web3 projec.t it's going to be an integration of both web2 & web3.

- Landing Page: https://sparkn.io

# Summary

This report contains **1 high severity** issue, **1 medium** severity issue and **5 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #       | Title                                                                                   | Severity                                         | Status                                                  |
| ------- | --------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [H-01]  | If a winner is blacklisted on any of the tokens they can't receive their funds          | ![](https://img.shields.io/badge/-High-red)      | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01]  | Inability to _fairly_ distribute rewards in the future when SparkN scales up            | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01]  | ProxyFactory imports OpenZeppelin's Ownable.sol which lacks a 2-step ownership transfer | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-02]  | Non-inclusive checks currently break contract's logic in a few instances                | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-03]  | No zero address checks in `proxy.sol`                                                   | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [NC-01] | Currently if organizer signs on a request there are no deadlines for this to expire     | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [NC-02] | Usage of `block.timestamp` should be frowned upon in some cases                         | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [H-01] If a winner is blacklisted on any of the tokens they can't receive their funds

### Summary

Normally this would be a big issue since transfers are done in a loop to all winners _i.e all winners wouldn't be able to get thier tokens_, but winners are chosen off chain and from [the Q&A section of SparkN onboarding video](https://www.youtube.com/watch?v=_VqXB1t9Evo) we can see that after picking a set of winners they can later on be changed, that's the set of winners.

This means that, reasonably, after an attempt to send the tokens to winners has been made and it reverts due to one or a few of the users being in the blacklist/blocklist of USDC/USDT, the set of winners can just be re-chosen without the blacklisted users, now whereas that helps other users from having their funds locked in the contract, this unfairly means that the blacklisted users would lose their earned tokens, since their share must be reshared to other winners to cause [this](https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/Distributor.sol#L134-L137) not to revert

```solidity
        if (totalPercentage != (10000 - COMMISSION_FEE)) {
            revert Distributor__MismatchedPercentages();
        }
```

### Vulnerability Detail

See summary

Additionally note that, the contest readMe's section has indicated that that general stablecoins would be used... _specifically hinting USDC, DAI, USDT & JPYC_,

Now important is to also keep in mind that https://github.com/d-xo/weird-erc20#tokens-with-blocklists shows that:

> Some tokens (e.g. USDC, USDT) have a contract level admin controlled address blocklist. If an address is blocked, then transfers to and from that address are forbidden.

### Impact

Two impacts, depending on how SparkN decides to sort this out,either:

- All winners funds ends up stuck in contract if sparkN doesn't want tomess up the percentages of each winner or
- Some users would have their funds unfairly given to other users

### Recommendation

Consider introducing a functionality that allows winners to specify what address they'd like to be paid, that way even a blocklisted account can specify a different address he/she owns, this case also doesn't really sort this as an attacker could just send any blacklisted address to re-grief the whole thing, so a pull over push method could be done to transfer rewards to winners

#### Additional Note

With this attack window in mind, if a pull method is going to be used then the `_commisionTransfer()` function needs to be refactored to only send the commision.

## [M-01] Inability to _fairly_ distribute rewards in the future when SparkN scales up

### Summary

The comment of the `_distribute()` states that:

>      * The winners and percentages array are supposed not to be so long, so the loop can stay unbounded

But there are no code asurances that this is not the case, see _Vulnerability Details_

### Vulnerability Detail

Take a look at [Distributor.sol#L85-L156](https://github.com/Cyfrin/2023-08-sparkn/blob/0f139b2dc53905700dd29a01451b330f829653e9/src/Distributor.sol#L85-L156)

<details>
  <summary>Click to see code reference</summary>

```solidity
    /**
     * @notice Distribute token to winners according to the percentages
     * @dev Only factory contract can call this function
     * @param token The token address to distribute
     * @param winners The addresses array of winners
     * @param percentages The percentages array of winners
     */
    function distribute(address token, address[] memory winners, uint256[] memory percentages, bytes memory data)
        external
    {
        if (msg.sender != FACTORY_ADDRESS) {
            revert Distributor__OnlyFactoryAddressIsAllowed();
        }
        _distribute(token, winners, percentages, data);
    }

    /**
     * @notice An internal function to distribute JPYC to winners
     * @dev Main logic of distribution is implemented here. The length of winners and percentages must be the same
     * The token address must be one of the whitelisted tokens
     * The winners and percentages array are supposed not to be so long, so the loop can stay unbounded
     * The total percentage must be correct. It must be (100 - COMMITION_FEE).
     * Finally send the remained token(fee) to STADIUM_ADDRESS with no dust in the contract
     * @param token The token address
     * @param winners The addresses of winners
     * @param percentages The percentages of winners
     * @param data The data to be logged. It is supposed to be used for showing the realation bbetween winners and proposals.
     */

    function _distribute(address token, address[] memory winners, uint256[] memory percentages, bytes memory data)
        internal
    {
        // token address input check
        if (token == address(0)) revert Distributor__NoZeroAddress();
        if (!_isWhiteListed(token)) {
            revert Distributor__InvalidTokenAddress();
        }
        // winners and percentages input check
        if (winners.length == 0 || winners.length != percentages.length) revert Distributor__MismatchedArrays();
        uint256 percentagesLength = percentages.length;
        uint256 totalPercentage;
        for (uint256 i; i < percentagesLength;) {
            totalPercentage += percentages[i];
            unchecked {
                ++i;
            }
        }
        // check if totalPercentage is correct
        if (totalPercentage != (10000 - COMMISSION_FEE)) {
            revert Distributor__MismatchedPercentages();
        }
        IERC20 erc20 = IERC20(token);
        uint256 totalAmount = erc20.balanceOf(address(this));

        // if there is no token to distribute, then revert
        if (totalAmount == 0) revert Distributor__NoTokenToDistribute();

        uint256 winnersLength = winners.length; // cache length
        for (uint256 i; i < winnersLength;) {
            uint256 amount = totalAmount * percentages[i] / BASIS_POINTS;
            erc20.safeTransfer(winners[i], amount);
            unchecked {
                ++i;
            }
        }

        // send commission fee as well as all the remaining tokens to STADIUM_ADDRESS to avoid dust remaining
        _commissionTransfer(erc20);
        emit Distributed(token, winners, percentages, data);
    }

```

</details>
Where as at the early stages of the CodeFox's SparkN project one can say this would not be an issue in the future there could be contest with a lot of winners, this is because there is presently no limit on the amount of people that can participate in contests, and in a case either the amount of winners get unfairly reduced cause the loop could get too large
Additionally do note that from [the SparkN's contest onboarding video](https://www.youtube.com/watch?v=_VqXB1t9Evo) the teams have specified that this project could even work with trying to solve issues that are government related, with MarkWu, specifically hinting a question in the lines of _What can the government do to tackle citizens moving from rural areas to the cities_ This seems to be a pretty interesting question but at the same time means that a lot of people could provide interesting ideas and even be "winner worthy" also depending on how big SparkN gets the amount of people ready to participate would massively increas, where as this is not 100% really related, take Code4rena as an example, at the early stages of the protocol, you can even have a contest where ~ 20 people competed, but as of last month Code4rena has over 5000 registered Wardens from across the world, and we can both agree that SparkN would even take a shorter time frame to reach this level of adoption since they are not really in a niche, say smart contract auditing in the case of codeand can have any type of contest

### Impact

Impact I beleive is a high, since it would only take time before this happens, where as this seems impossible as at the early stages of the protocol, once it gets big enough there would complete inability to distribute rewards to winners whenever a contest has a lot of winners

### Recommendation

Refactor the `_distribute()` function to take this into account, i.e create the possibility of calling it by batches,alternatively, one could set a limit for the amount of participant a contest can have to ensure this never happens.... but the latter opotion easily cripples adoption

## [L-01] ProxyFactory imports OpenZeppelin's Ownable.sol which lacks a 2-step ownership transfer

### Summary

The owner is essentially the backbone for SparkN, i.e everything directly/indirectly falls back to be their responsibility from deployment down to cases where if an organizer does not distribute the rewards after the expiration time it's the owner's job to do this, but current implementation puts the whole protocol at a risk when owners are going to be changed.

### Vulnerability Detail

First do note that the `ProxyFactory.sol` contract inherits OpenZeppelin's [Ownable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a5ed318634016a25be4000ee07044a31f363e60c/contracts/access/Ownable.sol#L77C1-L96C6)

<details>
  <summary>Click to see code reference</summary>

```solidity
    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public virtual onlyOwner {
        if (newOwner == address(0)) {
            revert OwnableInvalidOwner(address(0));
        }
        _transferOwnership(newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Internal function without access restriction.
     */
    function _transferOwnership(address newOwner) internal virtual {
        address oldOwner = _owner;
        _owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
```

</details>

As summarized in _Summary_, it's key to note that the onlyOwner is an important modifier which enables certain functions of the contracts (_including the `distributeByOwner()` in the case where organizer doesn't distribute it before the expiration_) to be only executed by the owner.

The above means that we all would agree that the owner of the contract is too important and needs to be handled with optimum care, especially when trying to change the ownership, an extra caution should be applied to the process, cause as at present implementation the wau the ownership is handled the entire protocol would be broken if anything goes wrong since all `onlyOwner` modifier functions will not be usable there after.

One step ownership change could lead to irreversibly setting a wrong and address but 2 step ownership change would allow the new owner to confirm their address to effect the change. This could lead to rendering `onlyOwner` functions inoperable.

### Impact

Impact is High cause asides the complete of halt of protocol when it comes to deployment of new contests, when this happens for any contest an organizer does not distribute the rewards then the funds for winners are completely stuck in the contract

### Recommendation

Consider importing and inheriting the OpenZeppelin's [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) contract instead, since it implements two- step transfers for the owner address

<details>
  <summary>Click to see code reference</summary>

```solidity
 /**
     * @dev Starts the ownership transfer of the contract to a new account. Replaces the pending transfer if there is one.
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public virtual override onlyOwner {
        _pendingOwner = newOwner;
        emit OwnershipTransferStarted(owner(), newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`) and deletes any pending owner.
     * Internal function without access restriction.
     */
    function _transferOwnership(address newOwner) internal virtual override {
        delete _pendingOwner;
        super._transferOwnership(newOwner);
    }

    /**
     * @dev The new owner accepts the ownership transfer.
     */
    function acceptOwnership() public virtual {
        address sender = _msgSender();
        require(pendingOwner() == sender, "Ownable2Step: caller is not the new owner");
        _transferOwnership(sender);
    }
```

</details>

## [L-02] Non-inclusive checks currently break contract's logic in a few instances

### Summary

Checks are used in protocol to set some limitations/deadlines, but some instances lack inclusivity at the ends where as they should be

### Vulnerability Detail

Take the [setContest()]() function as an example

<details>
  <summary>Click to see code reference</summary>

```solidity
    /**
     * @notice Only owner can set contest's properties
    * @notice close time must be less than 28 days from now
     * @dev Set contest close time, implementation address, organizer, contest id
     * @dev only owner can call this function
     * @param organizer The owner of the contest
     * @param contestId The contest id
     * @param closeTime The contest close time
     * @param implementation The implementation address
     */
    function setContest(address organizer, bytes32 contestId, uint256 closeTime, address implementation)
        public
        onlyOwner
    {
        if (organizer == address(0) || implementation == address(0)) revert ProxyFactory__NoZeroAddress();
        if (closeTime > block.timestamp + MAX_CONTEST_PERIOD || closeTime < block.timestamp) {
            revert ProxyFactory__CloseTimeNotInRange();
        }
        bytes32 salt = _calculateSalt(organizer, contestId, implementation);
        if (saltToCloseTime[salt] != 0) revert ProxyFactory__ContestIsAlreadyRegistered();
        saltToCloseTime[salt] = closeTime;
        emit SetContest(organizer, contestId, closeTime, implementation);
    }

    }
```

</details>

As seen it's been explicitly stated that the _close time **must be less than** 28 days from now_ but this is not applied in code, since in the edge case where `close time == 28 days + block.timestamp`, i.e _not less than 28 days_ the line below does not revert

>         if (closeTime > block.timestamp + MAX_CONTEST_PERIOD || closeTime < block.timestamp) { revert ProxyFactory__CloseTimeNotInRange(); }
>
> Also note that the same line does not revert if `closeTime == block.timestamp` which doesn't seem like the right way to operate a contest

Additinonally do note that a few other references still exist in the `ProxyFactory.sol` contract but for brevity reasons only the `setContest()` instance has been discussed in report

### Impact

A break in contract's logic in multiple instances

### Recommendation

Check if a check should be inclusive and make it so if yes, additionally would be nice to introduce a minimum contest period as it doesn't make sense to have a contest just last for say 5 minutes or that has `closeTime == block.timestamp`.

## [L-03] No zero address checks in `proxy.sol`

### Summary

Implementation address could get set to 0

### Vulnerability Detail

Take a look at []()

```
    constructor(address implementation) {
        _implementation = implementation;
    }
```

### Impact

Inaccessibility to delegate any calls to the any fallback calls

### Recommendation

Introduce a zero address check

## [NC-01] Currently if organizer signs on a request there are no deadlines for this to expire

### Summary

The current implementation allows an organizer to sign "someone" to help them submit the request for a contest, but there are currently no integration of deadlines of these signatures

### Vulnerability Detail

See summary, additionally do note that even if `someone` is trusted and would act as organizer suggests, i.e organizer relays to them that they no longer would like to create the request another person can just go ahead and forward the meta tx.

Additionally one would ask that how does another person get a hold of the _meta tx_, but [the SparkN's contest onboarding video](https://www.youtube.com/watch?v=_VqXB1t9Evo) the teams have hinted that the meta transactions are implemented to help non-tech savvy users, what this means is that CodeFox would probably have a specific entity/entities that help with this and someone can just watch them on-chain to see whatever tx they receive

### Impact

Lack of deadlines on signed meta txs means a request is signed for life, which arguably is not the right way to operate

### Recommendation

Refactor code to integrate deadlines/revokes on signatures

## [NC-02] Usage of `block.timestamp` should be frowned upon in some cases

### Summary

Using block.timestamp as part of the time checks could be modified by miners/validators to favor them in contracts that contain logic heavily dependent on them.

### Vulnerability Detail

Consider this problem and warn users that a scenario like this could occur. If the change of timestamps will not affect the protocol in any way, consider documenting the reasoning and writing tests enforcing that these guarantees will be preserved even if the code changes in the future.

### Impact

Low, since, while it's true that miners could influence this, they can only do it for a relatively short amount of time

### Recommendation

Use block.number instead
