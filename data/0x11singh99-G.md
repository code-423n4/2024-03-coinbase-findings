# Gas Optimizations

**Note : _G-08_, _G-11_ and _G-12_ contains only those instances which were missed by bot. Since they are major gas savings so I included those missed instances**

## Table of Contents

- [G-01] [Update `storage` variable once outside of the loop instead of updating it every time in loop it saves 1 SSTORE, 1 SLOAD per iteration(**Saves ~2200 Gas per iteration**)](#g-01-update-storage-variable-once-outside-of-the-loop-instead-of-updating-it-every-time-in-loop-it-saves-1-sstore-1sload-per-iterationsaves-2200-gas-per-iteration)

- [G-02] [Refactor `validatePaymasterUserOp` function to fail early and save 2 function calls (**Saves ~20K-21K Gas Half of the times**)](#g-02-refactor-validatepaymasteruserop-function-to-fail-early-and-save-2-function-calls-saves-20k-21k-gas-half-of-the-times)

- [G-03] [Call `_getMultiOwnableStorage()` one time to fetch storage pointer and avoid extra internal function call](#g-03-call-_getmultiownablestorage-one-time-to-fetch-storage-pointer-and-avoid-extra-internal-function-call)

- [G-04] [Check `withdrawAmount - maxCost` for greater than `0` if `0` it can save 1 `Gsreset` (~2900 Gas) and 1 `Gcoldsload` (**Saves ~5000 Gas**)](#g-04-check-withdrawamount---maxcost-for-greater-than-0-if-it-is-0-it-can-save-1-gsreset-2900-gas-and-1-gcoldsload-saves-5000-gas)

- [G-05] [Cache the resulted `keccak256 hash` of `constant strings` instead of re-calculating the hash on every function call](#g-05-cache-the-resulted-keccak256-hash-of-constant-string-instead-of-re-calculating-the-hash-on-every-function-call-saves-50-gas)

- [G-06] [`Switch` the order of `if` statement to fail early saves 1 function call half of the times where function have SLOAD (**Saves ~2100 Gas** Half of the times)](#g-06-switch-the-order-of-if-statement-to-fail-early-saves-1-function-call-half-of-the-times-where-function-have-sload-saves-2100-gas-half-of-the-times)

- [G-07] [Check `contract` balance before sending `ethers` to fail early](#g-07-check-contract-balance-before-sending-ethers-to-fail-early)

- [G-08] [Use unchecked where underflow not possible(**Gas Saved: ~150 Gas**)(_Missed by bot_)](#g-08-use-unchecked-where-underflow-not-possiblegas-saved-150-gasmissed-by-bot)
- [G-09] [Do not assign a variable with its default value](#g-09-do-not-assign-a-variable-with-its-default-value)
- [G-10] [Cache address(this).balance instead of recalculating it](#g-10-cache-addressthisbalance-instead-of-recalculating-it)
- [G-11] [Use nested if and separate if instead of && and || respectively(_Instances Missed by bot_)](#g-11-use-nested-if-and-separate-if-instead-of--and--respectivelyinstances-missed-by-bot)

- [G-12] [Use `abi.encodePacked` instead of `abi.encode` (_Instances Missed by bot_)](#g-12-use-abiencodepacked-instead-of-abiencode-instances-missed-by-bot)

## Auditor's Disclaimer :

All these findings are good findings and 100% safe to implement at no security/logic risk. They all are found by thorough manual review.

## [G-01] Update `storage` variable once outside of the loop instead of updating it every time in loop it saves 1 SSTORE, 1SLOAD per iteration(**Saves ~2200 Gas per iteration**)

Since `nextOwnerIndex` doesn't depend on loop iteration, it is simply being updated one by one. And `nextOwnerIndex` is not updated anywhere in the loop including inside `_addOwnerAtIndex` function except here in param nor it's updated value accessed. In `_addOwnerAtIndex` 2nd param updated `nextOwnerIndex` value passed from here. We can achieve the same result by caching `nextOwnerIndex` value. And update that cached stack value inside loop instead of this and passed also that cached updated value into `_addOwnerAtIndex` function. It will work as same since in place of updating storage `nextOwnerIndex` we are we are updating cached stack value. And after the end of loop outside the loop we will update the storage var. `_getMultiOwnableStorage().nextOwnerIndex` with the updated stack var. value. Which will result in same result ans saves a lot of gas.

### Saves 1 SSTORE, 1SLOAD and also 1 internal function call per iteration Saves ~2200 Gas per iteration

```solidity
File : SmartWallet/MultiOwnable.sol

163:  for (uint256 i; i < owners.length; i++) {
...
172:    _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
...
       }
```

[MultiOwnable.sol#L172](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L172)

**Recommended Mitigation Steps:**

```diff
File : SmartWallet/MultiOwnable.sol

+      uint256 cached_nextOwnerIndex = _getMultiOwnableStorage().nextOwnerIndex;//@audit cache initial nextOwnerIndex storage value.
163:  for (uint256 i; i < owners.length; i++) {
...
-172:    _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
+172:    _addOwnerAtIndex(owners[i], cached_nextOwnerIndex++);//@audit updating cached stack value instead of nextOwnerIndex storage value
...
       }
+    _getMultiOwnableStorage().nextOwnerIndex = cached_nextOwnerIndex; //@audit update storage var.  with final value
```

## [G-02] Refactor `validatePaymasterUserOp` function to fail early and save 2 function calls (**Saves ~20k-21k Gas Half of the times**)

Since `withdrawAmount` is not changed during the function calls. So we can check first `address(this).balance < withdrawAmount` to fail early by placing this if check before any call. Since this if check var. `withdrawAmount` not based on any function call nor updated by them so it will be same before or after those function call so place this if check before to fail early if this check is failing it can avoid up to 2 function calls. Where `_validateRequest` function doing 1 SSTORE and 1 SLOAD. Where SSTORE is used to change the value from false to true which takes ~20k Gas. So placing this if check before can save these operations when if check fails and can save up to ~20k gas.

### Can save 2 function calls , in `_validateRequest` function 1 SSTORE and 1 SLOAD, where `SSTORE` is used to change the value from `false` to `true` which take ~20k gas. Saves ~20-21k Gas Half of the times ie. when if check fails and reverts.

```solidity
File : MagicSpend/MagicSpend.sol

125:   _validateRequest(userOp.sender, withdrawRequest);
126:
127:    bool sigFailed = !isValidWithdrawSignature(userOp.sender, withdrawRequest);
128:    validationData = (sigFailed ? 1 : 0) | (uint256(withdrawRequest.expiry) << 160);
129:
130:    // Ensure at validation that the contract has enough balance to cover the requested funds.
131:    // NOTE: This check is necessary to enforce that the contract will be able to transfer the remaining funds
132:   //       when `postOp()` is called back after the `UserOperation` has been executed.
133:    if (address(this).balance < withdrawAmount) {
134:       revert InsufficientBalance(withdrawAmount, address(this).balance);
135:     }

```

[MagicSpend.sol#L125-L135](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L125-L135)

**Recommended Mitigation Steps:**

```diff
File : MagicSpend/MagicSpend.sol

+    if (address(this).balance < withdrawAmount) {
+       revert InsufficientBalance(withdrawAmount, address(this).balance);
+     }
125:   _validateRequest(userOp.sender, withdrawRequest);
126:
127:    bool sigFailed = !isValidWithdrawSignature(userOp.sender, withdrawRequest);
128:    validationData = (sigFailed ? 1 : 0) | (uint256(withdrawRequest.expiry) << 160);
129:
130:    // Ensure at validation that the contract has enough balance to cover the requested funds.
131:    // NOTE: This check is necessary to enforce that the contract will be able to transfer the remaining funds
132:   //       when `postOp()` is called back after the `UserOperation` has been executed.
-133:    if (address(this).balance < withdrawAmount) {
-134:       revert InsufficientBalance(withdrawAmount, address(this).balance);
-135:     }

```

## [G-03] Call `_getMultiOwnableStorage()` one time to fetch storage pointer and avoid extra internal function call

Since `_getMultiOwnableStorage()` returns same storage pointer of `MultiOwnableStorage` type so it is enough to call once and take that pointer in storage instead of calling `_getMultiOwnableStorage()` second time since it returns same storage pointer which is already returned and points to the same storage location. So weather it is returned through function or cache in storage pointer it will point to same storage location and update that same storage by that pointer. See Mitigation for clarity. So **Saves 1 internal function call**

This is below the function which is returning storage pointer of `MultiOwnableStorage` type

```solidity
File : SmartWallet/MultiOwnable.sol

212: function _getMultiOwnableStorage() internal pure returns (MultiOwnableStorage storage $) {
213:    assembly ("memory-safe") {
214:        $.slot := MUTLI_OWNABLE_STORAGE_LOCATION
215:    }
216:   }
```

[MultiOwnable.sol#L212-L216](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L212-L216)

---

### 2 Instances of this finding using same function twice in a function, So total 2 internal function calls saved

#### Instance 1

```solidity
File : SmartWallet/MultiOwnable.sol

102: function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {
103:    bytes memory owner = ownerAtIndex(index);
104:    if (owner.length == 0) revert NoOwnerAtIndex(index);
105:
106:    delete _getMultiOwnableStorage().isOwner[owner];
107:    delete _getMultiOwnableStorage().ownerAtIndex[index];

```

[MultiOwnable.sol#L102-L107](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L102-L107)

**Recommended Mitigation Steps:**

```diff
File : SmartWallet/MultiOwnable.sol

102: function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {
103:    bytes memory owner = ownerAtIndex(index);
104:    if (owner.length == 0) revert NoOwnerAtIndex(index);
+       MultiOwnableStorage storage $ = _getMultiOwnableStorage();
105:
-106:    delete _getMultiOwnableStorage().isOwner[owner];
+106:    delete $.isOwner[owner];
-107:    delete _getMultiOwnableStorage().ownerAtIndex[index];
+107:    delete $.ownerAtIndex[index];

```

#### Instance 2

```solidity
File : SmartWallet/MultiOwnable.sol

189: function _addOwnerAtIndex(bytes memory owner, uint256 index) internal virtual {
190:     if (isOwnerBytes(owner)) revert AlreadyOwner(owner);
191:
192:     _getMultiOwnableStorage().isOwner[owner] = true;
193:     _getMultiOwnableStorage().ownerAtIndex[index] = owner;

```

[MultiOwnable.sol#L189-L193](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L189-L193)

**Recommended Mitigation Steps:**

```diff
File : SmartWallet/MultiOwnable.sol

189: function _addOwnerAtIndex(bytes memory owner, uint256 index) internal virtual {
190:     if (isOwnerBytes(owner)) revert AlreadyOwner(owner);
+       MultiOwnableStorage storage $ = _getMultiOwnableStorage();
191:
-192:     _getMultiOwnableStorage().isOwner[owner] = true;
+192:     $.isOwner[owner] = true;
-193:     _getMultiOwnableStorage().ownerAtIndex[index] = owner;
+193:     $.ownerAtIndex[index] = owner;

```

## [G-04] Check `withdrawAmount - maxCost` for greater than `0` if it is `0` it can save 1 `Gsreset` (~2900 Gas) and 1 `Gcoldsload` (**Saves ~5000 Gas**)

_**From Ethereum yellow Paper** : `Gsreset` 2900 Gas , Paid for an SSTORE operation when the storage value’s zeroness remains unchanged or is set to zero._

Since if `withdrawAmount - maxCost` is 0 `_withdrawableETH[userOp.sender]` mapping will be updated by adding 0 which is unnecessary and only wastes gas. So it is recommended to check for `withdrawAmount - maxCost` greater than `0` then only add this into the mapping otherwise skip this mapping update and continue execution since `withdrawAmount - maxCost` is not storage var. so checking them will not cost more gas but updating the mapping by adding 0 will cost big chunk of gas.
Here zeroness remains unchanged by adding 0 to the mapping. So by adding the check we can avoid 1 `Gsreset`( ~2900 Gas) and 1 `Gcoldsload` (~2100 Gas) of reading `_withdrawableETH[userOp.sender]` unnecessarily when `withdrawAmount - maxCost` is 0.

### Saves 1 `Gsreset` (~2900 Gas) and 1 `Gcoldsload` (**Saves ~5000 Gas**)

```solidity
File : MagicSpend/MagicSpend.sol

138:  _withdrawableETH[userOp.sender] += withdrawAmount - maxCost;
```

[MagicSpend.sol#L138](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L138)

**Recommended Mitigation Steps:**

```diff
File : MagicSpend/MagicSpend.sol

+    if((withdrawAmount - maxCost) > 0){
+   _withdrawableETH[userOp.sender] += withdrawAmount - maxCost;
+   }
-138:  _withdrawableETH[userOp.sender] += withdrawAmount - maxCost;
```

## [G-05] Cache the resulted `keccak256 hash` of `constant string` instead of re-calculating the hash on every function call (**Saves ~50 Gas**)

Since string `"EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"` is constant so it's keccak256 hash will be same every time it is calculated.
So it is efficient to cache the result of `keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)")` in to a bytes32 type constant variable instead of re-calculating it on every function call. Saves it's hash calculating gas cost every time the `domainSeparator()` function will be called.

[keccak256 gas calculation](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a2-sha3)

Since it contains 3 32 byte words s approx 30 + 6\*3 becomes 48 and some mem cost so it will take approx ~50 Gas Every time.

### Saves ~ 50 Gas on every function call

```solidity
File : SmartWallet/ERC1271.sol

100:  function domainSeparator() public view returns (bytes32) {
        (string memory name, string memory version) = _domainNameAndVersion();
        return keccak256(
            abi.encode(
104:                keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
                keccak256(bytes(name)),
                keccak256(bytes(version)),
                block.chainid,
                address(this)
            )
        );
    }

```

[ERC1271.sol#L104](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L104)

**Recommended Mitigation Steps:**

```diff
File : SmartWallet/ERC1271.sol

+   bytes32  public constant DOMAIN_HASH= keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

   ...

100:  function domainSeparator() public view returns (bytes32) {
        (string memory name, string memory version) = _domainNameAndVersion();
        return keccak256(
            abi.encode(
- 104:                keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
+ 104:          DOMAIN_HASH
                keccak256(bytes(name)),
                keccak256(bytes(version)),
                block.chainid,
                address(this)
            )
        );
    }

```

## [G-06] `Switch` the order of `if` statement to fail early saves 1 function call half of the times where function have SLOAD (**Saves ~2100 Gas** Half of the times)

It is recommended to order checks from low to high gas consuming to revert early.

### Instance 1

First statement do a function call and second only checks stake variable so we can switch the order of these two if statements to save gas.

```solidity
File : MagicSpend/MagicSpend.sol

184:  if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
185:      revert InvalidSignature();
186:  }
187:
188:  if (block.timestamp > withdrawRequest.expiry) {
189:      revert Expired();
190:  }

```

[MagicSpend.sol#L184-L190](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L184-L190)

**Recommended Mitigation Steps:**

```diff
File : MagicSpend/MagicSpend.sol

+     if (block.timestamp > withdrawRequest.expiry) {
+        revert Expired();
+     }
184:  if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
185:      revert InvalidSignature();
186:  }
187:
-188:  if (block.timestamp > withdrawRequest.expiry) {
-189:      revert Expired();
-190:  }

```

## [G-07] Check `contract` balance before sending `ethers` to fail early.

To fail early check value is less than or equal to address(this).balance.

```solidity
File : SmartWallet/CoinbaseSmartWallet.sol

272:  function _call(address target, uint256 value, bytes memory data) internal {
273:     (bool success, bytes memory result) = target.call{value: value}(data);
274:      if (!success) {
275:          assembly ("memory-safe") {
276:            revert(add(result, 32), mload(result))
277:        }
278:    }
279:  }
```

[CoinbaseSmartWallet.sol#L272-L279](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L272-L279)

**Recommended Mitigation Steps:**

```diff
File : SmartWallet/CoinbaseSmartWallet.sol

272:  function _call(address target, uint256 value, bytes memory data) internal {
+      require(value <= address(this).balance),"Not enough balance";
273:     (bool success, bytes memory result) = target.call{value: value}(data);
274:      if (!success) {
275:          assembly ("memory-safe") {
276:            revert(add(result, 32), mload(result))
277:        }
278:    }
279:  }
```

## [G-08] Use unchecked where underflow not possible(**Gas Saved: ~150 Gas**)(_Missed by bot_)

Cache `n - r` in `unchecked` block to save 1 checked subtraction.

```solidity
File : FreshCryptoLib/FCL.sol

67:  x1 = addmod(x1, n - r, n);

```

[FCL.sol#L67](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L67)

**Recommended Mitigation Steps:**

```diff
File : FreshCryptoLib/FCL.sol

+   unchecked{
+    uint256 nSUBr = n - r;
+  }

-67:  x1 = addmod(x1, n - r, n);
+67:  x1 = addmod(x1, nSUBr, n);

```

## [G-09] Do not assign a variable with its default value

Do not assign `salt` and `extensions` variables with their default values.

```solidity
File : SmartWallet/ERC1271.sol

54: salt = salt; // `bytes32(0)`.
55: extensions = extensions; // `new uint256[](0)`.

```

[ERC1271.sol#L54-L55](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L54-L55)

## [G-10] Cache `address(this).balance` instead of recalculating it

```solidity
File : MagicSpend/MagicSpend.sol

133:  if (address(this).balance < withdrawAmount) {
134:     revert InsufficientBalance(withdrawAmount, address(this).balance);
135:     }
```

[MagicSpend.sol#L133-L135](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L133-L135)

**Recommended Mitigation Steps:**

```diff
File : MagicSpend/MagicSpend.sol

+      uint256 balanceOfAddressThis = address(this).balance;
-133:  if (address(this).balance < withdrawAmount) {
+133:  if (balanceOfAddressThis < withdrawAmount) {
-134:     revert InsufficientBalance(withdrawAmount, address(this).balance);
+134:     revert InsufficientBalance(withdrawAmount, balanceOfAddressThis);
135:     }
```

## [G-11] Use nested if and separate if instead of && and || respectively(_Instances Missed by bot_)

The code employs logical operators && (logical AND) and || (logical OR) to combine conditions within if statements. It's recommended to refactor these expressions by using nested if statements for logical AND operations and separate if statements for logical OR operations. This enhances code readability, modularity, and maintainability.

```solidity
File : FreshCryptoLib/FCL.sol

79:  if (((0 == x) && (0 == y)) || x == p || y == p) {
80:      return false;
81:    }
```

[FCL.sol#L79-L81](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L79-L81)

**Recommended Mitigation Steps:**

```diff
File : FreshCryptoLib/FCL.sol

-79:  if (((0 == x) && (0 == y)) || x == p || y == p) {
-80:      return false;
-81:    }

+ if (0 == x) {
+    if (0 == y) {
+        return false;
+    }
+ }
+ if (x == p || y == p) {
+    return false;
+}

```

## [G-12] Use `abi.encodePacked` instead of `abi.encode` (_Instances Missed by bot_)

By using abi.encodePacked, unnecessary padding is avoided, resulting in a more efficient encoding process and reducing gas consumption for transactions or contract deployments. This optimization aligns with best practices for gas-efficient smart contract development.

```solidity
File : MagicSpend/MagicSpend.sol

139:  context = abi.encode(maxCost, userOp.sender);
```

[MagicSpend.sol#L139](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L139)

**Recommended Mitigation Steps:**

Replace abi.encode with abi.encodePacked to optimize gas usage in encoding data.

```diff
File : MagicSpend/MagicSpend.sol

-139:  context = abi.encode(maxCost, userOp.sender);
+139:  context = abi.encodePacked(maxCost, userOp.sender);
```
