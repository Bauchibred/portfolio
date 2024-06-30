# Introduction

A time-boxed security review of the **Ajna** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Ajna](#About-Ajna)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://code4rena.com/contests/2023-05-ajna-protocol#top). The competition spanned **8** days, from **_3rd May, 2023_** to **_11th May, 2023_** on [Code4rena](https://code4rena.com/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/code-423n4/2023-05-ajna-findings).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/code-423n4/2023-05-ajna/commit/d80daab705a066828ef1c5d9ba85f315c7c932db) and primarily covered the following contracts:

| File                                                                                                                                                                            |                          SLOC                          |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------------------------------------------------: |
| _Contracts (3)_                                                                                                                                                                 |
| [ajna-grants/src/grants/GrantFund.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/GrantFund.sol)                                               |     [32](#nowhere "(nSLOC:28, SLOC:32, Lines:70)")     |
| [ajna-core/src/PositionManager.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/PositionManager.sol)                                                     |   [186](#nowhere "(nSLOC:165, SLOC:186, Lines:537)")   |
| [ajna-core/src/RewardsManager.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol)                                                       |   [386](#nowhere "(nSLOC:324, SLOC:386, Lines:823)")   |
| _Abstracts (3)_                                                                                                                                                                 |
| [ajna-grants/src/grants/base/Funding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/Funding.sol)                                         |    [66](#nowhere "(nSLOC:48, SLOC:66, Lines:160)")     |
| [ajna-grants/src/grants/base/ExtraordinaryFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/ExtraordinaryFunding.sol)               |   [102](#nowhere "(nSLOC:89, SLOC:102, Lines:309)")    |
| [ajna-grants/src/grants/base/StandardFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/base/StandardFunding.sol)                         |  [372](#nowhere "(nSLOC:307, SLOC:372, Lines:1026)")   |
| _Libraries (1)_                                                                                                                                                                 |
| [ajna-grants/src/grants/libraries/Maths.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol)                                   |     [38](#nowhere "(nSLOC:38, SLOC:38, Lines:58)")     |
| _Interfaces (4)_                                                                                                                                                                |
| [ajna-grants/src/grants/interfaces/IGrantFund.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IGrantFund.sol)                       |     [21](#nowhere "(nSLOC:14, SLOC:21, Lines:66)")     |
| [ajna-grants/src/grants/interfaces/IFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IFunding.sol)                           |     [35](#nowhere "(nSLOC:35, SLOC:35, Lines:96)")     |
| [ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IExtraordinaryFunding.sol) |    [41](#nowhere "(nSLOC:22, SLOC:41, Lines:150)")     |
| [ajna-grants/src/grants/interfaces/IStandardFunding.sol](https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/interfaces/IStandardFunding.sol)           |   [112](#nowhere "(nSLOC:75, SLOC:112, Lines:384)")    |
| Total (over 11 files):                                                                                                                                                          | [1391](#nowhere "(nSLOC:1145, SLOC:1391, Lines:3679)") |

~ 1391 NSLOC in 11 files

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About **Bauchibred**

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About **Ajna**

The Ajna protocol is a non-custodial, peer-to-peer, permissionless lending, borrowing and trading system that requires no governance or external price feeds to function. The protocol consists of pools: pairings of quote tokens provided by lenders and collateral tokens provided by borrowers. Ajna is capable of accepting fungible tokens as quote tokens and both fungible and non-fungible tokens as collateral tokens.

- [Whitepaper](https://www.ajna.finance/)
- [Twitter](https://mobile.twitter.com/ajnafi)
- [Website](https://www.ajna.finance/)

# Summary

This report contains **1 high** issue, **1 medium** severity issue & **2 minor** issues found by **Bauchibred** during the course of the time-boxed audit

| #      | Title                                                                                                                    | Severity                                         | Status                                                  |
| ------ | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------- |
| [H-01] | `stake()` or `updateBucketExchangeRatesAndClaim()` could cost the users their unclaimed rewards                          | ![](https://img.shields.io/badge/-High-d10b0b)   | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01] | RewardsManager.sol: Calculating new rewards is still susceptible to precision loss due to division before multiplication | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01] | Funding.sol: `_validateCallDatas()` undesired behaviour when arrays are too long                                         | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-01] | The current RewardsManager.sol does not implement the `onERC721Received()` function                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [H-01] `stake()` or `updateBucketExchangeRatesAndClaim()` could cost the users their unclaimed rewards

### Impact

While staking or updating buckets exchange rates to claim, senders could lose their rewards, this is due to the implementation of the `_transferAjnaRewards()`

NB: The core cause of this bug is from the internal `_transferAjnaRewards()` function and as such the same issue exists while trying to move staked liquidity via `moveStakedLiquidity()` or claiming rewards via `claimRewards()` since they both call the `_transferAjnaRewards()` function internally.

### Proof of Concept

[`RewardsManager._transferAjnaRewards();`](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L806-L821)

```solidity
 /** @notice Utility method to transfer `Ajna` rewards to the sender
     *  @dev   This method is used to transfer rewards to the `msg.sender` after a successful claim or update.
     *  @dev   It is used to ensure that rewards claimers will be able to claim some portion of the remaining tokens if a claim would exceed the remaining contract balance.
     *  @param rewardsEarned_ Amount of rewards earned by the caller.
     */
    function _transferAjnaRewards(uint256 rewardsEarned_) internal {
        // check that rewards earned isn't greater than remaining balance
        // if remaining balance is greater, set to remaining balance
        uint256 ajnaBalance = IERC20(ajnaToken).balanceOf(address(this));
        if (rewardsEarned_ > ajnaBalance) rewardsEarned_ = ajnaBalance;

        if (rewardsEarned_ != 0) {
            // transfer rewards to sender
            IERC20(ajnaToken).safeTransfer(msg.sender, rewardsEarned_);
        }
    }
```

As seen if at the time of execution the `(rewardsEarned_ > ajnaBalance)` senders only get sent ajnaBalance, note that this function is implemented in both the stake(z) and updateBucketExchangeRatesAndClaim() functions:

- [stake()](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L207-L260)

```solidty
    function stake(
        uint256 tokenId_
    ) external override {
        address ajnaPool = PositionManager(address(positionManager)).poolKey(tokenId_);

        // check that msg.sender is owner of tokenId
        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();

        StakeInfo storage stakeInfo = stakes[tokenId_];
        stakeInfo.owner    = msg.sender;
        stakeInfo.ajnaPool = ajnaPool;

        uint256 curBurnEpoch = IPool(ajnaPool).currentBurnEpoch();

        // record the staking epoch
        stakeInfo.stakingEpoch = uint96(curBurnEpoch);

        // initialize last time interaction at staking epoch
        stakeInfo.lastClaimedEpoch = uint96(curBurnEpoch);

        uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);

        for (uint256 i = 0; i < positionIndexes.length; ) {

            uint256 bucketId = positionIndexes[i];

            BucketState storage bucketState = stakeInfo.snapshot[bucketId];

            // record the number of lps in bucket at the time of staking
            bucketState.lpsAtStakeTime = uint128(positionManager.getLP(
                tokenId_,
                bucketId
            ));
            // record the bucket exchange rate at the time of staking
            bucketState.rateAtStakeTime = uint128(IPool(ajnaPool).bucketExchangeRate(bucketId));

            // iterations are bounded by array length (which is itself bounded), preventing overflow / underflow
            unchecked { ++i; }
        }

        emit Stake(msg.sender, ajnaPool, tokenId_);

        // transfer LP NFT to this contract
        IERC721(address(positionManager)).transferFrom(msg.sender, address(this), tokenId_);

        // calculate rewards for updating exchange rates, if any
        uint256 updateReward = _updateBucketExchangeRates(
            ajnaPool,
            positionIndexes
        );

        // transfer rewards to sender
        _transferAjnaRewards(updateReward);
    }
```

[updateBucketExchangeRatesAndClaim()](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L310C14-L318)

```solidty
       function updateBucketExchangeRatesAndClaim(
        address pool_,
        uint256[] calldata indexes_
    ) external override returns (uint256 updateReward) {
        updateReward = _updateBucketExchangeRates(pool_, indexes_);

        // transfer rewards to sender
        _transferAjnaRewards(updateReward);
    }
```

### Recommendation

Multiple ideas, but one should be allowing the user partially claim the reward when the smart contract has insufficient AJNA token balance to distribute the staking reward.

Or if the balance is insufficient, the transfer should be prevented, This will ensure that the user can receive their earned rewards. this might affect the case of normal flowing of staking or unstaking so consider adding a "force" parameter to allow users to bypass restrictions

## [M-01] RewardsManager.sol: Calculating new rewards is still susceptible to precision loss due to division before multiplication

### Impact

Stakers may not receive rewards due to precision loss.

### Proof of Concept

The `RewardsManager._calculateNewRewards()` function calculates the new rewards for a staker by dividing the product of `interestEarned_ and  totalBurnedInPeriod` by the `totalInterestEarnedInPeriod` and then multiplying by `REWARD_FACTOR` If the product of `interestEarned_ and  totalBurnedInPeriod`is small enough and `totalInterestEarnedInPeriod` is large enough, the division may result in a value of 0, resulting in the staker receiving 0 rewards.

[RewardsManager.\_calculateNewRewards()](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-core/src/RewardsManager.sol#L510-L551)

```solidty
    /**
     *  @notice Calculate new rewards between current and next epoch, based on earned interest.
     *  @param  ajnaPool_              Address of the pool.
     *  @param  interestEarned_        The amount of interest accrued to current epoch.
     *  @param  nextEpoch_             The next burn event epoch to calculate new rewards.
     *  @param  epoch_                 The current burn event epoch to calculate new rewards.
     *  @param  rewardsClaimedInEpoch_ Rewards claimed in epoch.
     *  @return newRewards_            New rewards between current and next burn event epoch.
     */
    function _calculateNewRewards(
        address ajnaPool_,
        uint256 interestEarned_,
        uint256 nextEpoch_,
        uint256 epoch_,
        uint256 rewardsClaimedInEpoch_
    ) internal view returns (uint256 newRewards_) {
        (
            ,
            // total interest accumulated by the pool over the claim period
            uint256 totalBurnedInPeriod,
            // total tokens burned over the claim period
            uint256 totalInterestEarnedInPeriod
        ) = _getPoolAccumulators(ajnaPool_, nextEpoch_, epoch_);

        // calculate rewards earned
        newRewards_ = totalInterestEarnedInPeriod == 0 ? 0 : Maths.wmul(
            REWARD_FACTOR,
            Maths.wdiv(
                Maths.wmul(interestEarned_, totalBurnedInPeriod),
                totalInterestEarnedInPeriod
            )//@audit M precision loss still possible, Maths.wdiv is not the outermost bracket
        );

        uint256 rewardsCapped = Maths.wmul(REWARD_CAP, totalBurnedInPeriod);

        // Check rewards claimed - check that less than 80% of the tokens for a given burn event have been claimed.
        if (rewardsClaimedInEpoch_ + newRewards_ > rewardsCapped) {

            // set claim reward to difference between cap and reward
            newRewards_ = rewardsCapped - rewardsClaimedInEpoch_;
        }
    }
```

### Tool used

Manual Review

### Recommendation

Consider calculating the new rewards by first doing all the multiplication i.e multiplying interestEarned\_ by totalBurnedInPeriod and then by `REWARD_FACTOR` before dividing by `totalInterestEarnedInPeriod` to avoid precision loss.

## [L-01] Funding.sol: `_validateCallDatas()` undesired behaviour when arrays are too long

### Impact

Could cause brickage due to out of gas errors if the arrays get too long

### Proof of Concept

[Funding.\_validateCallDatas():](https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L95-L141)

```solidty
    function _validateCallDatas(
        address[] memory targets_,
        uint256[] memory values_,
        bytes[] memory calldatas_
    ) internal view returns (uint128 tokensRequested_) {

        // check params have matching lengths
        if (targets_.length == 0 || targets_.length != values_.length || targets_.length != calldatas_.length) revert InvalidProposal();

        for (uint256 i = 0; i < targets_.length;) {

            // check targets and values params are valid
            if (targets_[i] != ajnaTokenAddress || values_[i] != 0) revert InvalidProposal();

            // check calldata function selector is transfer()
            bytes memory selDataWithSig = calldatas_[i];

            bytes4 selector;
            //slither-disable-next-line assembly
            assembly {
                selector := mload(add(selDataWithSig, 0x20))
            }
            if (selector != bytes4(0xa9059cbb)) revert InvalidProposal();

            // https://github.com/ethereum/solidity/issues/9439
            // retrieve tokensRequested from incoming calldata, accounting for selector and recipient address
            uint256 tokensRequested;
            bytes memory tokenDataWithSig = calldatas_[i];
            //slither-disable-next-line assembly
            assembly {
                tokensRequested := mload(add(tokenDataWithSig, 68))
            }

            // update tokens requested for additional calldata
            tokensRequested_ += SafeCast.toUint128(tokensRequested);

            unchecked { ++i; }
        }
    }

```

### Recommendation

A limit should be provided and the provided array lengths shouldn't be higher than this in order to curb the possibility of the execution's brickage

## [L-02] The current RewardsManager.sol does not implement the `onERC721Received()` function

### Impact

The former code base had the onERC721Received function implemented, but with the new update this is no longer the case:
Below are links to both old and new code bases for RewardsManager.sol:

- https://github.com/sherlock-audit/2023-01-ajna/blob/main/contracts/src/RewardsManager.sol

- https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-core/src/RewardsManager.sol

As seen the latter of the two which is the current code base lacks the function.

The absence of this function could prevent RewardsManager from receiving ERC721 tokens from other contracts via `safeTransferFrom()`.

### Proof of Concept

Check both old and new code bases of RewardsManager, links attached in **Impact**

### Recommendation

Consider adding an implementation of the `onERC721Received` function in the current codebase of `RewardsManager`
