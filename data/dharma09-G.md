## SMART WALLET GAS OPTIMIZATION


**Note: The issues addressed here were not reported by the bot, for packing variables, notes explaining the how and why are included**
## [G-01] Pack structs by reducing variable sizes to save storage slots (saves 2 SLOTS: 4.2k Gas)

Note: The bot report did not cover these instances.

### Details
We can replaced string clientDataJSON with bytes32 clientDataJSONHash. Storing the hash instead of the full string can significantly reduce gas costs if the string is relatively large.

For challengeTndex and typeIndex we can reduce data types size which is easily acoomodate large number and we can also reduce size of r,s to store in slot.By doing this we can save 2 slot.

### Proof of Code
- [WebAuthn.sol#L21C4-L36C6](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L21C4-L36C6)
```solidity
File: src/WebAuthnSol/WebAuthn.sol
21:  struct WebAuthnAuth {
        /// @dev The WebAuthn authenticator data.
        ///      See https://www.w3.org/TR/webauthn-2/#dom-authenticatorassertionresponse-authenticatordata.
        bytes authenticatorData;
        /// @dev The WebAuthn client data JSON.
        ///      See https://www.w3.org/TR/webauthn-2/#dom-authenticatorresponse-clientdatajson.
        string clientDataJSON;
        /// @dev The index at which "challenge":"..." occurs in `clientDataJSON`.
        uint256 challengeIndex;
        /// @dev The index at which "type":"..." occurs in `clientDataJSON`.
        uint256 typeIndex;
        /// @dev The r value of secp256r1 signature
        uint256 r;
        /// @dev The s value of secp256r1 signature
        uint256 s;
    }
```
### Optimized code:

```diff
diff --git a/src/WebAuthnSol/WebAuthn.sol b/src/WebAuthnSol/WebAuthn.sol
index a0b29e5..3887265 100644
--- a/src/WebAuthnSol/WebAuthn.sol
+++ b/src/WebAuthnSol/WebAuthn.sol
@@ -24,15 +24,15 @@ library WebAuthn {
         bytes authenticatorData;
         /// @dev The WebAuthn client data JSON.
         ///      See https://www.w3.org/TR/webauthn-2/#dom-authenticatorresponse-clientdatajson.
-        string clientDataJSON;
+        bytes32 clientDataJSON;
         /// @dev The index at which "challenge":"..." occurs in `clientDataJSON`.
-        uint256 challengeIndex;
+        uint128 challengeIndex;
         /// @dev The index at which "type":"..." occurs in `clientDataJSON`.
-        uint256 typeIndex;
+        uint128 typeIndex;
         /// @dev The r value of secp256r1 signature
-        uint256 r;
+        uint128 r;
         /// @dev The s value of secp256r1 signature
-        uint256 s;
+        uint128 s;
     }
```
## [G-02] State variables can be packed into fewer storage slot (saves 1 SLOTS: 2.1k Gas)
Note: The bot report did not cover these instances.

### Pack the following by reducing their size (saves 2.1k Gas)

### Proof of Code
- [WebAuthn.sol#L40C4-L55C86](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L40C4-L55C86)
```solidity
File: src/WebAuthnSol/WebAuthn.sol
40: bytes1 private constant AUTH_DATA_FLAGS_UP = 0x01;

    /// @dev Bit 2 of the authenticator data struct, corresponding to the "User Verified" bit.
    ///      See https://www.w3.org/TR/webauthn-2/#flags.
    bytes1 private constant AUTH_DATA_FLAGS_UV = 0x04;

```
### Optimized code:

```diff
diff --git a/src/WebAuthnSol/WebAuthn.sol b/src/WebAuthnSol/WebAuthn.sol
index a0b29e5..22860c6 100644
--- a/src/WebAuthnSol/WebAuthn.sol
+++ b/src/WebAuthnSol/WebAuthn.sol
@@ -43,13 +43,13 @@ library WebAuthn {
     ///      See https://www.w3.org/TR/webauthn-2/#flags.
     bytes1 private constant AUTH_DATA_FLAGS_UV = 0x04;
 
-    /// @dev Secp256r1 curve order / 2 used as guard to prevent signature malleabi
lity issue.
-    uint256 private constant P256_N_DIV_2 = FCL.n / 2;
-
     /// @dev The precompiled contract address to use for signature verification in
 the “secp256r1” elliptic curve.
     ///      See https://github.com/ethereum/RIPs/blob/master/RIPS/rip-7212.md.
     address private constant VERIFIER = address(0x100);
 
+    /// @dev Secp256r1 curve order / 2 used as guard to prevent signature malleabi
lity issue.
+    uint256 private constant P256_N_DIV_2 = FCL.n / 2;
+
     /// @dev The expected type (hash) in the client data JSON when verifying asser
tion signatures.
     ///      See https://www.w3.org/TR/webauthn-2/#dom-collectedclientdata-type
     bytes32 private constant EXPECTED_TYPE_HASH = keccak256('"type":"webauthn.get"
');
```

## [G-03] Cache calculations in loop to avoid re-calculating on each iteration

### Details
In `CoinbaseSmartWallet.sol:executeBatch()` : `calls[i]` we can cache once and use multiple times. Instead of repeatedly calling calls[i] in the loop, we can calculate it once and use the result.

### Proof of Code
- [CoinbaseSmartWallet.sol#L205C2-L212C6](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L205C2-L212C6)
```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
205:  function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
        for (uint256 i; i < calls.length;) {
            _call(calls[i].target, calls[i].value, calls[i].data); //@audit cache calls[i]
            unchecked {
                ++i;
            }
        }
    }
```
### Optimized code:

```diff
diff --git a/src/SmartWallet/CoinbaseSmartWallet.sol b/src/SmartWallet/CoinbaseSmartWallet.sol
index 2278a5a..1d09e54 100644
--- a/src/SmartWallet/CoinbaseSmartWallet.sol
+++ b/src/SmartWallet/CoinbaseSmartWallet.sol
@@ -204,7 +204,8 @@ contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271
     /// @param calls The list of `Call`s to execute.
     function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
         for (uint256 i; i < calls.length;) {
-            _call(calls[i].target, calls[i].value, calls[i].data);
+            uint256 _calls = calls[i];
+            _call(_calls.target, _calls.value, _calls.data);
             unchecked {
                 ++i;
             }
```
### Instances 2
In `MultiOwnable.sol:_initializeOwners()` : `owners.length[i]` we can cache once and use multiple times. Instead of repeatedly calling owners.length[i] in the loop, we can calculate it once and use the result.

### Proof of Code
- [MultiOwnable.sol#L162C3-L174C6](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L162C3-L174C6)
```solidity
File: src/SmartWallet/MultiOwnable.sol
162: function _initializeOwners(bytes[] memory owners) internal virtual {
        for (uint256 i; i < owners.length; i++) {
            if (owners[i].length != 32 && owners[i].length != 64) {
                revert InvalidOwnerBytesLength(owners[i]); //@audit
            }

            if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
                revert InvalidEthereumAddressOwner(owners[i]);
            }

            _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
        }
    }
```
### Optimized code:

```diff
diff --git a/src/SmartWallet/MultiOwnable.sol b/src/SmartWallet/MultiOwnable.sol
index 64fd19a..9a46f34 100644
--- a/src/SmartWallet/MultiOwnable.sol
+++ b/src/SmartWallet/MultiOwnable.sol
@@ -161,11 +161,12 @@ contract MultiOwnable {
     /// @param owners The intiial list of owners to register.
     function _initializeOwners(bytes[] memory owners) internal virtual {
         for (uint256 i; i < owners.length; i++) {
-            if (owners[i].length != 32 && owners[i].length != 64) {
+            uint256 owners_ = owners.length;
+            if (owners_ != 32 && owners_ != 64) {
                 revert InvalidOwnerBytesLength(owners[i]);
             }
 
-            if (owners[i].length == 32 && uint256(bytes32(own
ers[i])) > type(uint160).max) {
+            if (owners_ == 32 && uint256(bytes32(owners[i])) 
> type(uint160).max) {
                 revert InvalidEthereumAddressOwner(owners[i])
;
             }
```