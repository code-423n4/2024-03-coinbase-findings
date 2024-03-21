## [L-01] NatSpec comment on 255 owner limitation in `_addOwner()` being ambiguous
The discrepancy between the NatSpec comment in the `_addOwner` function of the MultiOwnable contract and its actual implementation could lead to misunderstandings about the contract's owner limit. The comment suggests a limit of 255 owners,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L176

```solidity
    /// @notice Convenience function used to add the first 255 owners.
```
but the code, utilizing `uint256` for indexing, 

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L8-L9

```solidity
    /// @dev Tracks the index of the next owner to add.
    uint256 nextOwnerIndex;
```
does not enforce any such limit, allowing for a significantly larger number of owners.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L179-L181

```solidity
    function _addOwner(bytes memory owner) internal virtual {
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```
This issue, while not affecting the contract's functionality or security directly, could cause confusion among developers and users, leading to potential misalignments in expectations and implementations. Addressing this through clarification in the documentation or by introducing an explicit limit, if desired, is recommended to ensure clear and accurate communication of the contract's capabilities. 

```diff
    function _addOwner(bytes memory owner) internal virtual {
+        require(_getMultiOwnableStorage().nextOwnerIndex < 255, "Owner limit reached");
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```
## [L-02] Risks associated with forceful ETH transfers to unprepared contracts
Forcing ETH transfers to contracts that are not designed to receive them, due to the absence of a `payable fallback` or `receive` function, can lead to permanently locked funds within these recipient contracts. This situation, facilitated by low-level operations like `selfdestruct`, presents a significant severity issue that undermines the predictability and safety of smart contract interactions in the Ethereum ecosystem. To mitigate such risks, it is recommended that sending contracts verify the recipient's ability to handle ETH in a standard manner and that developers design contracts with interoperability and safe ETH handling in mind, ensuring the broader reliability and trustworthiness of the ecosystem.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L158-L162

```solidity
        // Send the all remaining funds to the user accout.
        delete _withdrawableETH[account];
        if (withdrawable > 0) {
            SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);
        }
```
## [L-03] Limitations in fund withdrawal for non-gas and multi-asset transactions
The contract `MagicSpend` presents two primary limitations impacting its flexibility and utility in broader transaction contexts. First, there's a procedural limitation where, after the invocation of `validatePaymasterUserOp()` by the EntryPoint, the subsequent need for non-gas related fund withdrawals (e.g., for swaps or mints) within the same transaction is constrained to using `withdrawGasExcess()`. Attempting to invoke `withdraw()` would result in failure due to the nonce already being marked as used, restricting access to additional funds beyond the initial gas validation context. Second, there's a functional limitation where `withdrawGasExcess()` is strictly designed for ETH withdrawals, lacking the capability to handle ERC-20 tokens. This is in contrast to `withdraw()`, which is designed with the flexibility to manage both ETH and ERC-20 token withdrawals. Together, these limitations could significantly impact the contract's applicability in complex transaction scenarios requiring multi-asset interactions and nuanced fund management.

1. A user operation requires validation for gas coverage via [validatePaymasterUserOp()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L109-L140), which also marks the operation's nonce as used.
2. Post-validation, the operation requires additional funds for non-gas related activities (e.g., token swaps). However, due to the nonce already being marked as used, [withdraw()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L178-L194) cannot be invoked for this purpose.
3. Furthermore, should these additional activities require assets other than ETH, the contract's design restricts the user to [withdrawGasExcess()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L165-L176), which only supports ETH, thereby limiting multi-asset transaction capabilities within the same operation context.

Consider the following fix:
Enhance `withdrawGasExcess()` to support ERC-20 tokens in addition to ETH. This could involve introducing additional parameters or overloading the function to handle different asset types, ensuring broader applicability in multi-asset transaction contexts.

## [L-04] Missing check for passkey associated with `CoinbaseSmartWallet._validateSignature()`
Neither passkey nor an address goes through checks as implemented in [MultiOwnable._initializeOwners()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L162-L174) when added through `addOwnerAddress()` and `addOwnerPublicKey()` that have `_addOwner()` invoked,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L179-L181

