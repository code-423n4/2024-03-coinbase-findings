

# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Internal functions only called once can be inlined to save gas|4|
|[G‑02]| When possible, use assembly instead of unchecked{++i}|1|
|[G‑03]|Unlimited gas consumption risk due to external call recipients|2|
|[G‑04]|Use inline assembly to access the chain ID|3|
|[G‑05]|Shorten the array rather than copying to a new one|1|
|[G‑06]|Avoid unnecessary public variables|1|
|[G‑07]|Make 3 event parameters indexed when possible|3|
|[G‑08]|`internal` functions not called by the contract should be removed to save deployment gas|2|
|[G‑09]|Use assembly to calculate hashes to save gas|3|
|[G‑10]|`keccak256()` should only need to be called on a specific string literal once|1|
|[G‑11]|Don't compare boolean expressions to boolean literals|1|
|[G‑12]| Optimize External Calls with Assembly for Memory Efficiency|4|
|[G‑13]|abi.encode() is less efficient than abi.encodePacked()|3|




## [G-01] Internal functions only called once can be inlined to save gas

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




## [G-02] When possible, use assembly instead of unchecked{++i}

You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.





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



## [G‑03] Unlimited gas consumption risk due to external call recipients

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

## [G‑04] Use inline assembly to access the chain ID

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


## [G-05] Shorten the array rather than copying to a new one

ASSEMBLY can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array



```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
104   bytes[] memory owners = new bytes[](1);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104


## [G-06] Avoid unnecessary public variables

Public state variables in Solidity automatically generate getter functions, increasing contract size and potentially leading to higher deployment and interaction costs. To optimize gas usage and contract efficiency, minimize the use of public variables unless external access is necessary.

note:missed from bots.

- the `REPLAYABLE_NONCE_KEY` public variable only use in CoinbaseSmartWallet contract change the visibility to save gas
```solidity
File:  src/SmartWallet/CoinbaseSmartWallet.sol
43  uint256 public constant REPLAYABLE_NONCE_KEY = 8453;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L43


## [G-07] Make 3 event parameters indexed when possible
It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.



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



## [G-08] `internal` functions not called by the contract should be removed to save deployment gas


```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
    function _domainNameAndVersion() internal pure override(ERC1271) returns (string memory, string memory) {
        return ("Coinbase Smart Wallet", "1");
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L333:335




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




## [G-11] Don't compare boolean expressions to boolean literals
For cases of: `if (<x> == true)`, use `if (<x>)` instead. For cases of: `if (<x> == false)`, use `if (!<x>)` instead.

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
53              if (alreadyDeployed == false) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L53



## [G-12] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets.

```solidity
File:  src/MagicSpend/MagicSpend.sol
223  IEntryPoint(entryPoint()).withdrawTo(to, amount);

233  IEntryPoint(entryPoint()).addStake{value: amount}(unstakeDelaySeconds);

244  IEntryPoint(entryPoint()).unlockStake();

249  IEntryPoint(entryPoint()).withdrawStake(to);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L223


## [G-04] abi.encode() is less efficient than abi.encodePacked()
See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison


- Bot find incorrectly this finding [[G-04]](https://github.com/code-423n4/2024-03-coinbase/blob/main/bot-report.md#g-04-abiencode-is-less-efficient-than-abiencodepacked)

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


