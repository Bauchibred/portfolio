# Introduction

A time-boxed security review of the **Beedle** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Beedle](#About-Beedle)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://www.codehawks.com/contests/clkbo1fa20009jr08nyyf9wbx). The competition spanned **14** days, from **_July 24th, 2023_** to **_August 7th, 2023_**, and was hosted on [CodeHawks](https://www.codehawks.com). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/Cyfrin/2023-07-beedle/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/Cyfrin/2023-07-beedle/commit/658e046bda8b010a5b82d2d85e824f3823602d27)and primarily covered the following contracts:

```
src/
├── [Beedle.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/Beedle.sol)
├── [Fees.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/Fees.sol)
├── [Lender.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/Lender.sol)
├── [Staking.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/Staking.sol)
├── interfaces
│   ├── [IERC20.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/interfaces/IERC20.sol)
│   └── [ISwapRouter.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/interfaces/ISwapRouter.sol)
└── utils
    ├── [Errors.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/utils/Errors.sol)
    ├── [Ownable.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/utils/Ownable.sol)
    └── [Structs.sol](https://github.com/Cyfrin/2023-07-beedle/blob/main/src/utils/Structs.sol)
```

~ 706 NSLOC in 9 files

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Beedle

An ecosystem of DeFi primitives that includes an oracle free peer to peer perpetual lending.

- Twitter: https://twitter.com/beedlefi

# Summary

This report contains **1 high severity** issues, **4 medium** severity issues and **8 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #       | Title                                                                                                                                                       | Severity                                         | Status                                                  |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [H-01]  | `amountOutMinimum` being hardcoded to 0 makes swaps vulnerable to sandwich attacks                                                                          | ![](https://img.shields.io/badge/-High-red)      | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01]  | First depositor would encounter unexpected behaviours and might not receive any rewards                                                                     | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02]  | Centralisation risks: The owner is a single point of failure                                                                                                | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03]  | Fees.sol: `sqrtPriceLimitX96` being hardcoded to 0 is not ideal                                                                                             | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-04]  | Fee on transfer tokens will not behave as expected                                                                                                          | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01]  | Pragma isn't specified correctly which can lead to nonfunction/damaged contract when deployed on Arbitrum                                                   | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-02]  | Lack of Two-Step Role Transfer in Ownable.sol                                                                                                               | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-03]  | Array length mismatch in the `giveLoan` Function                                                                                                            | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-04]  | ERC20 return values not checked                                                                                                                             | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-05]  | Lack of zero-address validation in various functions                                                                                                        | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [NC-01] | Lender.sol: The error being `PoolConfig` in most cases is completely a downside of protocol as users can't know the reasons to why their transaction failed | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [NC-02] | Unnecessary `_diff` check in the `update()` function                                                                                                        | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [NC-03] | Setter functions should have equality checks                                                                                                                | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [H-01] `amountOutMinimum` being hardcoded to 0 makes swaps vulnerable to sandwich attacks

https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Fees.sol#L38

### Summary

The sellProfits function lacks slippage protection, making it vulnerable to potential sandwich attacks which could lead to financial losses for swappers

### Vulnerability Details

Take a look at [Fees.sol#L24-L45](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Fees.sol#L24-L45)

```solidity
    /// @notice swap loan tokens for collateral tokens from liquidations
    /// @param _profits the token to swap for WETH
    function sellProfits(address _profits) public {
        require(_profits != WETH, "not allowed");
        uint256 amount = IERC20(_profits).balanceOf(address(this));


        ISwapRouter.ExactInputSingleParams memory params = ISwapRouter
            .ExactInputSingleParams({
                tokenIn: _profits,
                tokenOut: WETH,
                fee: 3000,
                recipient: address(this),
                deadline: block.timestamp,
                amountIn: amount,
                //@audit
                amountOutMinimum: 0,
                sqrtPriceLimitX96: 0
            });


        amount = swapRouter.exactInputSingle(params);
        IERC20(WETH).transfer(staking, IERC20(WETH).balanceOf(address(this)));
    }
}
```

As seen the `sellProfits()` function is designed to swap loan tokens for collateral tokens from liquidations. The function uses ISwapRouter's exactInputSingle method to make this swap.

However, the function does not enforce slippage protection. This is evident from the fact that `amountOutMinimum` is set to zero which exposes the function to sandwich attacks-> financial loss.

### Impact

Significant financial losses whenever a swap gets sandwiched.

### Recommended Mitigation

Implement a mechanism to enforce slippage protection in the sellProfits function. This can be done by requesting a non-zero minimum output amount (amountOutMinimum)

## [M-01] First depositor would encounter unexpected behaviours and might not receive any rewards

### Summary

In the provided Staking contract's `deposit` function, the sequence of operations results in the first depositor being unable to receive any rewards. The issue arises due to the order of operations in the `deposit` function, where the `updateFor` function is called before the user's balance is updated. This, in combination with conditions in the `update` and `updateFor` functions, results in the first depositor's shares not being added.

### Vulnerability Details

In the `deposit` function, the contract calls `updateFor(msg.sender)` before adding the deposited amount to `balances[msg.sender]`. The `updateFor` function, in turn, calls the `update` function. The `update` function has a condition to only perform an update if `_balance > balance`. However, for the first depositor, `_balance` would be zero, as there are no rewards in the contract yet. Similarly, `balance` would also be zero, as it's defaulted to zero in the contract. So, no update would be made to the contract's index or balance in this case.

When execution returns to the `updateFor` function, it also has a condition to update only `if (_supplied > 0)`. But for the first depositor, `_supplied` would also be zero because the balance is updated in the `deposit` function only after calling `updateFor`. As a result, the execution wouldn't reach the line where the user's shares are added, causing the first depositor to not receive any rewards.

Here is the related code snippet:

```solidity
function deposit(uint _amount) external {
    TKN.transferFrom(msg.sender, address(this), _amount);
    updateFor(msg.sender); // called before updating the user's balance
    balances[msg.sender] += _amount; // @audituser's balance updated after `updateFor`
}
```

### Impact

The first depositor possibly missing out on rewards can have a significant impact on the fairness and appeal of the staking system. It also creates a potentially unintended imbalance in reward distribution.

### Recommended Mitigation

Pretty interesting case, but the recommended solution imo is to move the `updateFor(msg.sender)` function call to after the user's balance is updated in the `deposit` function. This ensures that the user's balance and the state of the contract are updated correctly before attempting to distribute rewards.

The corrected `deposit` function could look like this:

```solidity
function deposit(uint _amount) external {
    TKN.transferFrom(msg.sender, address(this), _amount);
    balances[msg.sender] += _amount; // move this before the `updateFor`
    updateFor(msg.sender); // move this after updating the user's balance
}
```

With this change, I think the first depositor should now be correctly accounted for.
|

## [M-02] Centralisation risks: The owner is a single point of failure

### Summary

The Beedle protocol, in its current state, manifests a single point of failure due to the overcentralization of authority. The contract owner can manipulate critical functions, which can be potentially harmful as it can lead to a system disruption if the owner is compromised or malicious.

### Vulnerability Details

Take the Beedle.sol contract for example, the `mint` function is subject to this vulnerability. This function is tagged with the `onlyOwner` modifier which means that only the contract owner has the capability to execute this function.
Take a look at [Beedle.sol#L36-L38](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Beedle.sol#L36-L38)

```solidity
function mint(address to, uint256 amount) external onlyOwner {
    _mint(to, amount);
}
```

The issue here is that if the owner's keys are compromised, an attacker can exploit these critical functions. The attacker can mint an arbitrary amount of tokens, which would lead to significant disruption of the system, token devaluation, and subsequent financial losses for the token holders.
Additionally if the owner is set to an EOA and not a multisig, the issue could even be worse

### Impact

Unexpected system disruption and potential financial losses for users.

### Recommended Mitigation

Consider introducing a multi-signature requirement for calling the `mint` function. This distributes control over critical functions across multiple trusted entities, reducing the single point of failure risk.

#### Additional Note

The centralization concerns submitted have been flagged as a medium-severity risk in previous contest's [discussions](https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837), emphasizing the need for awareness amongst users, also the admin priveleges issues attached in the discussion link should be checked and relayed to users in the case one affects the Beedle.fi protocol, so users are aware

## [M-03] Fees.sol: `sqrtPriceLimitX96` being hardcoded to 0 is not ideal

### Summary

The Fees contract sets the `sqrtPriceLimitX96` parameter to 0 in the Uniswap V3 swap operation, which makes the contract potentially susceptible to sandwich attacks.

### Vulnerability Details

Uniswap V3 introduced a concentrated liquidity model which provides the ability to limit slippage by setting the `sqrtPriceLimitX96` parameter. This parameter defines a price limit for a swap and prevents the trade from pushing the pool's price beyond this limit, for additional information you can check this [source](Source: https://ethereum.stackexchange.com/questions/121178/how-to-trade-directly-with-uniswap-v3-pool-instead-of-through-router
) However, in the `sellProfits()` function of the Fees contract, `sqrtPriceLimitX96` is hardcoded to 0 at [L39](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Fees.sol#L39) of Fees.sol:

Exarcebating this issue is the fact that the `amountOutMinimum` has also being hardcoded, however though being similar this report is not aimed at being a duplicate of the other report submitted by me on the slippage issue of having no specified `amountOutMinimum`. Additionally, from all this it'd be key to note that hardcoding `sqrtPriceLimitX96` to 0 in a swap transaction essentially removes any limit on price slippage. This can lead to different attacks one in which an attacker could even try to backrun a transaction, and cause a user financial loss

### Impact

Already explained in _Vulnerability Details_, but to reiterate my point this easily causes financial losses for users

### Recommended Mitigation

Allow users to set the `sqrtPriceLimitX96` parameter to mitigate this.

## [M-04] Fee on transfer tokens will not behave as expected

### Summary

Integrating fee-on-transfer tokens would completely break beedle.fi's internal accounting

### Vulnerability Details

The current implementation of the beedle.fi protocol doesn't work with fee-on-transfer underlying tokens, _emphasis on both the staking.sol and leder.sol contracts_
When a fee is charged on a transfer of tokens in Solidity, it is important to check the balance of the sender's account before and after the transfer to ensure that the fee has been correctly deducted from the sender's balance.

Take a look at [Staking.sol#L36-L42](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Staking.sol#L36-L42)

```solidity
    /// @notice deposit tokens to stake
    /// @param _amount the amount to deposit
    function deposit(uint _amount) external {
        TKN.transferFrom(msg.sender, address(this), _amount);
        updateFor(msg.sender);
        balances[msg.sender] += _amount;
    }
```

As seen no check is performed to really check if the amount of tokens received are what was sent note that since this check is not performed, it can result in an accounting error where the sender's balance is overstated, leading to potential issues with record keeping, reporting, and reconciliation.

For example the [withdraw()](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Staking.sol#L44-L46)) functon:

```solidity
    /// @notice withdraw tokens from stake
    /// @param _amount the amount to withdraw
    function withdraw(uint _amount) external {
        updateFor(msg.sender);
        balances[msg.sender] -= _amount;
        TKN.transfer(msg.sender, _amount);
    }
```

When this is called with the "amount" that was deposited, the contract would try to transfer the "amount" and as such would lead to taking part of other users deposit.

### Impact

Break of internal accounting, in the case of the `deposit()` function of `Staking.sol`, the contract directly registers the _amount_ provided and not the one received.

### Recommendation

Check the balances before and after any transfer with fees to ensure accurate accounting.

#### Additional Note

Would be key to note that this issue has multiple instances within the protocol (_over 20 instances of transfer/trasferFrom in Lender.sol_), but for brevity reasons this report only focused on the instance in `Beedle.sol` contract, advisably fixes should be made into all instances, lastly do note that there are some tokens who do not collect fees with transfers now, but this could change later on and this should be taken into account while this is being mitigated.
Sure, here's the reformatted audit report for the contract `Staking.sol`

## [L-01] Pragma isn't specified correctly which can lead to nonfunction/damaged contract when deployed on Arbitrum

### Summary

It's been relayed in that Beedle is going to be uploaded on optimism and in the future would probably ne uploaded elsewhere, this means that it easily would get deployed on arbitrum, being that both arbitrum and optimism use optimistic roll ups

### Vulnerability Detail

Floating pragma is used, allowing the contracts to be compiled with any 0.8.x compiler higher than the specified version. The problem with this is that Arbitrum is [NOT compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer due to the introduction of a new opcode `PUSH0`. Contracts compiled with those versions will result in a nonfunctional or potentially damaged version that won't behave as expected. The default behavior of compiler would be to use the newest version which would mean by default it will be compiled with the 0.8.20 version which will produce broken code.

### Impact

Damaged or nonfunctional contracts when deployed on Arbitrum.

### Recommendation

Constrain pragma could be something as follows:

    pragma solidity 0.8.19

## [L-02] Lack of two-step role transfer in Ownable.sol

### Summary

` Ownable.sol`does not utilize a two-step role transfer process for changing ownership. The `transferOwnership` function, which can only be invoked by the contract owner, is currently implemented as a single-step function.

### Vulnerability Details

Take a look at [Ownable.sol#L19-L22](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/utils/Ownable.sol#L19-L22)

```solidity
    function transferOwnership(address _owner) public virtual onlyOwner {
        owner = _owner;
        emit OwnershipTransferred(msg.sender, _owner);
    }
```

As seen the function merely requires the current owner to specify the new owner's address, which is then immediately set as the new owner. This implementation does not thoroughly validate the specified address. For instance, it does not account for the possibility that the address receiving the ownership role could be inaccessible. The current implementation is also prone to typographical errors, which may unintentionally lead to the transfer of ownership to an incorrect address.

### Tool Used

- Manual Audit
- Spearbit's [Clober Report](https://github.com/spearbit/portfolio/blob/master/pdfs/Clober-Spearbit-Security-Review.pdf)

### Recommended Mitigation

Implement a two-step ownership transfer process. This process involves the current owner proposing a new owner first. This proposed change doesn't take effect immediately. Instead, the address that has been proposed as the new owner has to accept the role to finalize the transfer.

This approach adds an extra layer of validation and decreases the likelihood of erroneous transfers since the new owner must actively accept the ownership role. It also allows for the correction of a mistake in the case the current owner sets an incorrect address as the proposed new owner.

**Modified Code**:

```solidity
address public proposedOwner;

function proposeNewOwner(address _proposedOwner) public virtual onlyOwner {
    proposedOwner = _proposedOwner;
}

function acceptOwnership() public {
    require(msg.sender == proposedOwner, "UNAUTHORIZED");
    emit OwnershipTransferred(owner, proposedOwner);
    owner = proposedOwner;
    proposedOwner = address(0);
}
```

## [L-03] Array length mismatch in the `giveLoan` Function

### Summary

The `giveLoan` function in the `Lender.sol` contract accepts two arrays, `loanIds` and `poolIds`, but it fails to validate that the lengths of these two input arrays match. In case of a mismatch, this can lead to unexpected behaviors and potential disruptions in the functioning of the contract.

### Vulneerability Details

The code instance of potential issue is in the [giveLoan()](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L351-L432) function in `Lender.sol` contract:

```solidity
    /// @notice give your loans to another pool
    /// can only be called by the lender
    /// @param loanIds the ids of the loans to give
    /// @param poolIds the id of the pools to give to
    function giveLoan(
        uint256[] calldata loanIds,
        bytes32[] calldata poolIds
    ) external {
        for (uint256 i = 0; i < loanIds.length; i++) {
            uint256 loanId = loanIds[i];
            bytes32 poolId = poolIds[i];
            // get the loan info
            Loan memory loan = loans[loanId];
            // validate the loan
            if (msg.sender != loan.lender) revert Unauthorized();
            // get the pool info
            Pool memory pool = pools[poolId];
            // validate the new loan
            if (pool.loanToken != loan.loanToken) revert TokenMismatch();
            if (pool.collateralToken != loan.collateralToken)
                revert TokenMismatch();
            // new interest rate cannot be higher than old interest rate
            if (pool.interestRate > loan.interestRate) revert RateTooHigh();
            // auction length cannot be shorter than old auction length
            if (pool.auctionLength < loan.auctionLength) revert AuctionTooShort();
            // calculate the interest
            (
                uint256 lenderInterest,
                uint256 protocolInterest
            ) = _calculateInterest(loan);
            uint256 totalDebt = loan.debt + lenderInterest + protocolInterest;
            if (pool.poolBalance < totalDebt) revert PoolTooSmall();
            if (totalDebt < pool.minLoanSize) revert LoanTooSmall();
            uint256 loanRatio = (totalDebt * 10 ** 18) / loan.collateral;
            if (loanRatio > pool.maxLoanRatio) revert RatioTooHigh();
            // update the pool balance of the new lender
            _updatePoolBalance(poolId, pool.poolBalance - totalDebt);
            pools[poolId].outstandingLoans += totalDebt;


            // update the pool balance of the old lender
            bytes32 oldPoolId = getPoolId(
                loan.lender,
                loan.loanToken,
                loan.collateralToken
            );
            _updatePoolBalance(
                oldPoolId,
                pools[oldPoolId].poolBalance + loan.debt + lenderInterest
            );
            pools[oldPoolId].outstandingLoans -= loan.debt;


            // transfer the protocol fee to the governance
            IERC20(loan.loanToken).transfer(feeReceiver, protocolInterest);


            emit Repaid(
                loan.borrower,
                loan.lender,
                loanId,
                loan.debt + lenderInterest + protocolInterest,
                loan.collateral,
                loan.interestRate,
                loan.startTimestamp
            );


            // update the loan with the new info
            loans[loanId].lender = pool.lender;
            loans[loanId].interestRate = pool.interestRate;
            loans[loanId].startTimestamp = block.timestamp;
            loans[loanId].auctionStartTimestamp = type(uint256).max;
            loans[loanId].debt = totalDebt;


            emit Borrowed(
                loan.borrower,
                pool.lender,
                loanId,
                loans[loanId].debt,
                loans[loanId].collateral,
                pool.interestRate,
                block.timestamp
            );
        }
    }
```

As seen this function attempts to match loan IDs to pool IDs based on their respective array indices. If the lengths of the `loanIds` and `poolIds` arrays do not match, this could lead to accessing an out-of-bounds index, which in turn might lead to exceptions or undefined behavior.

### Impact

If the lengths of the input arrays `loanIds` and `poolIds` do not match, an exception or undefined behavior could occur, disrupting the expected operation of the `giveLoan` function.

### Recommended Mitigation

Add an input validation step in the `giveLoan` function to ensure the lengths of the input arrays `loanIds` and `poolIds` match before proceeding with the operations. If the lengths do not match, the function should revert with a meaningful error message.

This check could look like:

```solidity
require(loanIds.length == poolIds.length, "Input array lengths do not match");
```

## [L-04] ERC20 return values not checked

### Summary

There are over _20 instances_ of` transfer/transferFrom()` in `Lender.sol` and none of their return values are checked

### Vulnerability Detail

The ERC20.transfer() and ERC20.transferFrom() functions return a boolean value indicating success. This parameter needs to be checked for success. Some tokens (like USDT) don't correctly implement the EIP20 standard, some tokens do not revert if the transfer failed but return false instead.

Since the beedle.fi protocol would allow multiple ERC20 tokens as the `loanTokens`. This should be taken into account and correctly mitigated against.

### Impact

Tokens that don't actually perform the transfer and return false are still counted as a correct transfer.

### Tool used

Manual Audit

### Recommendation

We recommend checking the success boolean of all .transfer calls for the unknown token contract or better use OpenZeppelin’s SafeERC20 versions with the safeTransfer functions that handle the return value check as well as non-standard-compliant tokens. Also, it is recommended to check the ERC20.approve return value as well.

#### Additional Note

For further information check out these links:

- [Link 1](https://github.com/d-xo/weird-erc20#missing-return-values)
- [Link 2](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca)

## [L-05] Lack of zero-address validation in various functions

### Summary

Lack of zero-address checks might lead to inoperable contract if parameter setings are not handled carefully.

### Vulnerability Details

A number of constructors/functions in the codebase do not revert if the zero address is passed in for a parameter that should not be set to zero.
An example can be seen in the [setFeeReceiver()](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L100) function of `Lender.sol`

```
    /// @notice set the fee receiver
    /// can only be called by the owner
    /// @param _feeReceiver the new fee receiver
    function setFeeReceiver(address _feeReceiver) external onlyOwner {
        feeReceiver = _feeReceiver;
    }
```

### Impact

If any of the parameters are accidentally set to address zero address, this wouldeasily cause huge impact to the protocol.

### Tool Used

Manual Audit

### Recommend Mitigation

Add zero-address validation for the parameters listed above. Review input validation across components.

#### Additional Note

For brevity reasons only one instance has been indicated in report but issue occurs multiple times in codebase and should correctly mitigated against in all instances.

## [NC-01] Lender.sol: The error being `PoolConfig` in most cases is completely a downside of protocol as users can't know the reasons to why their transaction failed

### Summary

There are 7 instances of `PoolConfig` being used as the revert message in `Lender.sol` this seems like an unnecessay complexity that actually leads to users not understanding why exactly their transaction is not going through

### Vulnerability Details

See _Summary_. additionally take these instances as an example:
[Lender.sol#L146](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L146)

```solidity
        // you can't change the outstanding loans
        if (p.outstandingLoans != pools[poolId].outstandingLoans)
            revert PoolConfig();

```

[Lender.sol#L198-L201](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L198-L201)

```solidity
    function removeFromPool(bytes32 poolId, uint256 amount) external {
        if (pools[poolId].lender != msg.sender) revert Unauthorized();
        if (amount == 0) revert PoolConfig();

```

As seen the first revert happens in a case where the provided outstanding loans is not same as whst's been stored in the contract, whereas the second instance is a revert happening if the attempted removal is of `amount = 0`, but in both instances the reverted error message is poolConfig which just seems too vague and could easily lead to confusion.

### Impact

Users hardship while interacting with protocol as they would have a hard time trying to find out what's wrong with their attempted function executions

### Recommend Mitigation

Error messsages should clearly indicate why a transcation has reverted, I recommend this mechanism to be implemented in the protocol

#### Additional Note

There is an incomplete code explanation comment at [L442 of Lender.sol](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L442), it states that `validate the loan` whereas the validation being made in the next line is for the loan's lender, so `lender` should be added to the comment.

## [NC-02] Unnecessary `_diff` check in the `update()` function

### Summary

In the Staking contract's `update` function, there's an unnecessary condition check where the contract verifies if `_diff` is greater than 0. This is redundant since `_diff` is calculated after validating that `balance` is less than `_balance`, meaning `_diff` will always be greater than zero.

### Vulnerability Details

In the following code snippet of the [update()](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Staking.sol#L60-L76) function:

```solidity
    /// @notice update the global index of earned rewards
    function update() public {
        uint256 totalSupply = TKN.balanceOf(address(this));
        if (totalSupply > 0) {
            uint256 _balance = WETH.balanceOf(address(this));
            if (_balance > balance) {
                uint256 _diff = _balance - balance;
                //@audit unnecessary check
                if (_diff > 0) {
                    uint256 _ratio = _diff * 1e18 / totalSupply;
                    if (_ratio > 0) {
                      index = index + _ratio;
                      balance = _balance;
                    }
                }
            }
        }
    }
```

`_diff` is calculated as the difference between `_balance` and `balance`, after checking that `balance` is less than `_balance`. This means that `_diff` will always be greater than 0, and checking this condition is unnecessary and adds to the gas costs of the operation.

### Impact

The impact of this issue is mostly related to efficiency rather than security. The unnecessary condition increases the gas cost of the `update` function, causing users to spend more than necessary on transaction fees. While this might be small for a single transaction, it could add up significantly over time and with many users.

### Tools Used

This issue was identified through manual code review, using Solidity, the language in which the smart contract is written.

### Recommend Mitigation

The recommended mitigation for this issue is to remove the condition `_diff > 0`. Given that `_diff` is always greater than zero due to the previous checks, this condition is redundant and removing it will make the function more gas-efficient.

```solidity
uint256 _ratio = _diff * 1e18 / totalSupply;
if (_ratio > 0) {
  index = index + _ratio;
  balance = _balance;
}
```

By removing the unnecessary condition, gas cost can be reduced without affecting the functionality of the `update` function.

## [NC-03] Setter functions should have equality checks

### Summary

Multiple setter functions in the protocol do not include an equality check, this includes the function `updateMaxLoanRatio` in the Lender.sol contract that's used for updating the maximum loan ratio for a pool. However implementing an equality check first would curb unnecessary storage operations when the new value _maximum loan ratio in this case_ is the same as the current one.

### Vulnerability Details

This report only focuses on the instance within the `updateMaxLoanRatio()` function , which takes a `poolId` and a `maxLoanRatio` as arguments.
Now if the caller of the function is not the lender of the pool or if the `maxLoanRatio` is 0, the function reverts. Otherwise, it updates the `maxLoanRatio` of the pool and emits an event.

[Lender.sol#L206-L215](https://github.com/Cyfrin/2023-07-beedle/blob/658e046bda8b010a5b82d2d85e824f3823602d27/src/Lender.sol#L206-L215):

```solidity
    /// @notice update the max loan ratio for a pool
    /// can only be called by the pool lender
    /// @param poolId the id of the pool to update
    /// @param maxLoanRatio the new max loan ratio
    function updateMaxLoanRatio(bytes32 poolId, uint256 maxLoanRatio) external {
        if (pools[poolId].lender != msg.sender) revert Unauthorized();
        if (maxLoanRatio == 0) revert PoolConfig();
        pools[poolId].maxLoanRatio = maxLoanRatio;
        emit PoolMaxLoanRatioUpdated(poolId, maxLoanRatio);
    }
```

### Impact

Unnecessary gas costs and data storage operations when the `updateMaxLoanRatio` function is called with a `maxLoanRatio` that is the same as the current one. Or whatever setter function is called with the same value. This impact may be somewhat minor but it's still an inefficiency that can be easily avoided.

### Recommended Mitigation

To avoid unnecessary operations, a check could be added to the function to verify whether the new value is the same as the current one. If it is, the function could revert with an error message indicating that an attemot was made to set the value to it's present value.
