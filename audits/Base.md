# Introduction

A time-boxed security review of the **Base** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Base](#About-Base)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://code4rena.com/contests/2023-05-base#top). The competition spanned **14** days, from **_May 26th, 2023_** to **_June 9th, 2023_**, and was hosted on [Code4rena](https://code4rena.com/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://code4rena.com/reports/2023-05-base).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The key components within the scope of this review include:

- [`L1 Contracts`](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1)

- [`L2 Contracts`](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2)

- [`op-node`](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/op-node)

- [`op-geth`](https://github.com/ethereum-optimism/op-geth)

~ 1308 NSLOC in 19 contracts for solidity, + golang

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Base

![Base Logo](https://raw.githubusercontent.com/base-org/node/main/logo.webp)

Base is a secure, low-cost, developer-friendly Ethereum L2 built to bring the next billion users on-chain.
It is built on the MIT-licensed OP Stack, in collaboration with Optimism. Coinbase is joining as the second Core Dev team working on the OP Stack to ensure it’s a public good available to everyone.

# Summary

This report contains **2 high severity** issues, **4 medium** severity issues, **14 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #       | Title                                                                                                                         | Severity                                         | Status                                                      |
| ------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| [H-01]  | Some NFTs can easily get frozen in the protocol                                                                               | ![](https://img.shields.io/badge/-High-d10b0b)   | ![](https://img.shields.io/badge/Acknowledged-brightgreen)  |
| [H-02]  | Bridge transaction for NFTs could fail which leads to user's loss of NFT ownership                                            | ![](https://img.shields.io/badge/-High-d10b0b)   | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [M-01]  | Current transfer flow in `L1ERC721Bridge.sol` provides a window for rewards/airdrops associated with locked NFTs to be stolen | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [M-02]  | Current implementation of `bridgeERC721To()` on L1/L2 could cost users their NFTs                                             | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [M-03]  | Optimism.sol: Inaccurate identification of EOA and smart contract callers in `depositTransaction()` function                  | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Disputed-grey)            |
| [M-04]  | Using `selfdestruct` as the ETH burning mechanism in `L2ToL1MessagePasser` is a downside                                      | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Disputed-grey)            |
| [L-01]  | Critical address changes should always use a two-step procedure                                                               | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-02]  | L1ERC721Bridge.sol: `safeTransferFrom()` should be used in the place of `transferFrom()`                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-03]  | Modifiers should only be used for checks                                                                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-04]  | L1ERC721Bridge does not comply with ERC721, breaking composability                                                            | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-05]  | Using vulnerable dependency version of OpenZeppelin                                                                           | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-06]  | Use of `assert()` instead of `require()`                                                                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-07]  | Known issues with compiler versions used to compile the contracts (sol 0.8.15 & 0.6.0)                                        | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-08]  | Low level function call does not check for contract existence                                                                 | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-09]  | Avoid using abi.encodePacked() with dynamic types when passing the result to a hash function                                  | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [L-10]  | Missing address(0) checks in critical functions                                                                               | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-brightgreen) |
| [NC-01] | Missing events for state changing operations                                                                                  | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/Acknowledged-brightgreen)  |
| [NC-02] | Owner can renounce ownership                                                                                                  | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/Acknowledged-brightgreen)  |
| [NC-03] | Uninitialized implementation contracts in proxy pattern                                                                       | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/Acknowledged-brightgreen)  |
| [NC-04] | Unspecific pragma version                                                                                                     | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/Acknowledged-brightgreen)  |

# Findings

## [H-01] Some NFTs can easily get frozen in the protocol

### Impact

Loss of user's NFT tokens

### Proof of Concept

The protocol enables users to transfer NFTs from Layer 2 (L2) to Layer 1 (L1) by bridging them to the recipient's account. On the L1 side, once the challenge period and validation have passed, the protocol facilitates the transfer of the NFT to the user while relaying the message.

However, certain NFTs, such as CryptoKitties and CryptoFighters, may have a pause functionality. If a user attempts to use these types of NFTs and they are discovered by malicious actors during the paused state, these actors can exploit the `finalizeWithdrawalTransaction()` function to intentionally cause the transaction to fail. As a result, the user's NFT becomes permanently frozen within the contract.
Crypto-figher NFT:
https://etherscan.io/address/0x87d598064c736dd0C712D329aFCFAA0Ccc1921A1#code#L873

```solidity
function transferFrom(
	address _from,
	address _to,
	uint256 _tokenId
)
	public
	whenNotPaused
{
```

Crypto-kitty NFT:
https://etherscan.io/address/0x06012c8cf97BEaD5deAe237070F9587f8E7A266d#code#L615

```solidity
function transferFrom(
	address _from,
	address _to,
	uint256 _tokenId
)
	external
	whenNotPaused
{

```

Take a look at these scenarios

