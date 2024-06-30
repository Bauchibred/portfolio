# Introduction

A time-boxed security review of the **Perennial** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Perennial](#About-Perennial)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/79). The competition spanned **24** days, from **_May 22, 2023_**, to **_June 15, 2023_**, and was hosted on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-05-perennial-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the following:

[root @ 914838c1cb2532325ecf5659807f9fca61d635e9](https://github.com/equilibria-xyz/root/tree/914838c1cb2532325ecf5659807f9fca61d635e9)

- [root/contracts/accumulator/types/Accumulator6.sol](root/contracts/accumulator/types/Accumulator6.sol)
- [root/contracts/accumulator/types/UAccumulator6.sol](root/contracts/accumulator/types/UAccumulator6.sol)
- [root/contracts/control/unstructured/CrossChainOwnable/UCrossChainOwnable.sol](root/contracts/control/unstructured/CrossChainOwnable/UCrossChainOwnable.sol)
- [root/contracts/control/unstructured/CrossChainOwnable/UCrossChainOwnable_Arbitrum.sol](root/contracts/control/unstructured/CrossChainOwnable/UCrossChainOwnable_Arbitrum.sol)
- [root/contracts/control/unstructured/CrossChainOwnable/UCrossChainOwnable_Optimism.sol](root/contracts/control/unstructured/CrossChainOwnable/UCrossChainOwnable_Optimism.sol)
- [root/contracts/control/unstructured/CrossChainOwner/UCrossChainOwner.sol](root/contracts/control/unstructured/CrossChainOwner/UCrossChainOwner.sol)
- [root/contracts/control/unstructured/CrossChainOwner/UCrossChainOwner_Arbitrum.sol](root/contracts/control/unstructured/CrossChainOwner/UCrossChainOwner_Arbitrum.sol)
- [root/contracts/control/unstructured/CrossChainOwner/UCrossChainOwner_Optimism.sol](root/contracts/control/unstructured/CrossChainOwner/UCrossChainOwner_Optimism.sol)
- [root/contracts/control/unstructured/UInitializable.sol](root/contracts/control/unstructured/UInitializable.sol)
- [root/contracts/control/unstructured/UOwnable.sol](root/contracts/control/unstructured/UOwnable.sol)
- [root/contracts/control/unstructured/UReentrancyGuard.sol](root/contracts/control/unstructured/UReentrancyGuard.sol)
- [root/contracts/curve/CurveMath18.sol](root/contracts/curve/CurveMath18.sol)
- [root/contracts/curve/CurveMath6.sol](root/contracts/curve/CurveMath6.sol)
- [root/contracts/curve/types/JumpRateUtilizationCurve.sol](root/contracts/curve/types/JumpRateUtilizationCurve.sol)
- [root/contracts/curve/types/JumpRateUtilizationCurve18.sol](root/contracts/curve/types/JumpRateUtilizationCurve18.sol)
- [root/contracts/curve/types/JumpRateUtilizationCurve6.sol](root/contracts/curve/types/JumpRateUtilizationCurve6.sol)
- [root/contracts/curve/types/UJumpRateUtilizationCurve18.sol](root/contracts/curve/types/UJumpRateUtilizationCurve18.sol)
- [root/contracts/curve/types/UJumpRateUtilizationCurve6.sol](root/contracts/curve/types/UJumpRateUtilizationCurve6.sol)
- [root/contracts/number/NumberMath.sol](root/contracts/number/NumberMath.sol)
- [root/contracts/number/types/Fixed18.sol](root/contracts/number/types/Fixed18.sol)
- [root/contracts/number/types/Fixed6.sol](root/contracts/number/types/Fixed6.sol)
- [root/contracts/number/types/PackedFixed18.sol](root/contracts/number/types/PackedFixed18.sol)
- [root/contracts/number/types/PackedUFixed18.sol](root/contracts/number/types/PackedUFixed18.sol)
- [root/contracts/number/types/UFixed18.sol](root/contracts/number/types/UFixed18.sol)
- [root/contracts/number/types/UFixed6.sol](root/contracts/number/types/UFixed6.sol)
- [root/contracts/storage/UStorage.sol](root/contracts/storage/UStorage.sol)
- [root/contracts/token/types/Token18.sol](root/contracts/token/types/Token18.sol)
- [root/contracts/token/types/Token6.sol](root/contracts/token/types/Token6.sol)
- [root/contracts/token/types/TokenOrEther18.sol](root/contracts/token/types/TokenOrEther18.sol)

[perennial-mono @ b06d5145db62a312dd88dfcafef0f8e2166c5e32](https://github.com/equilibria-xyz/perennial-mono/tree/b06d5145db62a312dd88dfcafef0f8e2166c5e32)

- [perennial-mono/packages/perennial-oracle/contracts/ChainlinkFeedOracle.sol](perennial-mono/packages/perennial-oracle/contracts/ChainlinkFeedOracle.sol)
- [perennial-mono/packages/perennial-oracle/contracts/ChainlinkOracle.sol](perennial-mono/packages/perennial-oracle/contracts/ChainlinkOracle.sol)
- [perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol](perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol)
- [perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkRegistry.sol](perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkRegistry.sol)
- [perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkRound.sol](perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkRound.sol)
- [perennial-mono/packages/perennial-vaults/contracts/balanced/BalancedVault.sol](perennial-mono/packages/perennial-vaults/contracts/balanced/BalancedVault.sol)
- [perennial-mono/packages/perennial-vaults/contracts/balanced/BalancedVaultDefinition.sol](perennial-mono/packages/perennial-vaults/contracts/balanced/BalancedVaultDefinition.sol)
- [perennial-mono/packages/perennial/contracts/collateral/Collateral.sol](perennial-mono/packages/perennial/contracts/collateral/Collateral.sol)
- [perennial-mono/packages/perennial/contracts/collateral/types/OptimisticLedger.sol](perennial-mono/packages/perennial/contracts/collateral/types/OptimisticLedger.sol)
- [perennial-mono/packages/perennial/contracts/controller/Controller.sol](perennial-mono/packages/perennial/contracts/controller/Controller.sol)
- [perennial-mono/packages/perennial/contracts/controller/UControllerProvider.sol](perennial-mono/packages/perennial/contracts/controller/UControllerProvider.sol)
- [perennial-mono/packages/perennial/contracts/incentivizer/Incentivizer.sol](perennial-mono/packages/perennial/contracts/incentivizer/Incentivizer.sol)
- [perennial-mono/packages/perennial/contracts/incentivizer/types/ProductManager.sol](perennial-mono/packages/perennial/contracts/incentivizer/types/ProductManager.sol)
- [perennial-mono/packages/perennial/contracts/incentivizer/types/Program.sol](perennial-mono/packages/perennial/contracts/incentivizer/types/Program.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/Accumulator.sol](perennial-mono/packages/perennial/contracts/interfaces/types/Accumulator.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/PackedAccumulator.sol](perennial-mono/packages/perennial/contracts/interfaces/types/PackedAccumulator.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/PackedPosition.sol](perennial-mono/packages/perennial/contracts/interfaces/types/PackedPosition.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/PayoffDefinition.sol](perennial-mono/packages/perennial/contracts/interfaces/types/PayoffDefinition.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/PendingFeeUpdates.sol](perennial-mono/packages/perennial/contracts/interfaces/types/PendingFeeUpdates.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/Position.sol](perennial-mono/packages/perennial/contracts/interfaces/types/Position.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/PrePosition.sol](perennial-mono/packages/perennial/contracts/interfaces/types/PrePosition.sol)
- [perennial-mono/packages/perennial/contracts/interfaces/types/ProgramInfo.sol](perennial-mono/packages/perennial/contracts/interfaces/types/ProgramInfo.sol)
- [perennial-mono/packages/perennial/contracts/multiinvoker/MultiInvoker.sol](perennial-mono/packages/perennial/contracts/multiinvoker/MultiInvoker.sol)
- [perennial-mono/packages/perennial/contracts/multiinvoker/MultiInvokerRollup.sol](perennial-mono/packages/perennial/contracts/multiinvoker/MultiInvokerRollup.sol)
- [perennial-mono/packages/perennial/contracts/periphery/CoordinatorDelegatable.sol](perennial-mono/packages/perennial/contracts/periphery/CoordinatorDelegatable.sol)
- [perennial-mono/packages/perennial/contracts/product/Product.sol](perennial-mono/packages/perennial/contracts/product/Product.sol)
- [perennial-mono/packages/perennial/contracts/product/UParamProvider.sol](perennial-mono/packages/perennial/contracts/product/UParamProvider.sol)
- [perennial-mono/packages/perennial/contracts/product/UPayoffProvider.sol](perennial-mono/packages/perennial/contracts/product/UPayoffProvider.sol)
- [perennial-mono/packages/perennial/contracts/product/types/accumulator/AccountAccumulator.sol](perennial-mono/packages/perennial/contracts/product/types/accumulator/AccountAccumulator.sol)
- [perennial-mono/packages/perennial/contracts/product/types/accumulator/VersionedAccumulator.sol](perennial-mono/packages/perennial/contracts/product/types/accumulator/VersionedAccumulator.sol)
- [perennial-mono/packages/perennial/contracts/product/types/position/AccountPosition.sol](perennial-mono/packages/perennial/contracts/product/types/position/AccountPosition.sol)
- [perennial-mono/packages/perennial/contracts/product/types/position/VersionedPosition.sol](perennial-mono/packages/perennial/contracts/product/types/position/VersionedPosition.sol)

~ 3951 NSLOC

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About **Bauchibred**

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Perennial

**Perennial** is a general-purpose synthetic derivatives primitive for decentralized finance that is is built from first-principles to be powerful, flexible and scaleable so as to meet the needs of DeFi traders, liquidity providers, and developers.

- Website: https://perennial.finance/
- Twitter: https://twitter.com/perenniallabs

# Summary

This report contains **1 high severity** issues, **4 medium** severity issues and **4 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                                                                             | Severity                                         | Status                                                  |
| ------ | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [H-01] | Users Funds could get stucked in protocol due to the dependency on epoch settling before Pending Deposit/Redemption are processed | ![](https://img.shields.io/badge/-High-red)      | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01] | Product's Flywheel Dependency on Oracle Updates is a Downside                                                                     | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [M-02] | ChainlinkRound: Incorrect phase IDs could be returned due to unsafe bitwise operation                                             | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [M-03] | No checks for when the Arbitrum sequencer is down                                                                                 | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [M-04] | Chainlink's latestRoundData() Returns Stale or Incorrect Result                                                                   | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01] | ` _aggregatorRoundIdToProxyRoundId()` does not follow the correct logic                                                           | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-02] | Unsafe downcasting in ChainlinkAggregator and all other Chainlink-prefixed contracts                                              | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-03] | Pragma isn't specified correctly, which can lead to non-functional/damaged contracts when deployed on Arbitrum                    | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-04] | Using Chainlink oracles would cause issues when the aggregator hits min answer                                                    | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [H-01] Users Funds could get stucked in protocol due to the dependency on epoch settling before Pending Deposit/Redemption are processed

### Summary

The protocol suffers from a critical issue where pending deposit and redemption requests can get stuck and remain unprocessed due to the dependency on epoch settling. This issue arises from the protocol's design, which requires the completion of epochs to be dependent on the advancement of all underlying oracles' versions. If any one of the oracles fails to update or advance its version, the protocol and vault may become stuck in the current epoch, preventing the processing of pending transactions. As a result, users pending redemptions can become effectively locked within the contract, impeding normal operations and causing frustration.

### Vulnerability Detail

The protocol's design requires all underlying oracles to advance their versions before an epoch can be considered complete. However, if any one of the oracles fails to update or advance its version, the protocol and vault will remain stuck in the current epoch. This dependency on all oracles advancing their versions poses a significant risk, as it can lead to a state of limbo where the protocol cannot progress to the next epoch.

[BalancedVault#L330-551:](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-vaults/contracts/balanced/BalancedVault.sol#L330-L351)

```solidity
    /**
     * @notice Returns the current epoch
     * @return The current epoch
     */
    function currentEpoch() public view returns (uint256) {
        return currentEpochComplete() ? _latestEpoch + 1 : _latestEpoch;
    }

    /**
     * @notice Returns the whether the current epoch is currently complete
     * @dev An epoch is "complete" when all of the underlying oracles have advanced a version
     * @return Whether the current epoch is complete
     */
    function currentEpochComplete() public view returns (bool) {
        for (uint256 marketId; marketId < totalMarkets; marketId++) {
            if (
                Math.min(markets(marketId).long.latestVersion(), markets(marketId).short.latestVersion()) ==
                _versionAtEpoch(marketId, _latestEpoch)
            ) return false;
        }
        return true;
    }
```

Note the L340: `     * @dev An epoch is "complete" when all of the underlying oracles have advanced a version`

The impact of a stuck epoch is twofold. Firstly, the protocol and vault are unable to move to the next epoch, resulting in a prolonged period of inactivity and hindering the protocol's ability to function as intended. Secondly, pending deposit and redemption requests remain unprocessed until an epoch can be successfully settled. This dependency on epoch completion creates a bottleneck in the protocol's functionality, causing delays and inconveniences for users.

[BalancedVault#L368-424:](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-vaults/contracts/balanced/BalancedVault.sol#L368-L424)

```solidity
    /**
     * @notice Hook that is called before every stateful operation
     * @dev Settles the vault's account on both the long and short product, along with any global or user-specific deposits/redemptions
     * @param account The account that called the operation, or 0 if called by a keeper.
     * @return context The current epoch contexts for each market
     * @return accountContext The current epoch contexts for each market for the given account
     */
    function _settle(address account) private returns (EpochContext memory context, EpochContext memory accountContext) {
        (context, accountContext) = _loadContextForWrite(account);

// *lines ommited for report's brevity*

            // process pending deposit / redemption after new epoch is settled
            _deposit = _pendingDeposit;
            _redemption = _pendingRedemption;
            _pendingDeposit = UFixed18Lib.ZERO;
            _pendingRedemption = UFixed18Lib.ZERO;
        }

// *lines ommited for report's brevity*
}
```

When an epoch is stuck and unable to settle, all pending deposit and redemption requests remain in a pending state within the contract. As a result, users' funds become effectively locked within the contract, it also prevents normal operations as deposits wouldn't go through causing frustration for users.

### Impact

The combined impact of the stuck epoch and pending deposit/redemption processing dependency is as follows:

1. Inability to progress: The protocol and vault are unable to move to the next epoch if any underlying oracle fails to update or advance its version. This leads to prolonged periods of inactivity, hindering the protocol's ability to function as intended.

2. Delayed (Or forever not ) processing of pending depositions: Pending deposit and requests remain unprocessed until an epoch can be successfully settled. Users experience delays in depositing or their funds, causing inconvenience and potential frustration.

3. Funds lock-up: Users' funds involved in pending redemption requests are effectively locked within the contract when an epoch is stuck and cannot settle. This prevents normal operations and restricts users' ability to access their funds.

### Recommendation

To address this issue two things should be revised:

1. Consider revising the epoch completion criteria to reduce reliance on the advancement of _all_ underlying oracles' versions.
   If not then Introduce alternative mechanisms or fallback solutions to prevent the protocol and vault from getting stuck in an epoch if one of the oracle fails to update.
   This could involve tracking each oracle independently and allowing for transaction processing even if an epoch doesn't settle. However, implementing such a solution would require careful consideration due to increased complexity.

2. Decoupling pending transaction processing from epoch completion: Explore options to decouple pending deposit and redemption processing from epoch completion. Implement mechanisms to process pending transactions separately, even if an epoch is unable to settle. This approach would ensure a smoother user experience and prevent funds from getting stuck within the contract.

By implementing these recommendations, the protocol can mitigate the risk of stuck epochs and ensure the timely processing of pending deposit and redemption requests. This will contribute to a more efficient and reliable user experience, addressing the issues caused by the dependency on epoch settling.

## [M-01] Product's Flywheel Dependency on Oracle Updates is a Downside

### Summary

The product's flywheel in protocol is dependent on continuous updates from an oracle. However, if the oracle stops updating, it can lead to a stagnant state where the flywheel stops spinning. This creates a significant downside as the protocol and vault become extensively stuck.
It is important to note that oracles can go offline for various reasons, such as maintenance or token prices falling to zero. In such cases, the flywheel won't spin, resulting in halted operations due to the unavailability of updated data.

### Vulnerability Detail

Currently, only one oracle source is integrated into the protocol (chainlink), exacerbating the issue.

Chainlink, a popular oracle provider, has taken oracles offline in extreme cases to ensure the accuracy and reliability of data. For instance, during the UST collapse, Chainlink temporarily paused the UST/ETH price oracle to prevent the dissemination of inaccurate data to protocols.

### Impact

The dependency on the oracle updates for the product's flywheel poses a significant risk. If an oracle goes down or stops updating, the flywheel will cease to function, leaving the protocol and vault(extensively) in a stucked state. This can have far-reaching consequences, including the inability to execute crucial operations and potential financial losses for users.

### Recommendation

Introduce Safeguards: Implement a safeguard mechanism to protect against potential issues arising from oracle downtimes or failures. This can involve integrating fallback or backup oracles that can be activated in case the primary oracle becomes unresponsive. This redundancy will help ensure the availability of updated data and enable the flywheel to continue spinning.

## [M-02] ChainlinkRound: Incorrect phase IDs could be returned due to unsafe bitwise operation

### Summary

This report is centered on a vulnerability discovered in the `phaseId` function of the `ChainlinkRoundLib` library. The bitwise shift operation being done unsafely could lead to incorrect phase IDs being returned when handling `roundId` values less than `2^64`.

### Vulnerability Detail

The `phaseId` function in the `ChainlinkRoundLib` library is designed to extract the phase ID from a given round by shifting the round ID 64 bits to the right. However, there are no checks in place to ensure that the round ID is greater than or equal to `2^64`. Thus, if the function is supplied with a `roundId` value less than `2^64`, the shift operation will always result in 0, leading to an incorrect phase ID being returned.

Additionally, the return type of the `phaseId` function is `uint16`, hence the result of the bitwise shift operation is implicitly cast to `uint16`. If the actual phase ID after the shift operation is greater than the maximum `uint16` value, it will result in incorrect downcasting and thus an incorrect phase ID.

```solidity
function phaseId(ChainlinkRound memory self) internal pure returns (uint16) {
    return uint16(self.roundId >> PHASE_OFFSET); // PHASE_OFFSET = 64
}
```

#### POC:

To demonstrate this bug, here is a minimalistic contract and a simple test in JavaScript using the Hardhat framework:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Staking {
    constructor() {}

    function getShiftedValue(uint256 x) public pure returns (uint256) {
        return x >> 64;
    }
}
```

```javascript
const { expect } = require("chai");

describe("Staking contract", function () {
  it("Should return incorrect value for inputs less than 2^64", async function () {
    const Staking = await ethers.getContractFactory("Staking");
    const staking = await Staking.deploy();
    await staking.deployed();

    let result = await staking.getShiftedValue(10);
    expect(result.toString()).to.equal("0"); // For all values less than 2^64, it will return 0

    result = await staking.getShiftedValue("18446744073709551615"); // 2^64 - 1
    expect(result.toString()).to.equal("0"); // For all values less than 2^64, it will return 0

    result = await staking.getShiftedValue("18446744073709551616"); // 2^64
    expect(result.toString()).to.equal("1"); // This should return 1, but will return 0 for all values less than 2^64
  });
});
```

This test case will show that the `getShiftedValue` function will return `0` for all inputs less than `2^64`, even when it should return `1` for an input of `2^64`. It helps to demonstrate the incorrect behavior of the shift operation for inputs less than `2^64`.

Also other implemnetation of the `roundId` in other contracts suggest that this value could actually be smaller than uint64 since unsafe downcasting are done on the value of the roundID

### Impact

The main impact of this vulnerability is that it may lead to the incorrect processing of round data. This can have severe implications, particularly in scenarios where accurate phase ID values are critical for the correct execution of the smart contract. An incorrect phase ID can mislead the contract into operating on incorrect data or phases.

### Recommendation

In order to fix this vulnerability, a check should be added to ensure that the `roundId` is greater than or equal to `2^64` before performing the shift operation. Furthermore, it would be beneficial to add additional checks to ensure that the phase ID after the shift operation is less than or equal to the maximum `uint16` value, in order to prevent incorrect downcasting.

If larger phase ID values are expected, consider changing the return type of the `phaseId` function to accommodate larger numbers. Always ensure the data types and operations used in your contract align with your specific use-case requirements and constraints.

## [M-03] No checks for when the Arbitrum sequencer is down

### Summary

The Chainlink oracles lack a check to determine if the Arbitrum sequencer is down, when using Chainlink feeds on L2 chains. This omission may result in obtaining prices that appear fresh but are actually outdated, leading to potential vulnerabilities.

### Vulnerability Detail

Instances of the vulnerable code snippet can be found in [ChainlinkAggregator.sol](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol#L32-L36) and numerous other Chainlink-prefixed contracts.

### Impact

The impact of this vulnerability depends on the usage of any of the queries but since these queries are going to be used to obtain prices for critical operations, such as determining asset values or executing financial transactions, the lack of sequencer status check would clearly lead to protocol using outdated prices, i.e get better borrows or avoid liquidations

### Recommendation

It is recommended to follow the code example of Chainlink:
https://docs.chain.link/data-feeds/l2-sequencer-feeds#example-code

TLDR: Implement a check to verify the status of the Arbitrum sequencer before retrieving prices from Chainlink feeds as this check should ensure that prices are fetched only when the sequencer is operational and providing up-to-date data.

## [M-04] Chainlink's latestRoundData() Returns Stale or Incorrect Result

### Summary

The contracts in ecosystem that are chainlink-prefixed use the `latestRoundData()` function to fetch price data. However, these contracts do not perform any checks to ensure that the returned prices are not stale. This vulnerability exposes the protocol to potential risks associated with using outdated or incorrect price data.

### Vulnerability Detail

In multiple Chainlink-prefixed contracts, a call to `latestRoundData()` is made either directly or indirectly, but no checks are implemented to validate the freshness of the prices returned. This means that the protocol may use stale prices for critical operations, which can lead to inaccurate calculations and incorrect decision-making.

An example of such usage can be found in the [ChainlinkAggregator.sol](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol#L32-L36) contract.

### Impact

The impact of this vulnerability is that the protocol may rely on outdated or incorrect price data, which can have severe consequences. Inaccurate prices can result in incorrect valuations, improper risk management, and potential financial losses for users of the protocol.

The vulnerable code snippet can be found in [ChainlinkAggregator.sol](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol#L32-L36).
And almost all other Chainlink-prefixed contracts

### Recommendation

To mitigate this vulnerability, it is recommended to implement checks when retrieving prices from the Chainlink oracles. The following checks can be added to ensure the freshness and accuracy of the data:

## [L-01] ` _aggregatorRoundIdToProxyRoundId()` does not follow the correct logic

### Summary

L170 of ChainlinkAggregator.sol states that the ` _aggregatorRoundIdToProxyRoundId()` function is to follow the `roundid-in-proxy` logic from chainlink

```solidity
     * @dev Follows the logic specified in https://docs.chain.link/data-feeds/price-feeds/historical-data#roundid-in-proxy

```

But unlike the impementation for getting the roundId from chainlink, `_aggregatorRoundIdToProxyRoundId()` uses a bitwise shift and arithmetic addition operation to compose `roundId` from `phaseId` and `aggregatorRoundId`.

```solidity
return (uint256(phaseId) << 64) + aggregatorRoundId;
```

Whereas the [original implementation](https://docs.chain.link/data-feeds/historical-data#roundid-in-proxy), uses a bitwise shift and a `bitwise OR operation`

```solidity
roundId = uint80((uint256(phaseId) << 64) | aggregatorRoundId);

```

This change introduces undesired behavior and bugs due to differences in how arithmetic addition and bitwise OR operations handle overlapping bits.

### Vulnerability Detail

The key difference lies in the usage of arithmetic addition (`+`) instead of bitwise OR (`|`). These two operators can sometimes behave in the same way, but there are crucial differences:

- Arithmetic addition (`+`): This operation actually adds the numeric values together, which can lead to unexpected results if there are overlapping bits, essentially causing a "carry" operation to the higher bits.

- Bitwise OR (`|`): This operation combines the bits of the two numbers without a carry. It is often used in situations where we want to ensure that two sets of bits don't interfere with each other.

[`_aggregatorRoundIdToProxyRoundId()`](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol#L167-L177)

```solidity
    /**
     * @notice Convert an aggregator round ID into a proxy round ID for the given phase
     * @dev Follows the logic specified in https://docs.chain.link/data-feeds/price-feeds/historical-data#roundid-in-proxy
     * @param phaseId phase ID for the given aggregator round
     * @param aggregatorRoundId round id for the aggregator round
     * @return Proxy roundId
     */

    function _aggregatorRoundIdToProxyRoundId(uint16 phaseId, uint80 aggregatorRoundId) private pure returns (uint256) {
        return (uint256(phaseId) << 64) + aggregatorRoundId;
    //  @audit the above differs from chainlink's implementation roundId = uint80((uint256(phaseId) << 64) | aggregatorRoundId);
    }
}
```

#### POC:

Copy paste this contract into remix

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract RoundIdTest {
    uint phaseId = 1;
    uint aggregatorRoundId = 2 ** 80 - 1;

    function getRoundIdWithAddition() public view returns (uint256) {
        return (uint256(phaseId) << 64) + aggregatorRoundId;
    }

    function getRoundIdWithBitwiseOr() public view returns (uint256) {
        return (uint256(phaseId) << 64) | aggregatorRoundId;
    }

    function test() public view returns (string memory) {
        uint256 roundIdAdd = getRoundIdWithAddition();
        uint256 roundIdOr = getRoundIdWithBitwiseOr();

        if (roundIdAdd != roundIdOr) {
            return "Inconsistent roundId computation!";
        } else {
            return "RoundId computation is consistent.";
        }
    }
}
```

The `test()` function will return "Inconsistent roundId computation!" cause there is a difference in roundId computation due to arithmetic addition and bitwise OR operation, showing the issue.

### Impact

By using arithmetic addition instead of bitwise OR, it could lead to incorrect values for `roundId` whenever there are overlapping bits in `phaseId` and `aggregatorRoundId`. In other words, if a bit is set in the same position in both `phaseId` and `aggregatorRoundId`, arithmetic addition will cause this bit to "carry over" into the next higher bit, while bitwise OR would not. This carry-over could result in `roundId` having an incorrect value.

### Recommendation

It is recommended to stick with the original implementation which uses bitwise OR operation to combine `phaseId` and `aggregatorRoundId`. This method is safer as it avoids potential issues from bit overlap that can occur with arithmetic addition.

## [L-02] Unsafe downcasting in ChainlinkAggregator and all other Chainlink-prefixed contracts

### Summary

Integer overflow vulnerability could happen due to unsafe type casting operations. This is seen in multiple instances in the ChainlinkAggregator.sol contract where `uint256` is unsafely cast to `uint80`, leading to possible overflow errors and erroneous data retrieval.

NB: This report only covers the instance in `getRound` and `_search` though issue exist in `_tryGetProxyRoundData` in the same ChainlinkAggregator.sol contract and multiple instances in other Chainlink-prefixed contracts

### Vulnerability Detail

The unsafe type casting is found in two functions: `getRound` and `_search`. In both instances, a `uint256` type variable is converted to a `uint80` type. If the `uint256` value is greater than the maximum `uint80` value (`2^80-1`), this would result in an integer overflow, effectively causing the value to wrap around to a much smaller, incorrect value.

This can cause critical issues as these functions are fetching data based on round Ids. An overflow could lead to data being fetched for incorrect round Ids, potentially causing the contract to return incorrect or unexpected values.

### Impact

Integer overflow can lead to unexpected behavior and security vulnerabilities. If the overflow occurs when fetching round data, it can lead to incorrect data being retrieved and used. This can impact the contract's normal functioning and could potentially be exploited by malicious actors to manipulate or disrupt the contract's operations.

In the [getRound()](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol#L44-L48) function, a `uint256` `roundId` is converted to `uint80` when calling the `getRoundData` method:

```solidity
function getRound(ChainlinkAggregator self, uint256 roundId) internal view returns (ChainlinkRound memory) {
    (, int256 answer, , uint256 updatedAt, ) =
        AggregatorProxyInterface(ChainlinkAggregator.unwrap(self)).getRoundData(uint80(roundId));
    return ChainlinkRound({roundId: roundId, timestamp: updatedAt, answer: answer});
}
```

In the [`_search()`](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol#L123-L156) function, a similar conversion takes place:

```solidity
function _search(AggregatorProxyInterface proxy, uint16 phaseId, uint256 targetTimestamp, uint256 minTimestamp, uint256 minRoundId) private view returns (uint256) {
    // ... some code skipped ...
    uint256 midRound = Math.average(minRoundId, maxRoundId);
    uint256 midTimestamp = _tryGetProxyRoundData(proxy, phaseId, uint80(midRound));
    // ... some code skipped ...
}
```

In both these cases, if the value is too large to be represented as `uint80`, the conversion will result in an overflow, leading to an incorrect round Id.

NB: One could argue that roundId shouldn't be that big but the instance below from the [ChainlinkRound.sol](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkRound.sol#L27-L29) contracts suggests otherwise, since roundId should be bigger than uint64 for the bitwise operation to execute error-free which implies that then can aswell be bigger than uint80 in _edge_ cases which validates the problems explained above

```solidity
    function phaseId(ChainlinkRound memory self) internal pure returns (uint16) {
        return uint16(self.roundId >> PHASE_OFFSET);
    }
```

### Recommendation

It is recommended to implement checks before each conversion to ensure that the `uint256` value is within the valid range for a `uint80`. Here's an example of such a safety check:

```solidity
require(roundId <= type(uint80).max, "RoundId overflow");
```

This line should be added before each conversion in both `getRound` and `_search` functions. It will throw an exception if the roundId value is too large to be safely cast to `uint80`, thus preventing the overflow and ensuring data is fetched for the correct round.

## [L-03] Pragma isn't specified correctly, which can lead to non-functional/damaged contracts when deployed on Arbitrum

### Summary

The current pragma setting in multiple contracts in scope allows compilation with a series of 0.8.x compiler version(in some cases > 0.8.15 versions), which includes versions that are not compatible with Arbitrum. The problem with this is that Arbitrum is [NOT compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with those versions will result in a nonfunctional or potentially damaged version that won't behave as expected. The default behavior of compiler would be to use the newest version which would mean by defualt it will be compiled with the 0.8.20 version which will produce broken code.

### Vulnerability Detail

Arbitrum is not compatible with Solidity versions 0.8.20 and newer. However, the pragma in some contracts (Use the chainlink prefixed contracts as an example) is set to ^0.8.15, which allows the usage of any 0.8.x compiler version above 0.8.15. This can lead to compatibility issues when deploying the contract on Arbitrum.

### Impact

The contracts compiled with incompatible versions of Solidity may not function as expected or could be damaged when deployed on Arbitrum. This can result in unexpected behavior, potential vulnerabilities, or even contract failures.

This occurs in multiple instances in contracts in scope, take this instance in [ChainlinkOracle.sol](https://github.com/sherlock-audit/2023-05-perennial-Bauchibred/blob/cbe53c9b4a80bbb541664627f34ccc1394b184c0/perennial-mono/packages/perennial-oracle/contracts/ChainlinkOracle.sol#L2) as an example

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity ^0.8.15;
// Contract code...
```

The issue lies in the pragma statement specifying the Solidity version.

### Recommendation

It is recommended to constrain the pragma statement to ensure compatibility with Arbitrum. Update the pragma statement as follows:

```solidity
pragma solidity >=0.8.0 <=0.8.19;

// Contract code...
```

By setting the pragma to a specific range of compatible versions (0.8.0 to 0.8.19), the contract will be compiled with a version that is known to be compatible with Arbitrum. This helps to ensure that the deployed contract will function properly on the Arbitrum network.

## [L-04] Using Chainlink oracles would cause issues when the aggregator hits min answer

### Summary

Chainlink aggregators, are solely relied upon as the oracle, can encounter issues when the aggregator hits the minimum answer price. This vulnerability arises from the circuit breaker mechanism built into Chainlink aggregators, which causes the aggregator to return the minimum price instead of the actual price when an asset experiences a significant drop in value. As a result, users can continue to interact with the protocol with an incorrect price, leading to potential exploits. An example of such an incident occurred with Venus on BSC during LUNA's collapse... [source](https://rekt.news/venus-blizz-rekt/)

### Vulnerability Detail

Multiple Chainlink-prefixed contracts utilize the `latestRoundData()` function to retrieve round data from Chainlink oracles. Chainlink aggregators employ circuit breakers, including minimum and maximum prices, to limit price fluctuations. However, if the price of an asset falls below the minimum price, the protocol will still value the token at the minimum price, disregarding its actual value. This allows users to interact with the protocol using incorrect pricing.

For instance, consider TokenA with a minimum price set at $1. If the price of TokenA drops to $0.10, the aggregator will still return $1. Consequently, users can interact with TokenA in protocol as if it were valued at $1, resulting in a capacity that is 10 times higher than its actual value.

It's important to note that since Chainlink oracles are intended to be used as the oracle system in the protocol. However, relying solely on Chainlink is bad and though combining it with other oracles eases the possibility of this vulenrabiltiy it does not completely eliminate it. Cause even when using additional oracles such as UniswapV3Oracle, which employs a long TWAP, the vulnerability persists if the TWAP price nears the minimum price during a downward trend. In such cases, another oracle becomes irrelevant as the matching prices from the two oracles can bypass it. Similarly, if other oracles like Band are employed, a malicious user can launch a DDoS attack on relayers to manipulate pricing updates.

An example of this vulnerability can be found in the [ChainlinkAggregator.sol](https://github.com/sherlock-audit/2023-05-perennial/blob/0f73469508a4cd3d90b382eac2112f012a5a9852/perennial-mono/packages/perennial-oracle/contracts/types/ChainlinkAggregator.sol#L32-L36) contract.

### Impact

In the event of a crash in asset prices (e.g., LUNA), the protocol can be susceptible to manipulation and exploit by leveraging inaccurate pricing.

### Recommendation

To mitigate this vulnerability, it is recommended to perform a simple check on the returned answer against the minimum and maximum price bounds. If the answer falls outside these bounds, the protocol should revert to prevent further operations based on incorrect pricing. The following code snippet demonstrates this check:

```solidity
if (answer >= maxPrice || answer <= minPrice) {
```