```solidity
    function _addOwner(bytes memory owner) internal virtual {
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```
When `CoinbaseSmartWallet._validateSignature()` is called, a needed check is performed when `ownerBytes.length == 32` but not so when `ownerBytes.length == 64`. Consider adding the needed check for a consistent earlier revert as follows:

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L316-L324

```diff
-        if (ownerBytes.length == 64) {
+        if (owners[i].length != 64) {
+                revert InvalidOwnerBytesLength(owners[i]);
+            }
            (uint256 x, uint256 y) = abi.decode(ownerBytes, (uint256, uint256));

            WebAuthn.WebAuthnAuth memory auth = abi.decode(sigWrapper.signatureData, (WebAuthn.WebAuthnAuth));

            return WebAuthn.verify({challenge: abi.encode(message), requireUV: false, webAuthnAuth: auth, x: x, y: y});
-        }

-        revert InvalidOwnerBytesLength(ownerBytes);
```
## [NC-01] Typographical error in constant variable name
The constant variable `MUTLI_OWNABLE_STORAGE_LOCATION` in the MultiOwnable contract contains a typographical error, where "MULTI" is misspelled as "MUTLI". While being non-critical and not impacting the functionality, security, or performance of the contract, the typo could potentially cause mild confusion or readability issues for developers/users reviewing or interacting with the code. Correcting the spelling would enhance code clarity and maintain naming consistency without affecting the contract's execution or behavior.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L36-L37

```diff
-    bytes32 private constant MUTLI_OWNABLE_STORAGE_LOCATION =
+    bytes32 private constant MULTI_OWNABLE_STORAGE_LOCATION =
        0x97e2c6aad4ce5d562ebfaa00db6b9e0fb66ea5d8162ed5b243f51a2e03086f00;
```
## [NC-02] Enhanced asset support in MagicSpend
MagicSpend.sol, initially documented to support only ETH withdrawals, inherently possesses a broader capability to manage multiple asset types, including ERC-20 tokens, through its versatile `_withdraw()` function. 

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L326-L340

```solidity
    /// @notice Withdraws funds from this contract.
    ///
    /// @dev Callers MUST validate that the withdraw is legitimate before calling this method as
    ///      no validation is performed here.
    ///
    /// @param asset  The asset to withdraw.
    /// @param to     The beneficiary address.
    /// @param amount The amount to withdraw.
    function _withdraw(address asset, address to, uint256 amount) internal {
        if (asset == address(0)) {
            SafeTransferLib.safeTransferETH(to, amount);
        } else {
            SafeTransferLib.safeTransfer(asset, to, amount);
        }
    }
```
This flexibility underscores the contract's design for comprehensive asset management, necessitating an update in the documentation to accurately reflect its multi-asset support capabilities. Such an update would clarify the contract's functionality, aligning the documentation with the implemented code and ensuring a clear understanding of its capabilities for developers and auditors alike. 

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L23

```diff
-        /// @dev The asset to withdraw. NOTE: Only ETH (associated with zero address) is supported for now.
+        /// @dev The asset to withdraw. Initially designed to support only ETH (associated with zero address),
+        /// but the contract is equipped to handle a broader range of assets, including ERC-20 tokens, as evidenced
+        /// by the flexibility of the _withdraw() function.
```
## [NC-03] Incorrect comment associated with `CoinbaseSmartWallet._validateSignature()`
The comment below is inaccurate,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L301-L306

```solidity
        if (ownerBytes.length == 32) {
            if (uint256(bytes32(ownerBytes)) > type(uint160).max) {
                // technically should be impossible given owners can only be added with @audit
                // addOwnerAddress and addOwnerPublicKey, but we leave incase of future changes. @audit
                revert InvalidEthereumAddressOwner(ownerBytes);
            }
```
given the fact that `addOwnerAddress()` and `addOwnerPublicKey()` both invoking `_addOwner()`,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L179-L181

```solidity
    function _addOwner(bytes memory owner) internal virtual {
        _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
    }
```
do not have checks as implemented in `_initializeOwners()`,

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L168-L170

```solidity
            if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
                revert InvalidEthereumAddressOwner(owners[i]);
            }
```
   