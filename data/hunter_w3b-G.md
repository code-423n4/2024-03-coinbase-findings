# Gas-Optimization

## [G-01] Use `immutable` for the `entryPoint` address

Since the `entryPoint` address is constant and set at construction, use the immutable keyword to save gas

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L217-L219

```solidity
    function entryPoint() public view virtual returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304-L306

```solidity
    function entryPoint() public pure returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }
```

```diff
+  address immutable ENTRY_POINT = 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;

+  function entryPoint() public view returns (address) {
+      return ENTRY_POINT;
}
```

## [G-02] Use unchecked arithmetic can't overflow because of prevois if statment

Use the unchecked keyword to disable overflow and underflow checks for arithmetic operations that cannot underflow or overflow.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L133-L138

```solidity
        if (address(this).balance < withdrawAmount) {
            revert InsufficientBalance(withdrawAmount, address(this).balance);
        }

        // NOTE: Do not include the gas part in withdrawable funds as it will be handled in `postOp()`.
        _withdrawableETH[userOp.sender] += withdrawAmount - maxCost;
```

## [G-03] Optimizing `MultiOwnable.sol::isOwner` Mapping

Instead of storing the owner's raw bytes as keys in the `isOwner` mapping, which can lead to high storage costs and gas usage, we can store a hash of the owner's bytes and use that as the key.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L24

Replace:

```solidity
    mapping(bytes account => bool isOwner_) isOwner;
```

With:

```solidity
mapping(bytes32 => bool) isOwnerHash;
```

And update functions accordingly to use hashes.

## [G-04] We can Avoid using libraries for simple functions

Avoid using libraries for simple functions that can be implemented in the contract. This way, you can avoid the extra SLOAD and SSTORE operations required to store the library address and call the library functions.

For example, instead of using the `SafeTransferLib` library for simple ETH transfers, you can implement the transfer function directly in the contract.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L334

```solidity
161            SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);

212    function entryPointDeposit(uint256 amount) external payable onlyOwner {
213        SafeTransferLib.safeTransferETH(entryPoint(), amount);
214    }


