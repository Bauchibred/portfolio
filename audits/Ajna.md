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

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/32), which took place over a span of **12** days from **_22nd May, 2023_** to **_3rd June, 2023_** on [Sherlock](https://sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-01-ajna-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](<(https://github.com/ajna-finance/ajna-core/tree/e3632f6d0b196fb1bf1e59c05fb85daf357f2386)>) and primarily covered the following contracts:

- [ajna-core/src/ERC20Pool.sol](ajna-core/src/ERC20Pool.sol)
- [ajna-core/src/ERC20PoolFactory.sol](ajna-core/src/ERC20PoolFactory.sol)
- [ajna-core/src/ERC721Pool.sol](ajna-core/src/ERC721Pool.sol)
- [ajna-core/src/ERC721PoolFactory.sol](ajna-core/src/ERC721PoolFactory.sol)
- [ajna-core/src/PoolInfoUtils.sol](ajna-core/src/PoolInfoUtils.sol)
- [ajna-core/src/PositionManager.sol](ajna-core/src/PositionManager.sol)
- [ajna-core/src/RewardsManager.sol](ajna-core/src/RewardsManager.sol)
- [ajna-core/src/base/FlashloanablePool.sol](ajna-core/src/base/FlashloanablePool.sol)
- [ajna-core/src/base/PermitERC721.sol](ajna-core/src/base/PermitERC721.sol)
- [ajna-core/src/base/Pool.sol](ajna-core/src/base/Pool.sol)
- [ajna-core/src/base/PoolDeployer.sol](ajna-core/src/base/PoolDeployer.sol)
- [ajna-core/src/interfaces/pool/IERC3156FlashBorrower.sol](ajna-core/src/interfaces/pool/IERC3156FlashBorrower.sol)
- [ajna-core/src/interfaces/pool/IERC3156FlashLender.sol](ajna-core/src/interfaces/pool/IERC3156FlashLender.sol)
- [ajna-core/src/interfaces/pool/IPool.sol](ajna-core/src/interfaces/pool/IPool.sol)
- [ajna-core/src/interfaces/pool/IPoolFactory.sol](ajna-core/src/interfaces/pool/IPoolFactory.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolBorrowerActions.sol](ajna-core/src/interfaces/pool/commons/IPoolBorrowerActions.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolDerivedState.sol](ajna-core/src/interfaces/pool/commons/IPoolDerivedState.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolErrors.sol](ajna-core/src/interfaces/pool/commons/IPoolErrors.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolEvents.sol](ajna-core/src/interfaces/pool/commons/IPoolEvents.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolImmutables.sol](ajna-core/src/interfaces/pool/commons/IPoolImmutables.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolInternals.sol](ajna-core/src/interfaces/pool/commons/IPoolInternals.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolKickerActions.sol](ajna-core/src/interfaces/pool/commons/IPoolKickerActions.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolLPActions.sol](ajna-core/src/interfaces/pool/commons/IPoolLPActions.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolLenderActions.sol](ajna-core/src/interfaces/pool/commons/IPoolLenderActions.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolSettlerActions.sol](ajna-core/src/interfaces/pool/commons/IPoolSettlerActions.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolState.sol](ajna-core/src/interfaces/pool/commons/IPoolState.sol)
- [ajna-core/src/interfaces/pool/commons/IPoolTakerActions.sol](ajna-core/src/interfaces/pool/commons/IPoolTakerActions.sol)
- [ajna-core/src/interfaces/pool/erc20/IERC20Pool.sol](ajna-core/src/interfaces/pool/erc20/IERC20Pool.sol)
- [ajna-core/src/interfaces/pool/erc20/IERC20PoolEvents.sol](ajna-core/src/interfaces/pool/erc20/IERC20PoolEvents.sol)
- [ajna-core/src/interfaces/pool/erc20/IERC20PoolBorrowerActions.sol](ajna-core/src/interfaces/pool/erc20/IERC20PoolBorrowerActions.sol)
- [ajna-core/src/interfaces/pool/erc20/IERC20PoolFactory.sol](ajna-core/src/interfaces/pool/erc20/IERC20PoolFactory.sol)
- [ajna-core/src/interfaces/pool/erc20/IERC20PoolLenderActions.sol](ajna-core/src/interfaces/pool/erc20/IERC20PoolLenderActions.sol)
- [ajna-core/src/interfaces/pool/erc20/IERC20PoolImmutables.sol](ajna-core/src/interfaces/pool/erc20/IERC20PoolImmutables.sol)
- [ajna-core/src/interfaces/pool/erc20/IERC20Taker.sol](ajna-core/src/interfaces/pool/erc20/IERC20Taker.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721Pool.sol](ajna-core/src/interfaces/pool/erc721/IERC721Pool.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721PoolBorrowerActions.sol](ajna-core/src/interfaces/pool/erc721/IERC721PoolBorrowerActions.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721PoolErrors.sol](ajna-core/src/interfaces/pool/erc721/IERC721PoolErrors.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721PoolEvents.sol](ajna-core/src/interfaces/pool/erc721/IERC721PoolEvents.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721PoolFactory.sol](ajna-core/src/interfaces/pool/erc721/IERC721PoolFactory.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721PoolImmutables.sol](ajna-core/src/interfaces/pool/erc721/IERC721PoolImmutables.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721PoolLenderActions.sol](ajna-core/src/interfaces/pool/erc721/IERC721PoolLenderActions.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721PoolState.sol](ajna-core/src/interfaces/pool/erc721/IERC721PoolState.sol)
- [ajna-core/src/interfaces/pool/erc721/IERC721Taker.sol](ajna-core/src/interfaces/pool/erc721/IERC721Taker.sol)
- [ajna-core/src/interfaces/position/IPositionManager.sol](ajna-core/src/interfaces/position/IPositionManager.sol)
- [ajna-core/src/interfaces/position/IPositionManagerDerivedState.sol](ajna-core/src/interfaces/position/IPositionManagerDerivedState.sol)
- [ajna-core/src/interfaces/position/IPositionManagerEvents.sol](ajna-core/src/interfaces/position/IPositionManagerEvents.sol)
- [ajna-core/src/interfaces/position/IPositionManagerErrors.sol](ajna-core/src/interfaces/position/IPositionManagerErrors.sol)
- [ajna-core/src/interfaces/position/IPositionManagerOwnerActions.sol](ajna-core/src/interfaces/position/IPositionManagerOwnerActions.sol)
- [ajna-core/src/interfaces/position/IPositionManagerState.sol](ajna-core/src/interfaces/position/IPositionManagerState.sol)
- [ajna-core/src/interfaces/rewards/IRewardsManager.sol](ajna-core/src/interfaces/rewards/IRewardsManager.sol)
- [ajna-core/src/interfaces/rewards/IRewardsManagerDerivedState.sol](ajna-core/src/interfaces/rewards/IRewardsManagerDerivedState.sol)
- [ajna-core/src/interfaces/rewards/IRewardsManagerErrors.sol](ajna-core/src/interfaces/rewards/IRewardsManagerErrors.sol)
- [ajna-core/src/interfaces/rewards/IRewardsManagerEvents.sol](ajna-core/src/interfaces/rewards/IRewardsManagerEvents.sol)
- [ajna-core/src/interfaces/rewards/IRewardsManagerOwnerActions.sol](ajna-core/src/interfaces/rewards/IRewardsManagerOwnerActions.sol)
- [ajna-core/src/interfaces/rewards/IRewardsManagerState.sol](ajna-core/src/interfaces/rewards/IRewardsManagerState.sol)
- [ajna-core/src/libraries/external/BorrowerActions.sol](ajna-core/src/libraries/external/BorrowerActions.sol)
- [ajna-core/src/libraries/external/KickerActions.sol](ajna-core/src/libraries/external/KickerActions.sol)
- [ajna-core/src/libraries/external/LPActions.sol](ajna-core/src/libraries/external/LPActions.sol)
- [ajna-core/src/libraries/external/LenderActions.sol](ajna-core/src/libraries/external/LenderActions.sol)
- [ajna-core/src/libraries/external/PoolCommons.sol](ajna-core/src/libraries/external/PoolCommons.sol)
- [ajna-core/src/libraries/external/PositionNFTSVG.sol](ajna-core/src/libraries/external/PositionNFTSVG.sol)
- [ajna-core/src/libraries/external/SettlerActions.sol](ajna-core/src/libraries/external/SettlerActions.sol)
- [ajna-core/src/libraries/external/TakerActions.sol](ajna-core/src/libraries/external/TakerActions.sol)
- [ajna-core/src/libraries/helpers/RevertsHelper.sol](ajna-core/src/libraries/helpers/RevertsHelper.sol)
- [ajna-core/src/libraries/helpers/PoolHelper.sol](ajna-core/src/libraries/helpers/PoolHelper.sol)
- [ajna-core/src/libraries/helpers/SafeTokenNamer.sol](ajna-core/src/libraries/helpers/SafeTokenNamer.sol)
- [ajna-core/src/libraries/internal/Buckets.sol](ajna-core/src/libraries/internal/Buckets.sol)
- [ajna-core/src/libraries/internal/Deposits.sol](ajna-core/src/libraries/internal/Deposits.sol)
- [ajna-core/src/libraries/internal/Loans.sol](ajna-core/src/libraries/internal/Loans.sol)
- [ajna-core/src/libraries/internal/Maths.sol](ajna-core/src/libraries/internal/Maths.sol)

[ajna-grants @ 65d52ce52039577b1cfefc76cbbf0030a87f4845](https://github.com/ajna-finance/ajna-grants/tree/65d52ce52039577b1cfefc76cbbf0030a87f4845)

- [ajna-grants/src/grants/GrantFund.sol](ajna-grants/src/grants/GrantFund.sol)
- [ajna-grants/src/grants/base/Storage.sol](ajna-grants/src/grants/base/Storage.sol)
- [ajna-grants/src/grants/interfaces/IGrantFund.sol](ajna-grants/src/grants/interfaces/IGrantFund.sol)
- [ajna-grants/src/grants/interfaces/IGrantFundActions.sol](ajna-grants/src/grants/interfaces/IGrantFundActions.sol)
- [ajna-grants/src/grants/interfaces/IGrantFundErrors.sol](ajna-grants/src/grants/interfaces/IGrantFundErrors.sol)
- [ajna-grants/src/grants/interfaces/IGrantFundEvents.sol](ajna-grants/src/grants/interfaces/IGrantFundEvents.sol)
- [ajna-grants/src/grants/interfaces/IGrantFundState.sol](ajna-grants/src/grants/interfaces/IGrantFundState.sol)
- [ajna-grants/src/grants/libraries/Maths.sol](ajna-grants/src/grants/libraries/Maths.sol)
- [ajna-grants/src/token/AjnaToken.sol](ajna-grants/src/token/AjnaToken.sol)
- [ajna-grants/src/token/BurnWrapper.sol](ajna-grants/src/token/BurnWrapper.sol)

~ 2229 NSLOC in 11 files

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Ajna

The Ajna protocol is a non-custodial, peer-to-peer, permissionless lending, borrowing and trading system that requires no governance or external price feeds to function. The protocol consists of pools: pairings of quote tokens provided by lenders and collateral tokens provided by borrowers. Ajna is capable of accepting fungible tokens as quote tokens and both fungible and non-fungible tokens as collateral tokens.

- [Whitepaper](https://www.ajna.finance/)
- [Twitter](https://mobile.twitter.com/ajnafi)
- [Website](https://www.ajna.finance/)

# Summary

This report contains **1 critical** issue, **1 high severity** issue and **3 medium** severity issues, and **1 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                                                           | Severity                                           | Status                                                  |
| ------ | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------- |
| [C-01] | Loss of Unclaimed RewardsDue to Position Zeroing Out                                                            | ![](https://img.shields.io/badge/-Critical-d10b0b) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [H-01] | Lack of Frontrun Protection in the `updateBucketExchangeRatesAndClaim` Function Could Disadvantage Honest Users | ![](https://img.shields.io/badge/-HIgh-red)        | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [M-01] | Hard-coded Slippage Value in Unstaking Function Can Result in Denial-of-Service                                 | ![](https://img.shields.io/badge/-Medium-orange)   | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [M-02] | PositionManager & PermitERC721 do not comply with EIP-4494                                                      | ![](https://img.shields.io/badge/-Medium-orange)   | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03] | Potential Loss of Bucket Rewards due to Missing Slippage Control in the Staking Function                        | ![](https://img.shields.io/badge/-Medium-orange)   | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-01] | Adopting the EIP-4494 Standard, Currently in its Draft Stage, May Lead to Potential Compatibility Challenges    | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Acknowledged-black)   |

## [C-01] Loss of Unclaimed Rewards Due to Position Zeroing Out

### Summary

A potential loss of unclaimed rewards when a position is zeroed out due to a bankrupt bucket within the `PositionManager.memorializePositions` method. In the current implementation, a lender's unclaimed rewards might be forfeited due to the lack of checking and claiming the rewards before zeroing out the LP balance in case of bankruptcy.

### Vulnerability Detail

In the `memorializePositions` method of the `PositionManager` contract, the contract records bucket indexes along with their deposit times and LP balances. This method also transfers LP ownership from the lender to the `PositionManager` contract. During this operation, the contract checks if there was a previous deposit and whether the bucket went bankrupt after that deposit.
Take a look at the [memorializePositions()](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/PositionManager.sol#L197-L255)

```solidity
    function memorializePositions(
        address pool_,
        uint256 tokenId_,
        uint256[] calldata indexes_
    ) external mayInteract(pool_, tokenId_) override {
        TokenInfo storage tokenInfo = positionTokens[tokenId_];
        EnumerableSet.UintSet storage positionIndexes = tokenInfo.positionIndexes;

        IPool   pool  = IPool(pool_);
        address owner = ownerOf(tokenId_);

        LendersBucketLocalVars memory vars;

        // local vars used in for loop for reduced gas
        uint256 index;
        uint256 indexesLength = indexes_.length;

        // loop through all bucket indexes and memorialize lp balance and deposit time to the Position.
        for (uint256 i = 0; i < indexesLength; ) {
            index = indexes_[i];

            // record bucket index at which a position has added liquidity
            // slither-disable-next-line unused-return
            positionIndexes.add(index);

            (vars.lpBalance, vars.depositTime) = pool.lenderInfo(index, owner);

            // check that specified allowance is at least equal to the lp balance
            vars.allowance = pool.lpAllowance(index, address(this), owner);

            if (vars.allowance < vars.lpBalance) revert AllowanceTooLow();

            Position memory position = tokenInfo.positions[index];

            // check for previous deposits
            if (position.depositTime != 0) {
                // check that bucket didn't go bankrupt after prior memorialization
                if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) {
                    // if bucket did go bankrupt, zero out the LP tracked by position manager
                    position.lps = 0;
                }
            }

            // update token position LP
            position.lps += vars.lpBalance;
            // set token's position deposit time to the original lender's deposit time
            position.depositTime = vars.depositTime;

            // save position in storage
            tokenInfo.positions[index] = position;

            unchecked { ++i; }
        }

        // update pool LP accounting and transfer ownership of LP to PositionManager contract
        pool.transferLP(owner, address(this), indexes_);

        emit MemorializePosition(owner, tokenId_, indexes_);
    }

```

Most especially at [L231-238](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/PositionManager.sol#L231-L238)

```solidity
// check for previous deposits
if (position.depositTime != 0) {
  // check that bucket didn't go bankrupt after prior memorialization
  if (_bucketBankruptAfterDeposit(pool, index, position.depositTime)) {
      //@audit
    // if bucket did go bankrupt, zero out the LP tracked by position manager
    position.lps = 0;
  }
}
```

When a bucket is found to be bankrupt after a previous deposit, the contract zeroes out the LPs tracked by the `PositionManager` for that bucket. However, in the current implementation, there's no check or claim for unclaimed rewards before this operation.

In contrast, if you look at the `RewardsManager`, specifically the [claimRewards](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/RewardsManager.sol#LL112C1-L144C6) function there's no requirement for the bucket to not be bankrupt. Thus, even if a bucket is bankrupt, the lender could still claim rewards, which is reliant on the tracked LP balance in the `PositionManager`.

### Impact

Loss of unclaimed rewards for lenders. When the `PositionManager` zeroes out the LP balance of a bankrupt bucket during a memorialization operation, lenders may inadvertently lose their unclaimed rewards. This is because claiming rewards is reliant on the tracked LP balance, which is zeroed out in this process.

### Recommendation

Introduce a check for unclaimed rewards before the LP balance is zeroed out in the `memorializePositions` method. This could be implemented in the form of a function call to claim any existing rewards before the LP balance is updated. Additionally, adequate warnings or prompts could be provided to inform the lender about unclaimed rewards before proceeding with the operation, thus giving the lender a chance to claim their rewards before their position is updated.

## [H-01] Lack of Frontrun Protection in the `updateBucketExchangeRatesAndClaim` Function Could Disadvantage Honest Users

### Summary

The function `updateBucketExchangeRatesAndClaim` within the Ajna protocol currently incentivizes users to update bucket exchange rates by rewarding the first participant to do so after each burn event. However, due to its first-come-first-serve nature, it is exposed to a type of frontrunning attack known as Miner Extractable Value (MEV) attacks. This scenario may allow well-resourced actors or bots to exploit the reward system, capturing the majority of rewards unfairly, and thus demotivating honest users from actively contributing to the system.

### Vulnerability Details

The `updateBucketExchangeRatesAndClaim` function permits any actor to update the bucket exchange rates and consequently claim a reward if eligible. As the reward distribution is based on a first-come-first-serve mechanism, it is highly susceptible to MEV bots.

MEV bots are capable of monitoring the pending transactions pool (mempool) and identifying transactions that call the `updateBucketExchangeRatesAndClaim` function. Upon detection, the bot can initiate a similar transaction but with a higher gas price. This tactic ensures that the bot's transaction is mined before the original transaction, allowing it to claim the reward first.

This issue is exacerbated in scenarios where non-tech-savvy users manually track the need for bucket updates. These users put effort into maintaining the system but are constantly outperformed by MEV bots, leading to a discouraging experience.

### Impact

This vulnerability can result in a disproportionate concentration of rewards within a group of well-resourced actors, undermining the decentralization and fairness of the reward mechanism. It may cause a decrease in system maintenance participation and potentially damage the overall health and functioning of the system.

[Here is the function vulnerable to the MEV attack:](https://github.com/sherlock-audit/2023-04-ajna-Bauchibred/blob/7cdb5548492caec638c0217e4e43b3b32b719150/ajna-core/src/RewardsManager.sol#L241-L261)

```solidity
function updateBucketExchangeRatesAndClaim(
    address pool_,
    bytes32 subsetHash_,
    uint256[] calldata indexes_
) external override returns (uint256 updateReward) {
    // revert if trying to update exchange rates for a non Ajna pool
    if (!positionManager.isAjnaPool(pool_, subsetHash_)) revert NotAjnaPool();

    updateReward = _updateBucketExchangeRates(pool_, indexes_);

    // transfer bucket update rewards to sender even if there's not enough balance for entire amount
    _transferAjnaRewards({
        transferAmount_: updateReward,
        minAmount_:      0
    });
}
```

### Recommendation

Introduce a mechanism of unpredictability to the reward distribution, making it less attractive to MEV attacks, this would help protect the `updateBucketExchangeRatesAndClaim` function from MEV attacks and ensure a fairer reward distribution for users who contribute to maintaining the system's state.

## [M-01] Hard-coded Slippage Value in Unstaking Function Can Result in Denial-of-Service

### Summary

A hard-coded slippage value in the unstaking function of Ajna Protocol's RewardsManager contract can potentially block users from unstaking their tokens, leading to a Denial-of-Service (DoS). This issue mainly arises when there's a high demand for unstaking or withdrawals or if a user wishes to withdraw a large amount. It is highly recommended that the protocol should allow users to set their own slippage value to prevent any potential DoS.

### Vulnerability Detail

Slippage in DeFi protocols refers to the difference between the expected price of a trade and the actual price at which the trade is executed. Ajna Protocol's RewardsManager contract's `unstake()` function has a hard-coded zero slippage, exarcebating the issue the slippage has been set to "0"
Take a look at the [unstake()](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/RewardsManager.sol#L216-L223) function, which calls the [`_unstake()`](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/RewardsManager.sol#L763-L814) function

```solidity
    function unstake(
        uint256 tokenId_
    ) external override {
        _unstake({
            tokenId_:      tokenId_,
            claimRewards_: true
        });
    }


    function _unstake(uint256 tokenId_, bool claimRewards_) internal {
        StakeInfo storage stakeInfo = stakes[tokenId_];


        if (msg.sender != stakeInfo.owner) revert NotOwnerOfDeposit();


        address ajnaPool = stakeInfo.ajnaPool;
        uint256 rewardsEarned;


        // gracefully unstake, claim rewards if any
        if (claimRewards_) {
            rewardsEarned = _calculateAndClaimAllRewards(
                stakeInfo,
                tokenId_,
                IPool(ajnaPool).currentBurnEpoch(),
                false,
                ajnaPool
            );
        }


        // remove bucket snapshots recorded at the time of staking
        uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);
        uint256 noOfIndexes = positionIndexes.length;


        for (uint256 i = 0; i < noOfIndexes; ) {
            delete stakeInfo.snapshot[positionIndexes[i]]; // reset BucketState struct for current position


            unchecked { ++i; }
        }


        // remove recorded stake info
        delete stakes[tokenId_];


        emit Unstake(msg.sender, ajnaPool, tokenId_);


        // gracefully unstake, transfer rewards to claimer ensuring entire amount
        if (claimRewards_) {
                //@audit Hardcoded slippage value would cause DOS to users while unstaking if rewardsEarned > is greater than contract's Ajna balance but user is ready to forfeit part of the rewards and receive a reward < the balance
            _transferAjnaRewards({
                transferAmount_: rewardsEarned,
                minAmount_:      rewardsEarned
            });
        }


        // transfer LP NFT from contract to sender
        IERC721(address(positionManager)).transferFrom(address(this), msg.sender, tokenId_);
    }
```

In cases where a lot of users are trying to unstake or withdraw, or when a single user tries to withdraw a large amount, a hard-coded zero slippage could prevent the transaction from going through if the available balance in the contract is less than the calculated rewards. This could potentially block users from unstaking their tokens and lead to a Denial-of-Service (DoS).

#### Hypothetical POC

Consider a scenario where the balance of the contract is 1,200 tokens. A user tries to unstake, and their reward is calculated as 1,300 tokens. The user is prepared to accept a 10% slippage as he understands, which would result in a reward of 1,170 tokens. Given that this is less than the contract balance, the transaction should go through in normal circumstances but there is no availability for user to provide his acceptable minAmount so users rewards get stucked in the contract

Note that in this instance, with a hardcoded zero slippage, the transaction reverts because the reward is greater than the available balance. Therefore, even though the user is willing to accept a lower amount due to slippage, they are denied the opportunity to unstake their tokens. This could lead to a potential DoS under high volatility scenarios.

### Impact

This flaw could potentially block a large number of users from unstaking their tokens especially when there's a high demand for unstaking or withdrawals. It restricts users from controlling their assets, possibly leading to financial losses due to missed market opportunities. Therefore, the severity of this vulnerability is considered high.

### Recommendation

To mitigate this issue, the protocol should allow users to specify their acceptable slippage when unstaking, replacing the hard-coded zero slippage. Here's an example of how the new code might look:

```solidity
function unstake(uint256 tokenId_, uint256 minAmount_) external override {
    _unstake({
        tokenId_:      tokenId_,
        minAmount_:    minAmount_,
        claimRewards_: true
    });
}

function _unstake(uint256 tokenId_, uint256 minAmount_, bool claimRewards_) internal {
    // ... omitted for brevity ...

    if (claimRewards_) {
        _transferAjnaRewards({
            transferAmount_: rewardsEarned,
            minAmount_:      minAmount_
        });
    }
}
```

By allowing users to set their own slippage value, they would have more control over their assets and prevent potential DoS, especially during times when there's a high demand for unstaking or withdrawals.

## [M-02] PositionManager & PermitERC721 do not comply with EIP-4494

### Summary

The [Scope Q&A](https://github.com/sherlock-audit/2023-04-ajna#q-is-the-codecontract-expected-to-comply-with-any-eips-are-there-specific-assumptions-around-adhering-to-those-eips-that-watsons-should-be-aware-of) explicitly stated an expectation for the contract/code under inspection to comply with EIP-4494. However, the `PermitERC721` and `PositionManager` contracts do not satisfy the requirements of the EIP-4494 standard. Specifically, they lack the implementation of the IERC165 interface and do not indicate support for the 0x5604e225 interface. These discrepancies, mark the contracts as non-compliant with the EIP-4494 standard, which could lead to potential interoperability issues.

### Vulnerability Detail

The [PermitERC721](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/base/PermitERC721.sol#L1-L50) and [PositionManager](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/PositionManager.sol#L1-L40) contracts have not been implemented according to the specifications of the [EIP-4494](https://eips.ethereum.org/EIPS/eip-4494) standard. They lack the crucial implementation of the IERC165 interface and do not support the 0x5604e225 interface, both of which are mandatory according to the EIP-4494 standard... quoting the EIP:
_This EIP requires EIP-165. EIP165 is already required in ERC-721, but is further necessary here in order to register the interface of this EIP. Doing so will allow easy verification if an NFT contract has implemented this EIP or not, enabling them to interact accordingly. The interface of this EIP (as defined in EIP-165) is 0x5604e225. Contracts implementing this EIP MUST have the supportsInterface function return true when called with 0x5604e225._

### Impact

The absence of the IERC165 implementation and lack of support for the 0x5604e225 interface in accordance with the EIP-4494 standard have several potential implications:

1. The `PositionManager` & `PermitERC721` contracts **do not comply with the EIP-4494 standard**.
2. This discrepancy could hamper the contracts' interoperability with other systems and smart contracts, as third-party contracts would be unable to identify their adherence to the `EIP-4494` standard.

### Recommendation

To fully comply with the EIP-4494 standard, both the `PermitERC721` and `PositionManager` contracts must implement the IERC165 interface and declare their support for the 0x5604e225 interface. This necessary adjustment will resolve the non-compliance and ensure smooth interoperability with other systems and smart contracts.

## [M-03] Potential Loss of Bucket Rewards due to Missing Slippage Control in the Staking Function

### Summary

The RewardsManager.sol contract in Ajna Protocol lacks a user-specified slippage mechanism in its staking function (Note that the stake() function unfairly sets it to 100%). This absence could lead to users losing out on their expected profits.

### Vulnerability Detail

In Ajna Protocol's RewardsManager.sol the [stake()](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/RewardsManager.sol#L146-L206) function is responsible for handling user staking actions. However this function contains a potential flaw due to the lack of user-defined slippage control.
Take a look at the [stake()](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/RewardsManager.sol#L146-L206) function

```solidity
    /**
     *  @inheritdoc IRewardsManagerOwnerActions
     *  @dev    === Revert on ===
     *  @dev    not owner `NotOwnerOfDeposit()`
     *  @dev    === Emit events ===
     *  @dev    - `Stake`
     */
    function stake(
        uint256 tokenId_
    ) external override {
        address ajnaPool = positionManager.poolKey(tokenId_);


        // check that msg.sender is owner of tokenId
        if (IERC721(address(positionManager)).ownerOf(tokenId_) != msg.sender) revert NotOwnerOfDeposit();


        StakeInfo storage stakeInfo = stakes[tokenId_];
        stakeInfo.owner    = msg.sender;
        stakeInfo.ajnaPool = ajnaPool;


        uint96 curBurnEpoch = uint96(IPool(ajnaPool).currentBurnEpoch());


        // record the staking epoch
        stakeInfo.stakingEpoch = curBurnEpoch;


        // initialize last time interaction at staking epoch
        stakeInfo.lastClaimedEpoch = curBurnEpoch;


        uint256[] memory positionIndexes = positionManager.getPositionIndexes(tokenId_);
        uint256 noOfPositions = positionIndexes.length;
        uint256 bucketId;


        for (uint256 i = 0; i < noOfPositions; ) {
            bucketId = positionIndexes[i];


            BucketState storage bucketState = stakeInfo.snapshot[bucketId];
            // record the number of lps in bucket at the time of staking
            bucketState.lpsAtStakeTime = positionManager.getLP(tokenId_, bucketId);
            // record the bucket exchange rate at the time of staking
            bucketState.rateAtStakeTime = IPool(ajnaPool).bucketExchangeRate(bucketId);


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


        // transfer bucket update rewards to sender even if there's not enough balance for entire amount
        _transferAjnaRewards({
            transferAmount_: updateReward,
            minAmount_:      0
        });
    }
```

Key to note that slippage is a common concept in DeFi, referring to the difference between the expected outcome and the actual result of a transaction.

Here is the crucial point of the vulnerability

```solidity
// transfer bucket update rewards to sender even if there's not enough balance for entire amount
_transferAjnaRewards({
transferAmount_: updateReward,
minAmount_: 0
});

```

In this case, when users stake their tokens, the calculated rewards are assumed to be fully paid out due to the hard-coded 100% slippage value. This idealized scenario, however, doesn't always align with reality. If the contract's balance falls short, users will receive a reward lower than the calculated amount, potentially even zero, while the system inaccurately records that the full amount of rewards has been distributed. This discrepancy could result in users unintentionally suffering losses as they may not have opted to stake their tokens had they known they wouldn't receive all of their rewards, or at least the minimum reward amount they would accept even considering slippage (NB: slippage in this case is the user-provided one)

### Impact

The potential impact of this vulnerability is that users may not receive the full rewards they are entitled to according to the system's records. This situation could lead to a trust deficit and financial losses for the users, thus compromising the reliability and fairness of the staking process.

### Recommendation

A recommended mitigation strategy for this vulnerability is to allow users to specify their acceptable minimum reward amount, or slippage, during the staking process. This modification can replace the current hard-coded 100% slippage, thereby empowering users to define their risk tolerance levels and potential profits. Here's an example of how this can be implemented:

```solidity
function stake(uint256 tokenId_, uint256 minAmount_) external override {
    // ... code omitted for brevity ...

    _transferAjnaRewards({
        transferAmount_: updateReward,
        minAmount_:      minAmount_
    });
}
```

By implementing user-defined slippage, users gain more control over their rewards, ensuring that they can decide the minimum acceptable return for their staking actions, thereby improving the trustworthiness and user-friendliness of the protocol.
Certainly! Here is the rearranged report:

## [L-01] Adopting the EIP-4494 Standard, Currently in its Draft Stage, May Lead to Potential Compatibility Challenges

### Summary

The PositionManager.sol contract in the Ajna protocol has employed the EIP-4494 standard for its permit implementation. Notably, this EIP is still in its Draft stage, which means it is prone to normative (breaking) changes. Such uncertainty could result in compatibility issues with the PositionManager contract, should the final approved EIP differ significantly from the current state.

Note that the tooltip that appears when hovering over the bolded **DRAFT!** label on the [EIP](https://eips.ethereum.org/EIPS/eip-4494) page issues a caution:

```js
This EIP is not yet recommended for general use or implementation, as it is subject to normative (breaking) changes.
```

### Vulnerability Details

The PositionManager.sol contract follows the EIP-4494 standard, currently in its Draft stage. As such, it presents certain risks associated with the instability of the EIP, which might undergo significant changes before reaching its final state, or might never gain approval at all. Consequently, this could lead to several complications such as:

- The contract might not align with the final approved version of the standard.
- The contract may be rendered incompatible with the standard chosen to implement the permit functionality for ERC721 tokens if EIP-4494 becomes obsolete.
- As tools are expected to support the final approved standard, future tool compatibility might be jeopardized.
- The contract may face issues similar to those caused by incorrect standard implementation.

### Impact

Adherence to a draft standard can lead to several issues, including non-compliance with the final standard, lack of support from future tools, and potential obsolescence if an alternate standard is chosen to implement the permit functionality for ERC721 tokens.

The code snippets provided below highlight the EIP-4494 standard's implementation:

- In the [PositionManager.sol file](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/PositionManager.sol#L40):

```solidity
contract PositionManager is ERC721, PermitERC721, IPositionManager, Multicall, ReentrancyGuard {
```

- In the [PermitERC721.sol file](https://github.com/sherlock-audit/2023-04-ajna/blob/e2439305cc093204a0d927aac19d898f4a0edb3d/ajna-core/src/base/PermitERC721.sol#L45-L50):

```solidity
/**
 *  @notice Functionality to enable `EIP-4494` permit calls as part of interactions with Position `NFT`s
 *  @dev    EIP-4494: https://eips.ethereum.org/EIPS/eip-4494
 *  @dev    References this implementation: https://github.com/dievardump/erc721-with-permits/blob/main/contracts/ERC721WithPermit.sol
 */
abstract contract PermitERC721 is ERC721, IPermit {
```

Check the status of the standard [here](https://eips.ethereum.org/EIPS/eip-4494).

### Recommendation

In light of the potential issues stemming from the draft status of EIP-4494, we recommend the following actions:

- Remove all references to EIP-4494 within the contract.
- Clearly indicate that this contract is an implementation of the ERC721-Permit from Uniswap V3.
