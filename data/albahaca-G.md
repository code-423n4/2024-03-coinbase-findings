### 1. [Optimization of `assert` Statement in `postOp` Function](#g-01-optimization-of-assert-statement-in-postop-function)
### 2. [Optimization Proposal for Withdraw Function in MagicSpend Contract](#g-02-optimization-proposal-for-withdraw-function-in-magicspend-contract)
### 3. [Gas Optimization Report: Refactoring `canSkipChainIdValidation` Function](#g-03-gas-optimization-report-refactoring-canskipchainidvalidation-function)
### 4. [abi.encode() is less efficient than abi.encodePacked()](#g-04-abiencode-is-less-efficient-than-abiencodepacked)
### 5. [Avoid unnecessary public variables](#g-05-avoid-unnecessary-public-variables)
### 6. [Make 3 event parameters indexed when possible](#g-06-make-3-event-parameters-indexed-when-possible)
### 7. [`internal` functions not called by the contract should be removed to save deployment gas](#g-07-internal-functions-not-called-by-the-contract-should-be-removed-to-save-deployment-gas)
### 8. [Internal functions only called once can be inlined to save gas](#g-08-internal-functions-only-called-once-can-be-inlined-to-save-gas)
### 9. [Use assembly to calculate hashes to save gas](#g-09-use-assembly-to-calculate-hashes-to-save-gas)
### 10. [`keccak256()` should only need to be called on a specific string literal once](#g-10-keccak256-should-only-need-to-be-called-on-a-specific-string-literal-once)
### 11. [When possible, use assembly instead of unchecked{++i}](#g-11-when-possible-use-assembly-instead-of-uncheckedi)
### 12. [Don't compare boolean expressions to boolean literals](#g-12-dont-compare-boolean-expressions-to-boolean-literals)
### 13. [Optimize External Calls with Assembly for Memory Efficiency](#g-13-optimize-external-calls-with-assembly-for-memory-efficiency)
### 14. [Unlimited gas consumption risk due to external call recipients](#g‑14-unlimited-gas-consumption-risk-due-to-external-call-recipients)
### 15. [Use inline assembly to access the chain ID](#g‑15-use-inline-assembly-to-access-the-chain-id)
### 16. [Shorten the array rather than copying to a new one](#g-16-shorten-the-array-rather-than-copying-to-a-new-one)



## [G-01] Optimization of `assert` Statement in `postOp` Function

**Reasoning for Change:**
While `assert` statements are useful for detecting unexpected conditions during development and testing, they are typically used to validate internal contract state rather than input parameters. In this context, using `assert` to validate an input parameter (`mode`) may not be the most appropriate approach, as it could lead to unexpected contract termination in case of failed assertions.

**Change Made:**
To address this concern, the `assert` statement was replaced with a `require` statement. Unlike `assert`, `require` statements are used to validate external inputs and ensure that functions are called with valid parameters. By using `require`, the function will revert with a specific error message if the condition is not met, providing better transparency and user feedback.


```solidity
File:  src/MagicSpend/MagicSpend.sol
150   assert(mode != PostOpMode.postOpReverted);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L150


## [G-02] Optimization Proposal for Withdraw Function in MagicSpend Contract

The `withdraw` function in the MagicSpend contract is currently structured such that it first calls the `_validateRequest` function, which involves reading and writing storage variables, and then checks if the signature provided in the `withdrawRequest` is valid. However, this sequence could potentially lead to gas inefficiencies, as the expensive operation of validating the request occurs before checking if the signature is valid.

```solidity
188 function withdraw(WithdrawRequest memory withdrawRequest) external {
        _validateRequest(msg.sender, withdrawRequest);

        if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
            revert InvalidSignature();
        }

        if (block.timestamp > withdrawRequest.expiry) {
            revert Expired();
        }

        // reserve funds for gas, will credit user with difference in post op
        _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
    }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L188-L194

- To optimize gas usage and improve efficiency, it's proposed to reorder the logic in the `withdraw` function as follows:
```solidity
function withdraw(WithdrawRequest memory withdrawRequest) external {
    // First, check if the provided signature is invalid
    if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
        revert InvalidSignature();
    }

    // Next, validate the request
    _validateRequest(msg.sender, withdrawRequest);

    // Check if the request has expired
    if (block.timestamp > withdrawRequest.expiry) {
        revert Expired();
    }

    // Reserve funds for gas and credit the user with the difference in post-op
    _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
}
```
By reordering the logic in this way, the expensive operation of validating the request in `_validateRequest` is deferred until after it is determined that the signature is valid. This ensures that gas is not wasted on unnecessary storage reads and writes if the signature check fails.


## [G-03] Gas Optimization Report: Refactoring `canSkipChainIdValidation` Function

**Reasons for Optimization:**
The original implementation of `canSkipChainIdValidation` relies on multiple logical OR operations to compare the given function selector against a set of predefined function selectors. However, each logical OR operation incurs additional gas costs. By refactoring the logic using inline assembly, we can potentially reduce gas consumption by consolidating the comparisons into a single operation.