334    function _withdraw(address asset, address to, uint256 amount) internal {
335        if (asset == address(0)) {
336            SafeTransferLib.safeTransferETH(to, amount);
337        } else {
338            SafeTransferLib.safeTransfer(asset, to, amount);
339        }
```

## [G-05] We can Reducing SLOAD Operations in `MultiOwnable::removeOwnerAtIndex` function

Update functions to avoid redundant storage reads. For example, cache commonly used values locally to avoid multiple SLOAD operations.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L102

Before:

```solidity
function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {
    bytes memory owner = ownerAtIndex(index);
    // Use owner...
}
```

After:

```solidity
function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {
    bytes storage owner = _getMultiOwnableStorage().ownerAtIndex[index];
    // Use owner...
}
```

## [G-06] storage layout can be optimized `MultiOwnable::MultiOwnableStorage` struct

The storage layout can be optimized by rearranging the order of struct members to reduce SLOAD operations.

Instead of:

```solidity
struct MultiOwnableStorage {
    uint256 nextOwnerIndex;
    mapping(uint256 index => bytes owner) ownerAtIndex;
    mapping(bytes account => bool) isOwner_;
}
```

Consider:

```solidity
struct MultiOwnableStorage {
    mapping(uint256 => bytes) ownerAtIndex;
    mapping(bytes32 => bool) isOwnerHash;
    uint256 nextOwnerIndex;
}
```

This layout minimizes the number of `SLOADs` required to access storage variables.

## [G‑07] INTERNAL functions only called once can be inlined to save Gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
File: src/SmartWallet/MultiOwnable.sol

201    function _checkOwner() internal view virtual {

```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L201

```solidity
File: src/SmartWallet/ERC1271.sol

121    function _eip712Hash(bytes32 hash) internal view virtual returns (bytes32) {

133    function _hashStruct(bytes32 hash) internal view virtual returns (bytes32) {

```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L121

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

291    function _validateSignature(bytes32 message, bytes calldata signature)
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291

## [G-08] A MODIFIER used only once and not being inherited should be inlined to save gas

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#-L100

```solidity
    modifier payPrefund(uint256 missingAccountFunds) virtual {
        _;

        assembly ("memory-safe") {
            if missingAccountFunds {
                // Ignore failure (it's EntryPoint's job to verify, not the account's).
                pop(call(gas(), caller(), missingAccountFunds, codesize(), 0x00, codesize(), 0x00))
            }
        }
    }
```

**This modifier is only used in this function it should inilned to save extra JUMP instructions and additional stack operations**

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L142

```solidity
    function validateUserOp(UserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)
        public
        payable
        virtual
        onlyEntryPoint
        payPrefund(missingAccountFunds)
        returns (uint256 validationData)
    {
```

## [G-09] Expressions for constant values such as a call to  KECCAK256(), should use IMMUTABLE rather than CONSTANT

```solidity
File: src/SmartWallet/ERC1271.sol

23    bytes32 private constant _MESSAGE_TYPEHASH = keccak256("CoinbaseSmartWalletMessage(bytes32 hash)");
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L23

```solidity
File: src/WebAuthnSol/WebAuthn.sol

55    bytes32 private constant EXPECTED_TYPE_HASH = keccak256('"type":"webauthn.get"');
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L55

## [G-10] Assigning STATE VARIABLES directly with named struct constructors wastes gas

Using named arguments for struct means that the compiler needs to organize the fields in memory before doing the assignment, which wastes gas. Set each field directly in storage (use dot-notation), or use the unnamed version of the constructor.

Leverage mapping and dot notation for struct assignment

In dot notation, values are directly written to storage variable, When we use the current method in the code the compiler will allocate some memory to store the struct instance first before writing it to storage.

```solidity
File: src/SmartWallet/ERC1271.sol

70        if (_validateSignature({message: replaySafeHash(hash), signature: signature})) {
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L70

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

321            return WebAuthn.verify({challenge: abi.encode(message), requireUV: false, webAuthnAuth: auth, x: x, y: y});
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L321

## [G-11] Access MAPPINGS directly rather than using accessor functions

When you have a mapping, accessing its values through accessor functions involves an additional layer of indirection, which can incur some gas cost. This is because accessing a value from a mapping typically involves two steps: first, locating the key in the mapping, and second, retrieving the corresponding value.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L299-L301

```solidity
    function nonceUsed(address account, uint256 nonce) external view returns (bool) {
        return _nonceUsed[nonce][account];
    }
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L117-L148

```solidity
    function isOwnerAddress(address account) public view virtual returns (bool) {
        return _getMultiOwnableStorage().isOwner[abi.encode(account)];
    }

    function isOwnerPublicKey(bytes32 x, bytes32 y) public view virtual returns (bool) {
        return _getMultiOwnableStorage().isOwner[abi.encode(x, y)];
    }

    function isOwnerBytes(bytes memory account) public view virtual returns (bool) {
        return _getMultiOwnableStorage().isOwner[account];
    }

    function ownerAtIndex(uint256 index) public view virtual returns (bytes memory) {
        return _getMultiOwnableStorage().ownerAtIndex[index];
    }
```

## [G-12] Shorten the array rather than copying to a new one

ASSEMBLY can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104

```solidity
        bytes[] memory owners = new bytes[](1);
```

## [G-13] Expensive operation inside a for loop

The loop is very expensive in every iteration has external call and send value.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L205-L212

```solidity
    function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
        for (uint256 i; i < calls.length;) {
            _call(calls[i].target, calls[i].value, calls[i].data);
            unchecked {
                ++i;
            }
        }
    }
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L272-L279

```solidity
    function _call(address target, uint256 value, bytes memory data) internal {
        (bool success, bytes memory result) = target.call{value: value}(data);
        if (!success) {
            assembly ("memory-safe") {
                revert(add(result, 32), mload(result))
            }
        }
    }
```
