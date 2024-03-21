# QA Report: Smart Wallet

## L-01 Employ ERC-1167 Clones for smart wallets

Using ERC-1167 clones allows each deployed smart wallet to operate independently (having its own state) while still using the same underlying logic. This is ideal for user-specific wallet contracts where each user gets a separate wallet but all wallets share the same functionality.

The primary feature of ERC-1167, creating lightweight proxy contracts, aligns with the need to deploy multiple smart wallet instances without the high cost associated with deploying full contract code for each instance. This is particularly beneficial for a factory pattern that aims to produce many instances of a particular contract type.

## L-02 Integration Constraints Imposed by isContract Checks

Many DeFi protocols and other smart contract systems use isContract checks to determine if a caller is an EOA or another smart contract. This is often used for security reasons or to enforce certain behaviors. For example, a contract might restrict certain functions to only be callable by EOAs to prevent reentrancy attacks or other contract-based exploits.

Interoperability Issues: If a project uses isContract checks to restrict interactions to EOAs, then smart contract wallets deployed as minimal proxies (or any smart contract, for that matter) would be unable to interact with these functions directly. This could limit the usability of smart contract wallets with such protocols, requiring additional steps or alternative approaches for users of smart contract wallets to engage with these systems.

## L-03 Denial of Service Due to Payload Size

Large initCode, callData, and paymasterAndData payloads can increase transaction costs and potentially exceed block gas limits, affecting the network and contract's performance.

#### [Loc:](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L109) 

Impose size limits on initCode, callData, and paymasterAndData to prevent excessive transaction sizes.

```solidity
require(userOp.initCode.length <= MAX_INIT_CODE_SIZE, "initCode too large");
require(userOp.callData.length <= MAX_CALL_DATA_SIZE, "callData too large");
```

## L-04 Denial of Service Due to High Gas Limits

callGasLimit, verificationGasLimit, preVerificationGas, maxFeePerGas, and maxPriorityFeePerGas could be manipulated to attempt DoS attacks by specifying gas limits that are too high or too low, potentially leading to transaction failure or exploiting the paymaster's funds.

#### [Loc:](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L109) 

Set maximum limits for callGasLimit and verificationGasLimit based on typical operation costs.

```solidity
require(userOp.callGasLimit <= MAX_CALL_GAS_LIMIT, "Excessive call gas limit");
require(userOp.verificationGasLimit <= MAX_VERIFICATION_GAS_LIMIT, "Excessive verification gas limit");
```


## L-05 Enforce Sequential Nonces

Ensure nonces are used sequentially without gaps, effectively mitigating replay attacks. This would involve tracking the last used nonce for each user and ensuring each new operation increments this nonce.

```solidity
uint256 expectedNonce = lastNonce[userOp.sender] + 1;
require(userOp.nonce == expectedNonce, "Invalid nonce");
lastNonce[userOp.sender] = expectedNonce;
```

## L-06 Limit Operation Frequency

For functions that involve stake management like entryPointAddStake, entryPointUnlockStake, and entryPointWithdrawStake impose a rate limit.

This reduces the risk of destabilizing the contract's operations through rapid, successive changes, allowing for more stable management of resources.

## L-07 No Setters for Configurable Parameters

Critical parameters like the EntryPoint address are hard-coded or lack the flexibility for updates. In a dynamic blockchain environment, the ability to update certain parameters without redeploying the contract could be crucial.

Add this:

Function: setEntryPointAddress(address newEntryPoint)
Purpose: Allows the admin to update the EntryPoint address.
Impact: Enhances flexibility and adaptability to changes in the underlying account abstraction infrastructure.

## L-08 Lack of Emergency Stop Mechanism

In case of detected vulnerabilities or attacks, an emergency stop mechanism (often implemented as a "circuit breaker") can temporarily halt critical contract operations.

Add pause and unpause functions. This will increases security by allowing immediate response to emergencies, protecting funds and preventing potential misuse.

## L-09 Lack of Withdrawal Limits

To further safeguard against unauthorized or accidental large withdrawals, setting daily or transactional limits could be beneficial.

Cansider adding this function, setWithdrawalLimit(uint256 newLimit)

It sets the maximum amount that can be withdrawn within a certain timeframe.

This prevents excessive withdrawals, adding an additional layer of financial security.
    

## L-10 No Role-Based Access Control

Allows for differentiated access levels, enabling more secure and flexible administration of the contract.

Improves security by limiting critical operations to authorized roles and facilitates delegation of specific responsibilities.

## L-11 Lack of Recovery Options for Lost Access
In case the owner's private key is lost or compromised, having a recovery mechanism can be crucial to regain control over the contract.

Add these functions: 
-initiateRecovery(address newOwner, uint256 recoveryDelay)
-finalizeRecovery()

This starts and completes the process of transferring ownership after a security delay.

It adds resilience to the contract by providing a path to recover from lost access or compromised keys, without compromising immediate security.

## L-12 No whitelists of supported tokens

Although the current implementation only supports withdrawing ETH (identified by the zero address), since you are planning to support other assets in the future. Implementing a check against a list of supported assets would make it easier to extend

```solidity
mapping(address => bool) public supportedAssets;

// In the constructor or a separate function
supportedAssets[address(0)] = true; // ETH

function _validateAsset(address asset) internal view {
    if (!supportedAssets[asset]) revert UnsupportedPaymasterAsset(asset);
}

```
## L-13 Emitting Misleading Events via OwnerWithdraw

The `ownerWithdraw` function allows the contract owner to withdraw assets to any address. If the contract owner acts maliciously or the owner's account is compromised, this function could be used to emit `MagicSpendWithdrawal` events that do not represent genuine user withdrawals.

Misuse of this functionality could flood the event log with false withdrawals, complicating the tracking of legitimate user activity and potentially misleading off-chain analytics or auditors.- 

Implementing a multi-signature requirement for sensitive owner operations can mitigate this risk.


## L-14 Inflated Withdrawal Events

A compromised admin account could repeatedly invoke functions that emit events, such as making numerous small withdrawals if the contract logic permits. This could lead to an inflated number of events, overwhelming off-chain analytics tools, and making it difficult to discern legitimate activities.

Introducing rate limits or thresholds for operations that trigger events can prevent abuse. For instance, setting a minimum withdrawal amount could deter the generation of numerous minimal withdrawal events.

## NC-01 Use of Magic Numbers

To improve clarity the number `160` could be defined as a named constant that explains its purpose. 

```solidity
// Represents the bit length of an Ethereum address, used for encoding purposes.
uint256 public constant ADDRESS_BIT_LENGTH = 160;
```

Then, you could use `ADDRESS_BIT_LENGTH` instead of the hard-coded `160` in your bit manipulation, making the intention behind this operation clearer to readers of the code:

```solidity
validationData = (sigFailed ? 1 : 0) | (uint256(withdrawRequest.expiry) << ADDRESS_BIT_LENGTH);
```

## NC-02 Hardcoded Addresses

Replace hardcoded addresses with named constants for clarity and potentially reducing gas costs if these constants are used multiple times.

```solidity
address public constant ENTRY_POINT_ADDRESS = 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
address public constant ETH_ADDRESS = address(0);

```