1. Alice owns NFTs like CryptoKitties and CryptoFighters and intends to bridge them from Layer 2 (L2) to Layer 1 (L1) using the protocol.
2. Bob, who is observing Alice's actions, becomes aware that she is initiating the bridging process for these specific NFTs. Although Alice's transaction is verified, she is waiting for the challenge period to conclude before finalizing it.
3. Coincidentally, the NFTs that Alice is attempting to bridge have been paused. After the challenge period elapses, Bob seizes the opportunity and calls the `finalizeWithdrawalTransaction()` function using Alice's transaction information as input.
4. Due to the pause state of the NFTs, the function call is unsuccessful. However, the finalization process does not handle failed calls, resulting in the transaction being completed without any error. Consequently, Alice's transaction is falsely marked as finalized, despite the fact that her NFTs are still paused. As a result, Alice loses her NFTs.
   [OptimismPortal.sol#LL398C1-L405C95](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/OptimismPortal.sol#L325-L420)

```solidity
   require(
            finalizedWithdrawals[withdrawalHash] == false,
            "OptimismPortal: withdrawal has already been finalized"
        );

        // Mark the withdrawal as finalized so it can't be replayed.
        finalizedWithdrawals[withdrawalHash] = true;

        // Set the l2Sender so contracts know who triggered this withdrawal on L2.
        l2Sender = _tx.sender;

        // Trigger the call to the target contract. We use a custom low level method
        // SafeCall.callWithMinGas to ensure two key properties
        //   1. Target contracts cannot force this call to run out of gas by returning a very large
        //      amount of data (and this is OK because we don't care about the returndata here).
        //   2. The amount of gas provided to the execution context of the target is at least the
        //      gas limit specified by the user. If there is not enough gas in the current context
        //      to accomplish this, `callWithMinGas` will revert.
        bool success = SafeCall.callWithMinGas(_tx.target, _tx.gasLimit, _tx.value, _tx.data);


```

Despite being reported in previous contest instances, the development team has not taken any action to address this issue or include it in the documentation to inform users about the risks involved.

### Recommended Mitigation Steps

It is highly recommended that the development team takes immediate action to mitigate this vulnerability. One possible solution is to implement a blacklist for the specific types of NFTs, such as CryptoKitties and CryptoFighters, which are susceptible to being paused. Alternatively, the team should clearly state in the documentation that users bear the responsibility for interacting with the contracts while the NFTs are in a paused state.

Prioritizing the security of the protocol is crucial, and addressing reported vulnerabilities promptly is essential to safeguard user assets. Furthermore, comprehensive documentation that outlines known vulnerabilities is necessary to ensure users are well-informed and can make educated decisions when utilizing the protocol.

## [H-02] Bridge transaction for NFTs could fail which leads to user's loss of NFT ownership

### Impact

In the current implementation of the NFT bridge contract, there's a potential for a transaction to fail, leading to a permanent loss of the NFT for the user. In L1/L2, if a transaction fails due to issues with the recipient address not properly implementing the `onERC721Received` interface or intentionally reverting the transaction, there is no mechanism to recover the lost NFT. Additionally, a malicious actor could potentially drain the caller's gas by invoking a guaranteed-to-fail function repetitively.

### Proof of Concept

The vulnerability is found in the `_initiateBridgeERC721()` amd `finalizeBridgeERC721()` functions in L1 and L2 contracts.
Take a look at [L1ERC721Bridge.\_initiateBridgeERC721():](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/L1ERC721Bridge.sol#L74-L106)

```
    function _initiateBridgeERC721(
        address _localToken,
        address _remoteToken,
        address _from,
        address _to,
        uint256 _tokenId,
        uint32 _minGasLimit,
        bytes calldata _extraData
    ) internal override {
        require(_remoteToken != address(0), "L1ERC721Bridge: remote token cannot be address(0)");

        // Construct calldata for _l2Token.finalizeBridgeERC721(_to, _tokenId)
        bytes memory message = abi.encodeWithSelector(
            L2ERC721Bridge.finalizeBridgeERC721.selector,
            _remoteToken,
            _localToken,
            _from,
            _to,
            _tokenId,
            _extraData
        );

        // Lock token into bridge
        deposits[_localToken][_remoteToken][_tokenId] = true;
        IERC721(_localToken).transferFrom(_from, address(this), _tokenId);

        // Send calldata into L2
        MESSENGER.sendMessage(OTHER_BRIDGE, message, _minGasLimit);
        emit ERC721BridgeInitiated(_localToken, _remoteToken, _from, _to, _tokenId, _extraData);
    }
```

the NFT is transferred to L2ERC721Bridge.sol

```
// Send calldata into L2
MESSENGER.sendMessage(OTHER_BRIDGE, message, _minGasLimit);
```

Then in L2, the [finalizeBridgeERC721()](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2/L2ERC721Bridge.sol#L33-L74)function below is called to settle the cross-chain NFT transfer.

```
    function finalizeBridgeERC721(
        address _localToken,
        address _remoteToken,
        address _from,
        address _to,
        uint256 _tokenId,
        bytes calldata _extraData
    ) external onlyOtherBridge {
        require(_localToken != address(this), "L2ERC721Bridge: local token cannot be self");

        // Note that supportsInterface makes a callback to the _localToken address which is user
        // provided.
        require(
            ERC165Checker.supportsInterface(_localToken, type(IOptimismMintableERC721).interfaceId),
            "L2ERC721Bridge: local token interface is not compliant"
        );

        require(
            _remoteToken == IOptimismMintableERC721(_localToken).remoteToken(),
            "L2ERC721Bridge: wrong remote token for Optimism Mintable ERC721 local token"
        );

        // When a deposit is finalized, we give the NFT with the same tokenId to the account
        // on L2. Note that safeMint makes a callback to the _to address which is user provided.
        IOptimismMintableERC721(_localToken).safeMint(_to, _tokenId);

        // slither-disable-next-line reentrancy-events
        emit ERC721BridgeFinalized(_localToken, _remoteToken, _from, _to, _tokenId, _extraData);
    }
```

Note that `safemint()` is called and since OptimismMintableERC721 inherits from Openzeppelin's ERC721 implementation, once `safeMint `is called, `_checkOnERC721Received` is called.

```
function _safeMint(address to, uint256 tokenId, bytes memory data) internal virtual {
	_mint(to, tokenId);
	require(
		_checkOnERC721Received(address(0), to, tokenId, data),
		"ERC721: transfer to non ERC721Receiver implementer"
	);
}

//@audit below is the implementation of _checkOnERC721Received()
function _checkOnERC721Received(
	address from,
	address to,
	uint256 tokenId,
	bytes memory data
) private returns (bool) {
	if (to.isContract()) {
		try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, data) returns (bytes4 retval) {
			return retval == IERC721Receiver.onERC721Received.selector;
		} catch (bytes memory reason) {
			if (reason.length == 0) {
				revert("ERC721: transfer to non ERC721Receiver implementer");
			} else {
				/// @solidity memory-safe-assembly
				assembly {
					revert(add(32, reason), mload(reason))
				}
			}
		}
	} else {
		return true;
	}
}

```

This function, which finalizes the transfer of NFTs between chains, can fail in several ways:

- The recipient address intentionally does not implement the `onERC721Received` interface, causing the transaction to revert.
- The recipient address returns a very large error message string, causing the transaction to run out of gas.
- The recipient address intentionally reverts the transaction during the external call to `onERC721Received`.
- Also the `_to` address should be checked not to be zero so as not to allow an unexpected burn of assets.

Furthermore, when a bridge is attempted for an NFT from L2 to L1, same assumptions needs to hold, because when we bridge the NFT and call, L2 NFT is already burned, if the settlement transaction revert in L1, the NFT is lost.

Since the `finalizeBridgeERC721` function does not keep track of transaction failures on-chain. This allows a malicious actor to keep invoking the function knowing it will fail, effectively draining the caller's wallet of gas.

### Recommended Mitigation Steps

To mitigate these vulnerabilities and improve the robustness of the NFT bridge contract, the following recommendations are made:

- Instead of using `safeMint` and `safeTransferFrom` functions that allow external calls, use `_mint` and `transferFrom` methods. These functions will avoid the need for the external call to the recipient address and mitigate the risks associated with it.
- Implement mechanisms to track if the `finalizeBridgeERC721` transaction has already failed. This can prevent the same transaction from being repeatedly called and help to avoid gas drainage.
- Implement a recovery stage for NFTs to avoid permanent loss of ownership in case of transaction failure. This could, for example, allow users to withdraw their NFT on L1 if the L2 minting process fails, and vice versa.

## [M-01] Current transfer flow in `L1ERC721Bridge.sol` provides a window for rewards/airdrops associated with locked NFTs to be stolen

### Impact

The vulnerability allows an attacker to exploit the L1ERC721Bridge contract and steal airdrops or rewards associated with locked NFTs. By taking advantage of the contract's transfer flow, since it firsts sets the value of the `deposits` mapping before attempting the transfer, an attacker can perform a flash loan attack to withdraw the NFTs and access their associated rewards without actually owning them. This exposes a significant risk to the integrity of the locked NFTs and the rewards they hold.

### Proof of Concept

The vulnerability in the L1ERC721Bridge contract arises from the incorrect order of operations in the `_initiateBridgeERC721()` function. It sets the value of the `deposits` mapping and then attempts to transfer the NFT with an external call. This results in an incorrect state during the external call, which can be exploited by an attacker who has knowledge of the NFT's hook function. The attacker can use this incorrect state to withdraw the NFT using `finalizeBridgeERC721()` and gain unauthorized access to airdrops or rewards associated with the NFT. Finally, the attacker returns the NFT in the remaining steps of `_initiateBridgeERC721()`.

Take a look at [L1ERC721Bridge.\_initiateBridgeERC721():](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/L1ERC721Bridge.sol#L74-L106)

```
    function _initiateBridgeERC721(
        address _localToken,
        address _remoteToken,
        address _from,
        address _to,
        uint256 _tokenId,
        uint32 _minGasLimit,
        bytes calldata _extraData
    ) internal override {
        require(_remoteToken != address(0), "L1ERC721Bridge: remote token cannot be address(0)");

        // Construct calldata for _l2Token.finalizeBridgeERC721(_to, _tokenId)
        bytes memory message = abi.encodeWithSelector(
            L2ERC721Bridge.finalizeBridgeERC721.selector,
            _remoteToken,
            _localToken,
            _from,
            _to,
            _tokenId,
            _extraData
        );

        // Lock token into bridge
        deposits[_localToken][_remoteToken][_tokenId] = true;
        IERC721(_localToken).transferFrom(_from, address(this), _tokenId);

        // Send calldata into L2
        MESSENGER.sendMessage(OTHER_BRIDGE, message, _minGasLimit);
        emit ERC721BridgeInitiated(_localToken, _remoteToken, _from, _to, _tokenId, _extraData);
    }
```

The vulnerability occurs because at [L100](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/L1ERC721Bridge.sol#L100) the code sets the `deposits[_localToken][_remoteToken][_tokenId]` value to true before transferring the NFT token. If the NFT token has a hook function that interacts with the `from` address (for example for checking transfer permission or custom NFT token in a project) then the execution would reach into from address and it can perform malicious actions as `deposits[_localToken][_remoteToken][_tokenId]` is true.

To exploit the vulnerability, an attacker could follow similar steps like thes:

1. User1 bridges NFT1 with ID=10 from L1 to L2 and receives a legitimate bridged NFT (NFT2) in return.
2. NFT1 (ID=10) is associated with rewards or airdrops in Project1, which can be claimed by calling `Project1.getReward()` while owning NFT1.
3. The attacker creates a malicious NFT (NFT3) in L2, implementing IOptimismMintableERC721 and returning NFT1's address when calling `IOptimismMintableERC721(_localToken).remoteToken()`. The attacker then mints NFT3 with ID=10 to their own address.
4. The attacker calls `L2ERC721Bridge.bridgeERC721(NFT3, NFT1, attackerContract, 10)`, triggering a message to `L1ERC721Bridge` to withdraw NFT1 for NFT3.
5. After the withdrawal message is processed by the L2 oracle and the withdrawal delay passes, the attacker performs the final attack:
   - The attacker creates a contract and calls `L1ERC721Bridge.bridgeERC721To(NFT1, NFT3, attackerAddress, 10)`.
   - In the process, the attacker sets `deposits[NFT1][NFT3][10] = true` and triggers the transfer of NFT1 from their address.
   - During the transfer, NFT1's hook function (in this case it exists) is called, invoking the attacker contract's function.
   - The attacker contract's hook function calls `OptimismPortal.finalizeWithdraw()` to finalize the withdrawal message sent from L2ERC721Bridge to L1ERC721Bridge.
   - L1ERC721Bridge receives the withdrawal message, checks the validity of `deposits[NFT1][NFT3][10]`, and transfers NFT1 to the attacker contract's address.
   - The attacker can now call `Project.getRewards()` and steal the rewards associated with NFT1 without being the rightful owner.
6. Eventually, L1ERC721Bridge transfers NFT1 from the attacker's address to the contract address.

By successfully executing this attack, the attacker gains access to locked NFTs in the L1 bridge as a flash loan, without having to pay any fees. Additionally, the attacker can steal rewards or airdrops associated with these NFTs, all without being the legitimate owner in L2 or having ownership rights to the bridged NFTs. It is important to note that this attack assumes the presence of a hook function in the NFT token when calling `safeTransferFrom()`, which allows the attacker to interact with the `from` address or another address under their control. While this assumption may not always hold true, this vulnerability poses a significant risk due to the general nature of the bridge, which supports various NFTs from different projects and accommodates diverse user and token types. Exploiting this vulnerability, an attacker can target multiple locked NFTs and steal a considerable amount of rewards and airdrops.

### Recommended Mitigation Steps

To address this vulnerability, it is recommended to modify the `_initiateBridgeERC721` function in the L1ERC721Bridge.
This pattern involves transferring the NFT token first and then setting the value of the `deposits` mapping.

The revised code for `_initiateBridgeERC721` would look like this:

```
    function _initiateBridgeERC721(
        address _localToken,
        address _remoteToken,
        address _from,
        address _to,
        uint256 _tokenId,
        uint32 _minGasLimit,
        bytes calldata _extraData
    ) internal override {
        require(_remoteToken != address(0), "L1ERC721Bridge: remote token cannot be address(0)");

        // Construct calldata for _l2Token.finalizeBridgeERC721(_to, _tokenId)
        bytes memory message = abi.encodeWithSelector(
            L2ERC721Bridge.finalizeBridgeERC721.selector,
            _remoteToken,
            _localToken,
            _from,
            _to,
            _tokenId,
            _extraData
        );

        // Lock token into bridge
        IERC721(_localToken).transferFrom(_from, address(this), _tokenId);
        deposits[_localToken][_remoteToken][_tokenId] = true;

        // Send calldata into L2
        MESSENGER.sendMessage(OTHER_BRIDGE, message, _minGasLimit);
        emit ERC721BridgeInitiated(_localToken, _remoteToken, _from, _to, _tokenId, _extraData);
    }
```

## [M-02] Current implementation of `bridgeERC721To()` on L1/L2 could cost users their NFTs

### Impact

In short users' ERC721 tokens will become permanently locked within the bridge cause NFTs will be frozen if receiver does not implement onERC721Received

NB: There can be multiple scenarios where ERC721 tokens are sent to contracts without the onERC721Received implementation. For instance:

1. Say on L1, Alice possesses a smart wallet, multisig, or vault that holds the NFTs but does not implement onERC721Received. She sends the NFTs from her smart wallet/multisig/vault to her EOA (Externally Owned Address) on L2 using `bridgeERC721To` to interact with L2 protocols.
2. Alice's EOA on L2 decides to send the NFTs back to her smart wallet/multisig/vault using `bridgeERC721To`
3. The NFT will be locked on the L1 bridge because the bridge cannot send transfer to her smart wallet/multisig/vault

The bridge protocol allows the transfer of ERC721 tokens to non-EOA (Externally Owned Address) addresses. However, if the recipient contracts do not implement the onERC721Received function, the ERC721 tokens will become locked within the bridge. Unfortunately, there is also no warning or information provided about this behavior in the protocol's documentation or code.

### Proof of Concept

The bridge protocol includes the bridgeERC721To function, which allows users to send ERC721 tokens to addresses on either L1 or L2. The NFTs are sent from the bridge to the corresponding layer and are either minted or transferred to the specified `_to` address.

If the `_to` address is a contract without an implementation of the onERC721Received function, the ERC721 tokens will become permanently locked within the bridge.

[On L1 within the L1ERC721Bridge contract:](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/L1ERC721Bridge.sol#L33-L72)

```solidity
    function finalizeBridgeERC721(
        address _to,
        uint256 _tokenId,
        bytes calldata _extraData
    ) external onlyOtherBridge {
        IERC721(_localToken).safeTransferFrom(address(this), _to, _tokenId);
    }
```

[On L2 within the L2ERC721Bridge contract:](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2/L2ERC721Bridge.sol#L33-L74)

```solidity
    function finalizeBridgeERC721(
        address _to,
        uint256 _tokenId,
        bytes calldata _extraData
    ) external onlyOtherBridge {
        IOptimismMintableERC721(_localToken).safeMint(_to, _tokenId);
    }
```

Both safeMint and safeTransferFrom functions utilize OpenZeppelin's \_checkOnERC721Received function, which reverts the transaction if the target address is a contract without the implementation of onERC721Received.

```solidity
    function _checkOnERC721Received(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) private returns (bool) {
        if (to.isContract()) {
            try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, data) returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch (bytes memory reason) {
                if (reason.length == 0) {
                    revert("ERC721: transfer to non ERC721Receiver implementer");
                } else {
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }
            }
        } else {
            return true;
        }
    }
```

Although OpenZeppelin has implemented safeMint and safeTransferFrom to prevent user errors when transferring to contracts that cannot handle ERC721 tokens, in this case, the revert does not prevent the user error but instead locks the NFTs within the bridge.

Therefore, a safer and more reasonable approach would be to "force" the transfer/minting of the NFTs.

### Recommended Mitigation Steps

While validating the recipient of the NFT on the other layer can be complex, there are potential changes that can help address this vulnerability.

One approach is to modify the bridge's implementation by replacing the calls to `safeMint` or `safeTransferFrom` with `mint` and `transferFrom` functions. This change would "force" the transfer of the token. After the transfer, an attempt can be made to call the `onERC721Received` function of the recipient using the following example code:

```solidity
if (Address.isContract(_to)) {
    try IERC721Receiver(_to).onERC721Received(address(this), _from, _tokenId, "") returns (bytes4 retval) {
    } catch (bytes memory reason) {}
}
```

Another possible solution is to add the `_to` address to the ERC721 token approvals and require the caller to "pull" the NFT from the bridge.

Implementing either of these approaches will help mitigate the vulnerability by ensuring that the NFTs are successfully transferred or minted, even if the recipient contract does not implement the `onERC721Received` function, another little hint would be to implement a mechanism to forward the NFT back to the users if the transfer failed.

## [M-03] Optimism.sol: Inaccurate identification of EOA and smart contract callers in `depositTransaction()` function

### Impact

The `depositTransaction()` function relies on the condition `msg.sender != tx.origin` to determine if the caller is a contract or an externally owned account (EOA). However, according to [EIP-3074](https://eips.ethereum.org/EIPS/eip-3074), using `msg.sender != tx.origin` as a means to differentiate between contracts and EOAs may no longer work as expected, due to the introduction of the `AUTH` and `AUTHCALL` EVM instructions opcodes. This can lead to incorrect identification of the caller's type, potentially impacting the logic of the contract since an attempt would be made to transform the from-address to its alias if the caller is a contract, where as `msg.sender` is not a contract in this instance.

TLDR: Using the `msg.sender != tx.origin` check to determine if calls are made from EOA or smart contracts will no longer work as expected and an incorrect attempt would be made to transform the from-address to its alias.

### Proof of Concept

Take a look at the [depositTransaction():](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/OptimismPortal.sol#L434-L483) function in the code snippet:

```solidity
function depositTransaction(
    address _to,
    uint256 _value,
    uint64 _gasLimit,
    bool _isCreation,
    bytes memory _data
) public payable metered(_gasLimit) {
    // Just to be safe, make sure that people specify address(0) as the target when doing
    // contract creations.
    if (_isCreation) {
        require(
            _to == address(0),
            "OptimismPortal: must send to address(0) when creating a contract"
        );
    }

    // Prevent depositing transactions that have too small of a gas limit. Users should pay
    // more for more resource usage.
    require(
        _gasLimit >= minimumGasLimit(uint64(_data.length)),
        "OptimismPortal: gas limit too small"
    );

    // Prevent the creation of deposit transactions that have too much calldata. This gives an
    // upper limit on the size of unsafe blocks over the p2p network. 120kb is chosen to ensure
    // that the transaction can fit into the p2p network policy of 128kb even though deposit
    // transactions are not gossipped over the p2p network.
    require(_data.length <= 120_000, "OptimismPortal: data too large");

    // Transform the from-address to its alias if the caller is a contract.
    address from = msg.sender;
    if (msg.sender != tx.origin) {
        from = AddressAliasHelper.applyL1ToL2Alias(msg.sender);
    }

    // Compute the opaque data that will be emitted as part of the TransactionDeposited event.
    // We use opaque data so that we can update the TransactionDeposited event in the future
    // without breaking the current interface.
    bytes memory opaqueData = abi.encodePacked(
        msg.value,
        _value,
        _gasLimit,
        _isCreation,
        _data
    );

    // Emit a TransactionDeposited event so that the rollup node can derive a deposit
    // transaction for this deposit.
    emit TransactionDeposited(from, _to, DEPOSIT_VERSION, opaqueData);
}
```

The vulnerable aspect of the code is the reliance on `msg.sender != tx.origin` to differentiate between an externally owned account (EOA) and a contract. However, with the introduction of EIP-3074, this differentiation method may no longer hold true. EIP-3074 allows users to sign a message with their private keys, which includes their intent, and include it in a transaction on-chain that calls an invoker. The invoker then utilizes the signed message along with the `AUTH` opcode to gain control over the user's account and can perform actions on behalf of the user using `AUTHCALL`. It is important to note that the transaction containing the user's signed message does not necessarily need to be sent from the user's account itself. This is just one of the possibilities enabled by EIP-3074.

Therefore, relying solely on `msg.sender != tx.origin` to determine the type of the caller can lead to inaccurate identification and potentially result in incorrect execution of the contract's logic.

For more information about the possibilities EIP-3074 brings in regards to users, we would recommend you go through this great article by [Ismael Darwish](https://medium.com/nethermind-eth/ethereum-wallets-today-and-tomorrow-eip-3074-vs-erc-4337-a7732b81efc8) diving deep into Ethereum wallets most specifically `EIP-3074 & ERC-4337`

### Recommended Mitigation Steps

To address the potential issue identified with `msg.sender != tx.origin`, it is recommended to explore alternative approaches for distinguishing between contracts and EOAs. One possible solution is to utilize OpenZeppelin's `Address` library and its `isContract()` function, which can be found in the OpenZeppelin documentation.

To implement this recommendation, follow these steps:

1. Import the `Address` library from OpenZeppelin:

   ```solidity
   import "@openzeppelin/contracts/utils/Address.sol";
   ```

2. Modify the `depositTransaction()` function as follows:

   ```solidity
   function depositTransaction(
       address _to,
       uint256 _value,
       uint64 _gasLimit,
       bool _isCreation,
       bytes memory _data
   ) public payable metered(_gasLimit) {
       // Just to be safe, make sure that people specify address(0) as the target when doing
       // contract creations.
       if (_isCreation) {
           require(
               _to == address(0),
               "OptimismPortal: must send to address(0) when creating a contract"
           );
       }

       // Prevent depositing transactions that have too small of a gas limit. Users should pay
       // more for more resource usage.
       require(
           _gasLimit >= minimumGasLimit(uint64(_data.length)),
           "OptimismPortal: gas limit too small"
       );

       // Prevent the creation of deposit transactions that have too much calldata. This gives an
       // upper limit on the size of unsafe blocks over the p2p network. 120kb is chosen to ensure
       // that the transaction can fit into the p2p network policy of 128kb even though deposit
       // transactions are not gossiped over the p2p network.
       require(_data.length <= 120_000, "OptimismPortal: data too large");

       // Transform the from-address to its alias if the caller is a contract.
       address from = msg.sender;
       //@audit use of isContract() instead
       if (Address.isContract(msg.sender)) {
           from = AddressAliasHelper.applyL1ToL2Alias(msg.sender);
       }

       // Compute the opaque data that will be emitted as part of the TransactionDeposited event.
       // We use opaque data so that we can update the TransactionDeposited event in the future
       // without breaking the current interface.
       bytes memory opaqueData = abi.encodePacked(
           msg.value,
           _value,
           _gasLimit,
           _isCreation,
           _data
       );

       // Emit a TransactionDeposited event so that the rollup node can derive a deposit
       // transaction for this deposit.
       emit TransactionDeposited(from, _to, DEPOSIT_VERSION, opaqueData);
   }
   ```

By using OpenZeppelin's `Address.isContract()` function, you can reliably determine whether the caller is an EOA or a contract. This mitigates the potential vulnerability associated with relying on `msg.sender != tx.origin` and ensures the correct execution of the contract logic based on the caller's type.

Please note that when using `Address.isContract()`, there are certain edge cases to consider, such as calling it from a constructor, which may have limitations. Exercise caution and perform thorough testing when implementing this solution.

## [M-04] Using `selfdestruct` as the ETH burning mechanism in `L2ToL1MessagePasser` is a downside

### Impact

Key to Note that there are two sides to report.

- First is regarding the ETH burning mechanism in `L2ToL1MessagePasser`
- Secondly, the `SelfDestruction.sol's` `destruct()` function is now Ineffective

As of Cancun ([EIP 6780](https://eips.ethereum.org/EIPS/eip-6780)) the original behaviour of `SELFDESTRUCT` has been changed and will now only work within the same tx. Which means that if the attempt to burn ETH from` L2ToL1MessagePasser` is not done in one transaction it fails. Most importantly this means that the notion of deactivating `SELFDESTRUCT` (EIP-4758) is getting [closer by the day](https://www.youtube.com/watch?v=Mgld_3JjFXQ&t=219s&ab_channel=EthereumCatHerders).

Note that After the [EIP-4758](https://eips.ethereum.org/EIPS/eip-4758) fork, the ETH burning mechanism in the L2ToL1MessagePasser contract will no longer work as expected. This means that when ETH is withdrawn from L2, the amount of ETH in the L2 circulating supply will not be reduced as intended. This could lead to an inflated amount of ETH on L2.

For the second case this also renders the `destruct()` function in the `SelfDestruction.sol` contract completely ineffective. The purpose of the function is to forward the contract's balance to itself and then destroy the contract. However, after the implementation of EIP-6780, the function only transfers the balance to itself without actually destroying the contract.

### Proof of Concept

- First Case:

The burning of ETH on L2 is currently performed using the selfdestruct function in the L2ToL1MessagePasser contract. However, with the deactivation of the SELFDESTRUCT opcode after the EIP-4758 fork, this mechanism will no longer be effective. The code snippets below illustrate the relevant portions of the code:

Snippet from [L2ToL1MessagePasser.sol:](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2/L2ToL1MessagePasser.sol#L85-L89)

```solidity
function burn() external {
    uint256 balance = address(this).balance;
    Burn.eth(balance);
    emit WithdrawerBalanceBurnt(balance);
}
```

Snippet from [Burn.sol:](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/libraries/Burn.sol#L8-L42)

```solidity
function eth(uint256 _amount) internal {
    new Burner{ value: _amount }();
}

contract Burner {
    constructor() payable {
        selfdestruct(payable(address(this)));
    }
}
```

- Second Case:
  Take a look at [SelfDestruction.sol:](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/integration-tests/contracts/SelfDestruction.sol)

```

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

contract SelfDestruction {
bytes32 public data = 0x0000000000000000000000000000000000000000000000000000000061626364;

    function setData(bytes32 _data) public {
        data = _data;
    }

    function destruct() public {
        address payable self = payable(address(this));
        selfdestruct(self);
    }

}

```

The above contract contains the following functions:

- `setData`: Intended to allow the contract owner to set the value of the data variable.
- `destruct`: Intended to initiate the self-destruct process, destroying the contract and transferring any remaining Ether to the designated address (self).

The first still works as planned, but the second case no longer works as intended cause the call to `destruct()` is not made in the same transaction with the creation of the contract so the intended implementation of performing a powerful and irreversible operation of destroying the contract after execution would fail and rather just send it's balance to itself instead

If an implementation of `selfdestruct` (which is not recommended since it'd soon be completely deprecated) is still going to be continued then we recomment you read [this thread](https://twitter.com/pcaversaccio/status/1662381154526195714?s=12&t=ySKwes13sImLKPK3DJ-8TA) to find out what changes have occured and how to mitigate these changes.

### Recommended Mitigation Steps

Considering the new implementation of the SELFDESTRUCT opcode, an alternative method chould be implemented to reduce the supply of ETH on L2 after withdrawal. Try to explore other approaches within the OVM to handle the reduction of ETH in a different manner. This will ensure the proper functioning of the ETH burning mechanism and prevent the inflation of ETH on L2.

## [L-01] Critical address changes should always use a two-step procedure

### Vulnerability Details

OpenZepplin's Ownable provides a One-Step process for Transfer Ownership of the contract. Critical Address Changes Should always be Two-step Procedure.

The lack of a two-step procedure for critical operations leaves them error-prone. A One-Step Transfer OwnerShip procedure can fall victim to accidentally inputting the wrong future owner's address, resulting in loss of contract's ownership forever.

[CrossDomainOwnable.sol#L1-L6](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2/CrossDomainOwnable.sol#L1-L5)

### Recommendation

A two step approach should be instead implemented as this saves a ton incase a mistake is made while transferring ownership

There is another Openzeppelin Ownable contract, `Ownable2Step.sol`, which provides a two-step procedure for transfer ownership. `transferOwnership()` & `acceptOwnership()`, this could be inherited insteasd.

## [L-02] L1ERC721Bridge.sol: `safeTransferFrom()` should be used in the place of `transferFrom()`

### Vulenerability Details

Note that both ERC721 transferFrom method and ERC20 transferFrom method has the same byte4 signature 0x23b872dd

This means that If localToken is not ERC721 but a ERC20, transfer can still go through and the ERC20 is mistakenly bridged from L1 to L2.

However, the token cannot be bridged back, because when bridged back, the function below called:

[L1ERC721Bridge.sol#L100-L107](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/L1ERC721Bridge.sol#L100-L107)

```solidity
        deposits[_localToken][_remoteToken][_tokenId] = true;
        IERC721(_localToken).transferFrom(_from, address(this), _tokenId);

        // Send calldata into L2
        MESSENGER.sendMessage(OTHER_BRIDGE, message, _minGasLimit);
        emit ERC721BridgeInitiated(_localToken, _remoteToken, _from, _to, _tokenId, _extraData);
    }
```

Also The recipient could have logic in the onERC721Received() function, which is only triggered in the safeTransferFrom() function and not in transferFrom(). A notable example of such contracts is the Sudoswap pair:

```
function onERC721Received(
  address,
  address,
  uint256 id,
  bytes memory
) public virtual returns (bytes4) {
  IERC721 _nft = nft();
  // If it's from the pair's NFT, add the ID to ID set
  if (msg.sender == address(_nft)) {
    idSet.add(id);
  }
  return this.onERC721Received.selector;
}
```

### Recommendation

It is recommended to use safeTransferFrom() instead of transferFrom() when transferring ERC721s to hinder the aforementioned issues

```diff
- IERC721(_localToken).transferFrom(_from, address(this), _tokenId);
+ IERC721(_localToken).safeTransferFrom(_from, address(this), _tokenId);
```

## [L-03] Modifiers should only be used for checks

### Vulnerability Details

In [ResourceMetering.sol](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/ResourceMetering.sol) the [metered](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/ResourceMetering.sol#L75-L84) modifier used for more than just for checks

```
    modifier metered(uint64 _amount) {
        // Record initial gas amount so we can refund for it later.
        uint256 initialGas = gasleft();

        // Run the underlying function.
        _;

        // Run the metering function.
        _metered(_amount, initialGas);
    }
```

See [this](https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/), but in short it's known that the code inside a modifier is usually executed before the function body, so any state changes or external calls will violate the Checks-Effects-Interactions pattern. Moreover, these statements may also remain unnoticed by the developer, as the code for the modifier may be far from the function declaration.

### Recommendation

Try to use modifiers only for checks

## [L-04] L1ERC721Bridge does not comply with ERC721, breaking composability

### Velnerability Details

Not implementing the `ERC721Receiver` interface in the `L1ERC721Bridge` contract while holding ERC721 tokens is considered an anti-pattern and goes against the desired standard for handling ERC721 tokens.

The contract [L1ERC721Bridge](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/L1ERC721Bridge.sol#L100-L107) should implement the `ERC721Receiver` interface to ensure compatibility with the ERC721 token standard.

### Recommendation

To address this vulnerability, it is recommended to modify the `ERC721Bridge` contract to implement the `ERC721Receiver` interface. This will ensure compliance with the ERC721 token standard and align with best practices for handling ERC721 tokens.

## [L-05] Using vulnerable dependency version of OpenZeppelin

### Vulnerability Details

The [package.json](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/package.json#L57-L58) configuration file says that the project is using 4.7.3 of OZ which is not the latest update version:

```json
    "@openzeppelin/contracts": "4.7.3",
    "@openzeppelin/contracts-upgradeable": "4.7.3",

```

### Recommendation

Upgrade the dependencies to their latest counterparts

## [L-06] Use of `assert()` instead of `require()`

### Vulnerability Details

Contracts use assert() instead of require() in multiple places. This causes a Panic error on failure and prevents the use of error strings.

Prior to solc 0.8.0, assert() used the invalid opcode which used up all the remaining gas while require() used the revert opcode which refunded the gas and therefore the importance of using require() instead of assert() was greater. However, after 0.8.0, assert() uses revert opcode just like require() but creates a Panic(uint256) error instead of Error(string) created by require(). Nevertheless, Solidity’s documentation says:

"Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Language analysis tools can evaluate your contract to identify the conditions and function calls which will cause a Panic.”

whereas

“The require function either creates an error without any data or an error of type Error(string). It should be used to ensure valid conditions that cannot be detected until execution time. This includes conditions on inputs or return values from calls to external contracts.”

Also, you can optionally provide a message string for require, but not for assert.

Take a look at [CrossDomainMessenger.sol#L341-L342](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/universal/CrossDomainMessenger.sol#L341-L342)

```
            assert(msg.value == _value);
            assert(!failedMessages[versionedHash]);
```

Now also look at this instance from [ProxyAdmin.sol](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/universal/ProxyAdmin.sol#L227)

```
            assert(false);
```

### Recommendation

Use require() with informative error strings instead of assert().

## [L-07] Known issues with compiler versions used to compile the contracts (sol 0.8.15 & 0.6.0)

### Vulnerability Details

The contracts in scope have been compiled using older versions of the Solidity compiler (specifically, up to version 0.8.15). However, there are known issues that have been addressed in the latest compiler versions (> 0.8.15). Unfortunately, these addressed issues are still present in the 0.8.15 version used by the contracts.

One known issue relates to the `abi.encode` family of functions, which can lead to vulnerabilities when certain conditions are met. The contracts in question fulfill the criteria for being vulnerable to this issue. For more technical details about this issue, you can refer to the following link: [Solidity Blog Post](https://blog.soliditylang.org/2022/08/08/calldata-tuple-reencoding-head-overflow-bug/).

Additionally, another known issue is specific to the 0.6.0 compiler version. This version is used in the `oracle.sol` contract from `op-geth`. The issue arises when attempting to push an empty admin address (zero) in a function call. It occurs due to how the admin list is loaded into memory and stored in storage. More information about this issue can be found here: [Solidity Blog Post](https://blog.soliditylang.org/2022/06/15/dirty-bytes-array-to-storage-bug/).

### Recommendation

To mitigate these known issues and benefit from the improvements and bug fixes in the Solidity compiler, it is strongly recommended to upgrade the compiler version to either 0.8.16 or 0.8.17 for all the contracts.

By using the latest compiler version, the contracts can avoid potential vulnerabilities and ensure better code quality. Updating the compiler version is an essential step in maintaining the security and reliability of the contracts.

It is also advisable to regularly check and follow the Solidity documentation for any updates, security recommendations, and bug fixes related to the compiler version being used.

Links to the affected code snippets and the Solidity documentation for more information:

Solidity Bugs Documentation: [Solidity Documentation](https://docs.soliditylang.org/en/v0.8.18/bugs.html)

By addressing these compiler-related vulnerabilities and staying up-to-date with the latest Solidity compiler version, the contracts can enhance their security and maintain compliance with best practices.

Additionally, using a solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>)

## [L-08] Low level function call does not check for contract existence

### Vulnerability Details

The safeCall.call() is being implemented in multiple instances in code

Here is the current implementation of the call from the SafeCall library

```
library SafeCall {
    /**
     * @notice Perform a low level call without copying any returndata
     *
     * @param _target   Address to call
     * @param _gas      Amount of gas to pass to the call
     * @param _value    Amount of value to pass to the call
     * @param _calldata Calldata to pass to the call
     */
    function call(
        address _target,
        uint256 _gas,
        uint256 _value,
        bytes memory _calldata
    ) internal returns (bool) {
        bool _success;
        assembly {
            _success := call(
                _gas, // gas
                _target, // recipient
                _value, // ether value
                add(_calldata, 0x20), // inloc
                mload(_calldata), // inlen
                0, // outloc
                0 // outlen
            )
        }
        return _success;
    }
}
```

As seen this call does not check for account's existence prior to interacting with account

Note that this low level is performed in OptimismPortal when a withdrawal transaction is to be finalized

```
    function finalizeWithdrawalTransaction(Types.WithdrawalTransaction memory _tx) external {
....
// the above function  calls the beloe
....
// Trigger the call to the target contract. We use SafeCall because we don't
// care about the returndata and we don't want target contracts to be able to force this
// call to run out of gas via a returndata bomb.
bool success = SafeCall.call(
	_tx.target,
	gasleft() - FINALIZE_GAS_BUFFER,
	_tx.value,
	_tx.data
);
```

Do keep in mind that the same function is used in both of `relayMessage()` and `finalizeBridgeETH()` The low level is used in CrossDomainMessager.sol when relay the message.

```
function relayMessage(
	uint256 _nonce,
	address _sender,
	address _target,
	uint256 _value,
	uint256 _minGasLimit,
	bytes calldata _message
) external payable nonReentrant whenNotPaused {
  .....
    // the above calls
.....

xDomainMsgSender = _sender;
bool success = SafeCall.call(_target, gasleft() - RELAY_GAS_BUFFER, _value, _message);
xDomainMsgSender = Constants.DEFAULT_L2_SENDER;

****

And finally, the low level call is used in  the finalizeBridgeETH() too

function finalizeBridgeETH(
	address _from,
	address _to,
	uint256 _amount,
	bytes calldata _extraData
) public payable onlyOtherBridge {
	require(msg.value == _amount, "StandardBridge: amount sent does not match amount required");
	require(_to != address(this), "StandardBridge: cannot send to self");
	require(_to != address(MESSENGER), "StandardBridge: cannot send to messenger");

	emit ETHBridgeFinalized(_from, _to, _amount, _extraData);

	bool success = SafeCall.call(_to, gasleft(), _amount, hex"");
	require(success, "StandardBridge: ETH transfer failed");
}
```

Whenever the function call is a smart contract call with call data and ETH attached if the target address to call does not exist, the transaction should instead fail, but it sliently goes through.

### Recommendation

If the function call is a smart contract call with call data attached, check the contract existense before executing the function, to not let the transaction that expect to fail sliently execute.

## [L-09] Avoid using abi.encodePacked() with dynamic types when passing the result to a hash function

### Vulnerability Details

Take a look at [Oracle.sol#L99](https://github.com/ethereum-optimism/op-geth/blob/bdab05ca786fddaa81c60cbd4ee06df3b2585fa3/contracts/checkpointoracle/contract/oracle.sol#L99)

```
bytes32 signedHash = keccak256(abi.encodePacked(byte(0x19), byte(0), this, _sectionIndex, _hash));
```

Using `abi.encodePacked()` with dynamic types when passing the result to a hash function can lead to potential hash collisions. The `abi.encode()` function pads items to 32 bytes, ensuring better collision prevention. It is important to note that if there is only one parameter, it is possible to cast it to `bytes()` or `bytes32()` instead of using `abi.encodePacked()`. When dealing with multiple parameters that are strings or bytes, it is recommended to use `bytes.concat()`.
Key to also note that `abi.encodePacked()` is deprecated in newest solidity versions

### Recommendation

To prevent hash collisions, it is recommended to avoid using `abi.encodePacked()` with dynamic types when passing the result to a hash function. Instead, use `abi.encode()` which pads items to 32 bytes, ensuring better collision prevention. If there is only one parameter, consider casting it to `bytes()` or `bytes32()` in place of `abi.encodePacked()`. Additionally, when dealing with multiple parameters that are strings or bytes, use `bytes.concat()` to concatenate them before passing them to the hash function. By following these recommendations, you can enhance the integrity and security of the hash-based operations in your contract.

## [L-10] Missing address(0) checks in critical functions

### Vulenerability Details

Most functions do not check whether input parameter is address(0).

Look at the instance below:

```solidity
    function initiateWithdrawal(
        address _target,
        uint256 _gasLimit,
        bytes memory _data
    ) public payable {
        bytes32 withdrawalHash = Hashing.hashWithdrawal(
            Types.WithdrawalTransaction({
                nonce: messageNonce(),
                sender: msg.sender,
                target: _target,
                value: msg.value,
                gasLimit: _gasLimit,
                data: _data
            })
        );
```

And a lot of instances in multiple contracts in spec.

### Recommendation

Include zero address checks

## [NC-01] Missing events for state changing operations

### Vulenerability Details

Take a look at
[L1Block.setL1BlockValues()](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2/L1Block.sol#L79-L103)

```
    function setL1BlockValues(
        uint64 _number,
        uint64 _timestamp,
        uint256 _basefee,
        bytes32 _hash,
        uint64 _sequenceNumber,
        bytes32 _batcherHash,
        uint256 _l1FeeOverhead,
        uint256 _l1FeeScalar
    ) external {
        require(
            msg.sender == DEPOSITOR_ACCOUNT,
            "L1Block: only the depositor account can set L1 block values"
        );

        number = _number;
        timestamp = _timestamp;
        basefee = _basefee;
        hash = _hash;
        sequenceNumber = _sequenceNumber;
        batcherHash = _batcherHash;
        l1FeeOverhead = _l1FeeOverhead;
        l1FeeScalar = _l1FeeScalar;
    }
```

### Recommendation

Consider emiting an event.

## [NC-02] Owner can renounce ownership

### Vulnerability Details

The non-fungible OwnableUpgradeable and Ownable used in several project contracts inherit/implement renounceOwnership(). This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

[From CrossDomainOwnable2.sol:](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2/CrossDomainOwnable2.sol#L6)
`import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol"; `

[From CrossDomainOwnable.sol:](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L2/CrossDomainOwnable.sol#L6)
`import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol"; `

[From SystemConfig.sol](https://github.com/ethereum-optimism/optimism/blob/382d38b7d45bcbf73cb5e1e3f28cbd45d24e8a59/packages/contracts-bedrock/contracts/L1/SystemConfig.sol#LL4C1-L6C76)

```
import {
    OwnableUpgradeable
} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```

### Recommendation

Consider re-implementing the function to disable it or clearly specifying if it is part of the contract design.

## [NC-03] Uninitialized implementation contracts in proxy pattern

### Vulnerability Details

Almost all of the contract in the protocol are based on proxy pattern to be upgradable but some of the implementation contract are not initialized. attacker can become owner of this contract and perform malicious actions. even if there are any clear malicious action right now with every update or change the risk of the bad action by owner of the implementation contract would be there. contracts SystemDictator is not initialized.

### Recommendation

Constructors should include the implementation of `intialize()`

## [NC-04] Unspecific pragma version

### Vulnerability Details

Check While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different evm compilation that is ultimately deployed on the blockchain.

From [CrossDomainOwnable#L2]():
`pragma solidity ^0.8.0;`

### Recommendation

Consider locking the pragma verison.