```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
    function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
        if (
            functionSelector == MultiOwnable.addOwnerPublicKey.selector
                || functionSelector == MultiOwnable.addOwnerAddress.selector
                || functionSelector == MultiOwnable.removeOwnerAtIndex.selector
                || functionSelector == UUPSUpgradeable.upgradeToAndCall.selector
        ) {
            return true;
        }
        return false;
    }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L252-L26

**Proposed Optimization:**
The proposed optimization involves using inline assembly to perform a bitwise OR operation on the function selector and compare it directly against a bitmask representing the combination of all targeted function selectors.

**Implementation:**
1. **Define Function Selectors:** First, we define the function selectors for the targeted functions (e.g., `addOwnerPublicKey`, `addOwnerAddress`, `removeOwnerAtIndex`, and `upgradeToAndCall`). These selectors are represented as hexadecimal values.

2. **Bitwise OR Operation:** Within the inline assembly block, we perform a bitwise OR operation (`or`) on the function selector with each of the predefined function selectors.

3. **Compare with Targeted Selectors:** The result of the bitwise OR operation is then compared (`eq`) against a bitmask representing the combination of all targeted function selectors.

4. **Return Result:** Finally, the result of the comparison is returned as a boolean value, indicating whether the given function selector matches any of the targeted function selectors.




## [G-04] abi.encode() is less efficient than abi.encodePacked()
See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

There are 2 instance(s) of this issue:

- Bot find incorrectly this finding [[G-04]](https://github.com/code-423n4/2024-03-coinbase/blob/main/bot-report.md#g-04-abiencode-is-less-efficient-than-abiencodepacked)

- note : there is no chance of collisions when you use abi.encodePacked in these instance
```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
235  return keccak256(abi.encode(UserOperationLib.hash(userOp), entryPoint()));

321  return WebAuthn.verify({challenge: abi.encode(message), requireUV: false, webAuthnAuth: auth, x: x, y: y});
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L235


```solidity
File:  src/SmartWallet/ERC1271.sol
134   return keccak256(abi.encode(_MESSAGE_TYPEHASH, hash));
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L134



## [G-05] Avoid unnecessary public variables
Public state variables in Solidity automatically generate getter functions, increasing contract size and potentially leading to higher deployment and interaction costs. To optimize gas usage and contract efficiency, minimize the use of public variables unless external access is necessary. Instead, use internal or private visibility combined with explicit getter functions when required. This practice not only reduces contract size but also provides better control over data access and manipulation, enhancing security and readability. Prioritize lean, efficient contracts to ensure cost-effectiveness and better performance on the blockchain.

note: this instance missed from bots.

There are 1 instance(s) of this issue:
- the `REPLAYABLE_NONCE_KEY` public variable only use in CoinbaseSmartWallet contract change the visibility to save gas
```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
43  uint256 public constant REPLAYABLE_NONCE_KEY = 8453;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L43


## [G-06] Make 3 event parameters indexed when possible
It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

There are 3 instances of this issue:


```solidity
File:  src/MagicSpend/MagicSpend.sol
45    event MagicSpendWithdrawal(address indexed account, address indexed asset, uint256 amount, uint256 nonce);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L45


```solidity
File:  src/SmartWallet/MultiOwnable.sol
68   event AddOwner(uint256 indexed index, bytes owner);

74   event RemoveOwner(uint256 indexed index, bytes owner);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L68


## [G-07] `internal` functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
    function _domainNameAndVersion() internal pure override(ERC1271) returns (string memory, string memory) {
        return ("Coinbase Smart Wallet", "1");
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L333:335



## [G-08] Internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
291         function _validateSignature(bytes32 message, bytes calldata signature)
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291

```solidity
File: src/SmartWallet/ERC1271.sol
121         function _eip712Hash(bytes32 hash) internal view virtual returns (bytes32) {

133         function _hashStruct(bytes32 hash) internal view virtual returns (bytes32) {
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L133:133

```solidity
File: src/SmartWallet/MultiOwnable.sol
201         function _checkOwner() internal view virtual {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L201


## [G-09] Use assembly to calculate hashes to save gas

Using assembly to calculate hashes can save 80 gas per instance

note: missed from bots
```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
82              salt = keccak256(abi.encode(owners, nonce));
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L82

```solidity
File: src/SmartWallet/ERC1271.sol
105                     keccak256(bytes(name)),

106                     keccak256(bytes(version)),
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L105




## [G-10] `keccak256()` should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once.


```solidity
File: src/SmartWallet/ERC1271.sol
104                     keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L104



## [G-11] When possible, use assembly instead of unchecked{++i}
You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```
Gas: 1329
```
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```
Gas: 709

```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
206     for (uint256 i; i < calls.length;) {
            _call(calls[i].target, calls[i].value, calls[i].data);
            unchecked {
                ++i;
            }
        }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L206-L211


## [G-12] Don't compare boolean expressions to boolean literals
For cases of: `if (<x> == true)`, use `if (<x>)` instead. For cases of: `if (<x> == false)`, use `if (!<x>)` instead.

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
53              if (alreadyDeployed == false) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L53



## [G-13] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

```solidity
File:  src/MagicSpend/MagicSpend.sol
223  IEntryPoint(entryPoint()).withdrawTo(to, amount);

233  IEntryPoint(entryPoint()).addStake{value: amount}(unstakeDelaySeconds);

244  IEntryPoint(entryPoint()).unlockStake();

249  IEntryPoint(entryPoint()).withdrawStake(to);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L223


## [G‑14] Unlimited gas consumption risk due to external call recipients
When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
273  (bool success, bytes memory result) = target.call{value: value}(data);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L273

```solidity
File:  src/MagicSpend/MagicSpend.sol
233   IEntryPoint(entryPoint()).addStake{value: amount}(unstakeDelaySeconds);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L233

## [G‑15] Use inline assembly to access the chain ID
The original implementation uses the block.chainid global variable, while the optimized version employs inline assembly to access the chain ID, potentially reducing gas consumption.

```solidity
File:  src/MagicSpend/MagicSpend.sol
284   block.chainid,
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L284

```solidity
File:  src/SmartWallet/ERC1271.sol
52  chainId = block.chainid;

107   block.chainid,
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L52


## [G-16] Shorten the array rather than copying to a new one
ASSEMBLY can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
104   bytes[] memory owners = new bytes[](1);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104

