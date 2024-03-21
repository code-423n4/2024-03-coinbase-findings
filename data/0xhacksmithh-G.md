### [Gas-01] Instaed of increamenting state variable in every loop we could keep track of number of increament, and push result to storage with one write

below we can see that `_getMultiOwnableStorage().nextOwnerIndex` get incremented by 1, in every iteration of looop.

Instead we can do like 
1. cache `_getMultiOwnableStorage().nextOwnerIndex` before loop, and this cache value used in loop and get incremented in each iteration.
2. at end of loop we simply update `_getMultiOwnableStorage().nextOwnerIndex` with that cached value.

```solidity
    function _initializeOwners(bytes[] memory owners) internal virtual {
+       uint256 idx = _getMultiOwnableStorage().nextOwnerIndex;
        for (uint256 i; i < owners.length; i++) {
            if (owners[i].length != 32 && owners[i].length != 64) { 
                revert InvalidOwnerBytesLength(owners[i]);
            }

            if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
                revert InvalidEthereumAddressOwner(owners[i]);
            }

 -          _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++); // @audit G::      

 +          _addOwnerAtIndex(owners[i], idx + 1);
  }

+      _getMultiOwnableStorage().nextOwnerIndex = idx;
    }
```
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L172
```

### [Gas-02] `owners[i]` could cached in memory rather than calculating it 5 times

```solidity
    function _initializeOwners(bytes[] memory owners) internal virtual {
+       bytes owner;
        for (uint256 i; i < owners.length; i++) {
+           owner = owners[i];
-           if (owners[i].length != 32 && owners[i].length != 64) { 
+           if (owner.length != 32 && owner.length != 64) { 
                revert InvalidOwnerBytesLength(owners[i]);
            }

            if (owner.length == 32 && uint256(bytes32(owner)) > type(uint160).max) {
                revert InvalidEthereumAddressOwner(owners[i]);
            }

            _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++); 
        }
    }
```
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L164-L172
```

### [Gas-03] Multiple address/ID mappings can be combined into a single mapping of an address/ID to struct

```solidity
    mapping(uint256 nonce => mapping(address user => bool used)) internal _nonceUsed;
```
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L37
```


### [Gas-04] In case of calculation of `keccak256(` instead of `constant`, `immutable` could be used

```solidity
-   bytes32 private constant _MESSAGE_TYPEHASH = keccak256("CoinbaseSmartWalletMessage(bytes32 hash)");

+   bytes32 private immutable _MESSAGE_TYPEHASH = keccak256("CoinbaseSmartWalletMessage(bytes32 hash)");
```
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L23
```



