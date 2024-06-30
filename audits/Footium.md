# Introduction

A time-boxed security review of the **Footium** protocol was done by **Bauchibred**, with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Footium](#About-Footium)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/71). The competition spanned **3** days, from **_May 2nd, 2023_** to **_May 5th, 2023_**, and was hosted on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-04-footium-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review centered on the following commit hash [hash](https://github.com/logiclogue/footium-eth-shareable/tree/6c181ea79af7f6715e3891e65ea5ee8def1e957c) taking the following contracts as the core scope

- [footium-eth-shareable/contracts/FootiumAcademy.sol](footium-eth-shareable/contracts/FootiumAcademy.sol)
- [footium-eth-shareable/contracts/FootiumClub.sol](footium-eth-shareable/contracts/FootiumClub.sol)
- [footium-eth-shareable/contracts/FootiumClubMinter.sol](footium-eth-shareable/contracts/FootiumClubMinter.sol)
- [footium-eth-shareable/contracts/FootiumEscrow.sol](footium-eth-shareable/contracts/FootiumEscrow.sol)
- [footium-eth-shareable/contracts/FootiumGeneralPaymentContract.sol](footium-eth-shareable/contracts/FootiumGeneralPaymentContract.sol)
- [footium-eth-shareable/contracts/FootiumPlayer.sol](footium-eth-shareable/contracts/FootiumPlayer.sol)
- [footium-eth-shareable/contracts/FootiumPrizeDistributor.sol](footium-eth-shareable/contracts/FootiumPrizeDistributor.sol)
- [footium-eth-shareable/contracts/FootiumToken.sol](footium-eth-shareable/contracts/FootiumToken.sol)
- [footium-eth-shareable/contracts/common/Errors.sol](footium-eth-shareable/contracts/common/Errors.sol)
- [footium-eth-shareable/contracts/interfaces/IFootiumClub.sol](footium-eth-shareable/contracts/interfaces/IFootiumClub.sol)
- [footium-eth-shareable/contracts/interfaces/IFootiumPlayer.sol](footium-eth-shareable/contracts/interfaces/IFootiumPlayer.sol)

NSLOC : ~584

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

To get the NSLOC count of a file the following are done:

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Footium

**Footium** is a multiplayer football management game where players own and manage their own digital football club that's to be deployed on the famous arbitrum L2.

Website: https://footium.club/
Twitter: https://twitter.com/footium

# Summary

This report contains **3 medium** severity issues, **6 minor** issues found by **Bauchibred** during the course of the time-boxed audit

| #   | Title                                                                                                                   | Severity                                         | Status                                                  |
| --- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| 1   | FootiumClub.sol and FootiumPlayer.sol do not comply with ERC721, breaking composability                                 | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| 2   | Limited support to a specific subset of ERC20 tokens                                                                    | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| 3   | Did not approve to zero first                                                                                           | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| 4   | FootiumPrizeDistributor.sol wrong subtraction in claimETHPrize                                                          | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| 5   | Low-level transfer via call() can fail silently                                                                         | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| 6   | Single-step process for critical ownership transfer/renounce is risky                                                   | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| 7   | Missing events for critical parameter changing operations by owner                                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| 8   | Admin privilege - A single point of failure can allow a hacked or malicious owner use critical functions in the project | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| 9   | Renouncing ownership would affect the normal operation of Footium                                                       | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [M-01] FootiumClub.sol and FootiumPlayer.sol do not comply with ERC721, breaking composability

### Summary

FootiumPlayer and FootiumClub should both comply with ERC721, but with the current implementation of the `supportsInterface()` function since it only supports the ERC165 interfaces not even the ERC721 extensions, and it also does not comply with ERC721 itself.
From [EIP721](https://eips.ethereum.org/EIPS/eip-721):
"Every ERC-721 compliant contract must implement the ERC721 and ERC165 interfaces (subject to “caveats” below):"

### Vulnerability Detail

Below is the [supportsInterface()](https://github.com/sherlock-audit/2023-04-footium/blob/11736f3f7f7efa88cb99ee98b04b85a46621347c/footium-eth-shareable/contracts/FootiumPlayer.sol#L118-L133) function used in contract:

```soliditty
    /**
     * @dev See {IERC165-supportsInterface}.
     */
    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(
            ERC721Upgradeable,
            AccessControlUpgradeable,
            ERC2981Upgradeable
        )
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}

```

As seen this just returns `super.supportsInterface(interfaceId)` now both contracts inherit the below
`import {ERC2981Upgradeable} from "@openzeppelin/contracts-upgradeable/token/common/ERC2981Upgradeable.sol";`

```solidity
contract FootiumPlayer is
    ERC721Upgradeable,
    AccessControlUpgradeable,
    ERC2981Upgradeable,//@audit
    PausableUpgradeable,
    ReentrancyGuardUpgradeable,
    OwnableUpgradeable
```

Below is the [supportsInterface()](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/token/common/ERC2981Upgradeable.sol#L39-L44) function used in the OZ contract:

```
    /**
     * @dev See {IERC165-supportsInterface}.
     */
    function supportsInterface(bytes4 interfaceId) public view virtual override(IERC165Upgradeable, ERC165Upgradeable) returns (bool) {
        return interfaceId == type(IERC2981Upgradeable).interfaceId || super.supportsInterface(interfaceId);
    }
```

As seen this does not include the checks below to ensure that it complies with ERC721 or it's extensions

```solidity
interfaceId == type(IERC721Enumerable).interfaceId ||
interfaceId == type(IERC721Metadata).interfaceId || ;
interfaceId == type(IERC721).interfaceId ||
```

One could argue that the `ERC2981Upgradeable.sol` contracts also inherit `supportsInterface()` from `ERC165Upgradeable`
`abstract contract ERC2981Upgradeable is Initializable, IERC2981Upgradeable, ERC165Upgradeable `
But even ERC165Upgradeable.supportsInterface() only checks for compliance to ERC165
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/utils/introspection/ERC165Upgradeable.sol#L29-L34

```solidity
    /**
     * @dev See {IERC165-supportsInterface}.
     */
    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
        return interfaceId == type(IERC165Upgradeable).interfaceId;
    }
```

You could also check this finding from the Paraspace contest for more info:
https://code4rena.com/reports/2022-11-paraspace#m-24-mintableincentivizederc721-and-ntoken-do-not-comply-with-erc721-breaking-composability

### Impact

Any contract that will make sure it is dealing with an ERC721 compliant NFT will not interoperate with FootiumPlayer and FootiumClub. Marketplaces and any NFT facilities will not operate with FootiumPlayer.

### Recommendation

Change supportedInterface function and look for a way to introduce compliance of ERC721s

```solidity

- @dev See {IERC165-supportsInterface}.
  \*/
  function supportsInterface(bytes4 interfaceId)
  external
  view
  virtual
  override(IERC165)
  returns (bool)
  {
  return
  interfaceId == type(IERC721Enumerable).interfaceId ||
  interfaceId == type(IERC721Metadata).interfaceId || ;
  interfaceId == type(IERC721).interfaceId ||
  }
```

Or use something like solmate's ERC165 implementation

```
    /*//////////////////////////////////////////////////////////////
                              ERC165 LOGIC
    //////////////////////////////////////////////////////////////*/

    function supportsInterface(bytes4 interfaceId) public view virtual returns (bool) {
        return
            interfaceId == 0x01ffc9a7 || // ERC165 Interface ID for ERC165
            interfaceId == 0xd9b67a26 || // ERC165 Interface ID for ERC1155
            interfaceId == 0x0e89341c; // ERC165 Interface ID for ERC1155MetadataURI
    }
```

https://github.com/transmissions11/solmate/blob/03e425421b24c4f75e4a3209b019b367847b7708/src/tokens/ERC1155.sol#L137-L146

## [M-02] Limited support to a specific subset of ERC20 tokens

### Summary

Depending on the ERC20 token, some transfer errors may result in passing unnoticed, and/or some successfull transfer may be treated as failed.

Currently the only supported ERC20 tokens are the ones that fulfill both the following requirements:

always revert on failure;
always returns boolean true on success.
An example of a very well known token that is not supported is Tether USD (USDT).

_👋 IMPORTANT
This issue is not the same as reporting that "return value must be verified to be true" where the checks are missing! Indeed such a simplistic report could be considered invalid as it still does not solve all the problems but rather introduces others. See Vulnerability Details section for rationale._

### Vulnerability Detail

Tokens have different ways of signalling success and failure, and this affect mostly transfer(), transferFrom() and approve() in ERC20 tokens. While some tokens revert upon failure, others consistently return boolean flags to indicate success or failure, and many others have mixed behaviours.

See below a snippet of the USDT Token contract compared to the 0x's ZRX Token contract where the USDT Token transfer function does not even return a boolean value, while the ZRX token consistently returns boolean value hence returning false on failure instead of reverting.

_USDT Token snippet (no return value) from Etherscan_

```solidity
function transferFrom(address _from, address _to, uint _value) public onlyPayloadSize(3 * 32) {
	var _allowance = allowed[_from][msg.sender];

	// Check is not needed because sub(_allowance, _value) will already throw if this condition is not met
	// if (_value > _allowance) throw;

	uint fee = (_value.mul(basisPointsRate)).div(10000);
	if (fee > maximumFee) {
		fee = maximumFee;
	}
	if (_allowance < MAX_UINT) {
		allowed[_from][msg.sender] = _allowance.sub(_value);
	}
	uint sendAmount = _value.sub(fee);
	balances[_from] = balances[_from].sub(_value);
	balances[_to] = balances[_to].add(sendAmount);
	if (fee > 0) {
		balances[owner] = balances[owner].add(fee);
		Transfer(_from, owner, fee);
	}
	Transfer(_from, _to, sendAmount);
}
```

_ZRX Token snippet (consistently true or false boolean result) from Etherscan_

```solidity
function transferFrom(address _from, address _to, uint _value) returns (bool) {
	if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && balances[_to] + _value >= balances[_to]) {
		balances[_to] += _value;
		balances[_from] -= _value;
		allowed[_from][msg.sender] -= _value;
		Transfer(_from, _to, _value);
		return true;
	} else { return false; }
}
```

Here is the `transferERC20()` function from Footium.Escrow:

```solidity
    /**
     * @notice Transfers ERC20 tokens to `to` address.
     * @param erc20Contract ERC20 contract address.
     * @param to Token receiver address.
     * @param amount Token amount to transfer.
     * @dev only the club owner address allowed.
     */
    function transferERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.transfer(to, amount);
    }
```

### Impact

Given the usage of token transfers in FootiumContract.sol there can be 2 types of impacts depending on the ERC20 contract being traded.

_Impact type 1_
The ERC20 token being traded is one that consistently returns a boolean result in the case of success and failure like for example 0x's ZRX Token contract. Where the return value is currently not verified to be true (i.e.: #1, #2, #3, #4, #5, #6) the transfer may fail (e.g.: no tokens transferred due to insufficient balance) but the error would not be detected by the Buffer contracts.

_Impact type 2_

The ERC20 token being traded is one that do not return a boolean value like for example the well knonw Tether USD Token contract. Successful transfers would cause a revert in the Buffer contracts where the return value is verified to be true (i.e.: #1, #2, #3, #4) due to the token not returing boolean results.

Same is true for appove calls.

### Recommendation

To handle most of these inconsistent behaviors across multiple tokens, either use OpenZeppelin's SafeERC20 library, or use a more reusable implementation (i.e. library) of the following intentionally explicit, descriptive example code for an ERC20 transferFrom() call that takes into account all the different ways of signalling success and failure, and apply to all ERC20 transfer(), transferFrom(), approve() calls in the Buffer contracts.

```solidity
IERC20 token = whatever_token;

(bool success, bytes memory returndata) = address(token).call(abi.encodeWithSelector(IERC20.transferFrom.selector, sender, recipient, amount));

// if success == false, without any doubts there was an error and callee reverted
require(success, "Transfer failed!");

// if success == true, we need to check whether we got a return value or not (like in the case of USDT)
if (returndata.length > 0) {
	// we got a return value, it must be a boolean and it should be true
	require(abi.decode(returndata, (bool)), "Transfer failed!");
} else {
	// since we got no return value it can be one of two cases:
	// 1. the transferFrom does not return a boolean and it did succeed
	// 2. the token address is not a contract address therefore call() always return success = true as per EVM design
	// To discriminate between 1 and 2, we need to check if the address actually points to a contract
	require(address(token).code.length > 0, "Not a token address!");
}
```

## [M-03] Did not approve to zero first

### Summary

Allowance is not set to zero first before changing the allowance.

### Vulnerability Detail

Some ERC20 tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value. For example Tether (USDT)'s `approve()` function will revert if the current approval is not zero, to protect against front-running changes of approvals.

The following attempt to call the `approve()` function without setting the allowance to zero first.

https://github.com/sherlock-audit/2023-04-footium/blob/11736f3f7f7efa88cb99ee98b04b85a46621347c/footium-eth-shareable/contracts/FootiumEscrow.sol#L68-L81

```solidity
    /**
     * @notice Sets approval for ERC20 tokens.
     * @param erc20Contract ERC20 contract address.
     * @param to The address of token spender.
     * @param amount Token amount to spend.
     * @dev only the club owner address allowed.
     */
    function setApprovalForERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.approve(to, amount);
    }
```

However, if the token involved is an ERC20 token that does not work when changing the allowance from an existing non-zero allowance value, it will break a number of key functions or features of the protocol as the `FootiumEscrow.setApprovalForERC20()` function is utilised extensively within the protocol

### Impact

A number of features within protocol will not work if the approve function reverts.

### Recommendation

It is recommended to set the allowance to zero before increasing the allowance and use safeApprove/safeIncreaseAllowance.

## [L-01] FootiumPrizeDistributor.sol wrong subtraction in claimETHPrize

### Summary

Incomplete check/validation could cause execution of `claimETHPrize()` fail due to the integer underflow whenever the ETH `_amount` user `-to`would like to claim is less than `totalETHClaimed[_to]`

### Vulnerability Detail

Look at the lines from[FootiumPrizeDistributor.sol#L163-L166:]](https://github.com/sherlock-audit/2023-04-footium/blob/11736f3f7f7efa88cb99ee98b04b85a46621347c/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L163-L166)

```solidity
        uint256 value = _amount - totalETHClaimed[_to];

        if (value > 0) {
            totalETHClaimed[_to] += value;
```

### Impact

All `claimETHPrize()` execution could fail due to the integer underflow whenever the ETH `_amount` user `-to`would like to claim is less than `totalETHClaimed[_to]`

### Recommendation

A check should introduced and the lines should instead be changed to:

```
 _amount > totalETHClaimed[_to] ?
        uint256 value = _amount - totalETHClaimed[_to]: 0

        if (value > 0) {
            totalETHClaimed[_to] += value;
```

Also these lines need more comment as i couldn't really get my head around what they do

## [L-02] Low-level transfer via call() can fail silently

### Summary

In a few functionls in scope calls are executed with the following code:

```solidity
            (bool sent, ) = _to.call{value: value}("");
            if (!sent) {
                revert FailedToSendETH(value);
            }
```

Per the Solidity docs:

```solidity
The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed.
```

Therefore, transfers may fail silently.

### Vulnerability Detail

Please find the documentation here: https://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions

### Impact

Cause protocl implements transfer on a low-level via call(), this could fail silently.

### Recommendation

Check for the account's existence prior to transferring.

## [L-03] Single-step process for critical ownership transfer/renounce is risky

### Summary

If an incorrect address, e.g. for which the private key is not known, is used accidentally then it prevents the use of all the `onlyOwner()` functions forever

### Vulnerability Detail

The Footium protocol allows owners to set/approve/transfer and make other important execution which implies that the contract ownership plays a critical role in the protocol.

Given that most contracts i scope inherit from OwnableUpgradeable, the ownership management of this contract defaults to OZ’s transferOwnership() and renounceOwnership() methods which are not overridden here. Such critical address transfer/renouncing in one-step is very risky because it is irrecoverable from any mistakes.

Scenario: If an incorrect address, e.g. for which the private key is not known, is used accidentally then it prevents the use of all the `onlyOwner()` functions forever, which includes the changing of various critical addresses and parameters. This use of incorrect address may not even be immediately apparent given that these functions are probably not used immediately. When noticed, due to a failing `onlyOwner()` function call, it will force the redeployment of these contracts and require appropriate changes and notifications for switching from the old to new address. This will diminish trust in the protocol and incur a significant reputational damage.

For more insights do check the below:

- Similar High Risk severity finding from Trail-of-Bits Audit of Hermez: https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf

- Similar Medium Risk severity finding from Trail-of-Bits Audit of Uniswap V3: https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf

- OwnableUpgradable's transfer/renounce ownership: https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/081776bf5fae2122bfda8a86d5369496adfdf959/contracts/access/OwnableUpgradeable.sol#L59-L76

### Impact

Complete loss of access to all `onlyOwner()` function of all contracts under `footium-eth-shareable` asides `FootiumEscrow`

### Recommendation

Override the inherited methods to null functions and use separate functions for a two-step address change: 1) Approve a new address as a pendingOwner 2) A transaction from the pendingOwner address claims the pending ownership change. This mitigates risk because if an incorrect address is used in step (1) then it can be fixed by re-approving the correct address. Only after a correct address is used in step (1) can step (2) happen and complete the address/ownership change.

Also, consider adding a time-delay for such sensitive actions. And at a minimum, use a multisig owner address and not an EOA.

## [L-04] Missing events for critical parameter changing operations by owner

### Summary

The owner of FootiumClubMinter.sol contract, who is potentially trusted as per specification (though this can change anytime) can change the market critical parameters such as the addresses of the Club/Player/baseURI/ etc, though some other critical variables emit an event once set the aforementioned do not

### Vulnerability Detail

[See similar High-severity finding OpenZeppelin’s Audit of Audius ](https://blog.openzeppelin.com/audius-contracts-audit/#high)
and Medium-severity finding:
1- [OpenZeppelin’s Audit of UMA Phase 4](https://blog.openzeppelin.com/uma-audit-phase-4/)
2- [CodeArena contest for tracer](https://code4rena.com/contests/2021-06-tracer-contest)

### Impact

Owner changes the critical addresses or values that significantly change the security posture/perception of the protocol. No events are emitted and users lose funds/confidence. The protocol takes a reputation hit.

### Recommendation

Consider emitting events when these addresses/values are updated. This will be more transparent and it will make it easier to keep track of the status of the system.

## [L-05] Admin privilege - A single point of failure can allow a hacked or malicious owner use critical functions in the project

### Summary

Two sides to report

1. The owner role has a single point of failure and onlyOwner can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

2. The idea of having a function that executes a transfer of the contract's ether balamce to the address provided by the owner is just a window for the owner to steal ether

### Vulnerability Detail

For 1:

Below are just a few instances of functions just from `FootiumAcademy.sol`

```solidity
    /**
     * @notice Changes the `_maxGenerationId` storage variable.
     * @param maxGenerationId The new value for the `maxGenerationId` storage variable.
     * @dev Only owner address allowed.
     */
    function changeMaxGenerationId(uint256 maxGenerationId) public onlyOwner {
        _maxGenerationId = maxGenerationId;
        emit ChangedMaxGenerationId(_maxGenerationId);
    }

    /**
     * @notice Changes the `_clubDivsMerkleRoot` variable.
     * @param _merkleRoot is the value for the `_clubDivsMerkleRoot` variable.
     * @dev Only owner address allowed.
     */
    function setClubDivsMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
        _clubDivsMerkleRoot = _merkleRoot;

        emit ChangedClubDivsMerkleRoot(_clubDivsMerkleRoot);
    }

    /**
     * @notice Updates division fees.
     * @param _fees an array of division fees to be set.
     * @dev Only owner address allowed.
     */
    function setDivisionFees(uint256[] memory _fees) public onlyOwner {
        uint256 max = _fees.length;

        for (uint256 i = 0; i < max; ) {
            uint256 fee = _fees[i];
            divisionToFee[i + 1] = fee;

            unchecked {
                i++;
            }
        }

        emit ChangedDivisionFees(_fees);
    }

    /**
     * @notice Changes the `currentSeasonId` storage variable.
     * @param _newSeasonId The new value for the `currentSeasonId` storage variable.
     * @dev Only owner address allowed
     */
    function changeCurrentSeasonId(uint256 _newSeasonId) external onlyOwner {
        currentSeasonId = _newSeasonId;

        emit ChangedCurrentSeasonId(currentSeasonId);
    }

    /**
     * @notice Unpause the contract
     * @dev Only owner address allowed.
     */
    function activateContract() external onlyOwner {
        _unpause();
    }

    /**
     * @notice Pause the contract
     * @dev Only owner address allowed.
     */
    function pauseContract() external onlyOwner {
        _pause();
    }
```

Note that the above are just instances within `FootiumAcademy.sol` and other instances exist in other contracts in scope which are covered in `Code Snippet`

For 2:

[FootiumAcademy.withdraw();](https://github.com/sherlock-audit/2023-04-footium/blob/11736f3f7f7efa88cb99ee98b04b85a46621347c/footium-eth-shareable/contracts/FootiumAcademy.sol#L214-L226)

```solidity
    /**
     * @notice Transfers contract available ether balance to the contact owner address
     * @dev Only owner address allowed
     */
    function withdraw() external onlyOwner {
        uint256 balance = address(this).balance;
        if (balance > 0) {
            (bool sent, ) = payable(owner()).call{value: balance}("");
            if (!sent) {
                revert FailedToSendETH(balance);
            }
        }
    }
```

### Impact

1. Hacked owner or malicious owner can immediately use critical functions in the project.

2. Owner can steal ether from the `FootiumAcademy` contract which is a severe undermining of decentralization

### Recommendation

For 1, Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.
Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Note that the status quo regarding significant centralization vectors has always been to award `M severity`, in order to warn users of the protocol of this category of risks. See [here](https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837) for list of centralization issues previously judged.

For 2, Restrict a few accesses from owner

## [L-05] Renouncing ownership would affect the normal operation of Footium

### Summary

Abnormal operation of protocol if/when ownership gets renounced

### Vulnerability Detail

All the contracts that extend from OwnableUpgradeable inherit a method called
renounceOwnership. The owner of the contract can use this method to give up their ownership, thereby leaving
the contract without an owner. If that were to happen, it would not be possible to perform any owner-specific
functionality on that contract anymore.
Under the code snippet is a list of all affected contracts and their impact if the ownership has been renounced.

### Impact

Inability to perform any owner-specific
functionality in all contracts under `footium-eth-shareable` asided `FootiumEscrow`, since owner is now `0x0`

### Recommendation

Review if the renounceOwnership function is required for each of the affected contracts and remove them if they are not needed. Ensure that renouncing ownership in any of the affected contracts will not affect the normal operation of Footium
