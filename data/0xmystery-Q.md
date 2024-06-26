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
## [L-05] Adapting to L2's decentralized sequencing: Navigating New Frontiers in Transaction Fairness
As Layer 2 like Base considers moving towards a more decentralized sequencer model, the platform faces the challenge of maintaining its current mitigation of frontrunning risks inherent in a "first come, first served" system. The transition could reintroduce vulnerabilities to transaction ordering manipulation, demanding innovative solutions to uphold transaction fairness. Strategies such as commit-reveal schemes, submarine sends, Fair Sequencing Services (FSS), decentralized MEV mitigation techniques, and the incorporation of time-locks and randomness could play pivotal roles. These measures aim to preserve the integrity of transaction sequencing, ensuring that the L2's evolution towards decentralization enhances its ecosystem without compromising the security and fairness that are crucial for user trust and platform reliability.

## [L-06] Possioble deployment and functional failure of Contracts on L2s due to Dencun Opcodes
With the protocol intending to operate on various L2 chains as is inferred from [executeWithoutChainIdValidation()](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L170-L187), it may encounter deployment and operational challenges if utilizing any of the new opcodes that enhance Ethereum's functionality, such as those related to shard blob transactions (EIP-4844) and others as described in the link below:

https://www.coinlive.com/news/ethereum-dencun-hard-fork-content-introduction

Devoid of `Dencun upgrade`, contracts relying on the new opcodes will likely fail because the Ethereum Virtual Machine (EVM) on these L2s would not recognize or know how to execute the new instructions, leading to reverts or other unexpected behaviors.  

As of todate, Optimism, Arbitrum, and Base have implemented the Dencun upgrade. Consider implementing conditional logic in contracts or hold off deploying to L2 that hasn't had the upgrade updated.

## [L-07] Lack of EIP-712 compliance when using `keccak256()` directly on an array or a struct variable
Directly using the actual variable instead of encoding the array values goes against the EIP-712 specification according to the link below:

https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md#definition-of-encodedata

**Note**: OpenSea's [Seaport's example with offerHashes and considerationHashes](https://github.com/ProjectOpenSea/seaport/blob/a62c2f8f484784735025d7b03ccb37865bc39e5a/reference/lib/ReferenceGettersAndDerivers.sol#L130-L131) can be used as a reference to understand how array or structs should be encoded.

Here's an instance entailed where the bytes array, `owners` should have been separately encoded first:

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L81-L83

```solidity
    function _getSalt(bytes[] calldata owners, uint256 nonce) internal pure returns (bytes32 salt) {
        salt = keccak256(abi.encode(owners, nonce));
    }
```
## [NC-01] Typographical error in constant variable name
The constant variable `MUTLI_OWNABLE_STORAGE_LOCATION` in the MultiOwnable contract contains a typographical error, where "MULTI" is misspelled as "MUTLI". While being non-critical and not impacting the functionality, security, or performance of the contract, the typo could potentially cause mild confusion or readability issues for developers/users reviewing or interacting with the code. Correcting the spelling would enhance code clarity and maintain naming consistency without affecting the contract's execution or behavior.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L36-L37

```diff
-    bytes32 private constant MUTLI_OWNABLE_STORAGE_LOCATION =
+    bytes32 private constant MULTI_OWNABLE_STORAGE_LOCATION =
        0x97e2c6aad4ce5d562ebfaa00db6b9e0fb66ea5d8162ed5b243f51a2e03086f00;
```
## [NC-02] Comment mismatch on future enhanced asset support in MagicSpend
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
## [NC-04] Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that is more efficient on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier below may be refactored as follows:

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L92-L96

```diff
+    function _onlyEntryPoint() private view {
+        if (msg.sender != entryPoint()) revert Unauthorized();
+    }

     modifier onlyEntryPoint() {
-        if (msg.sender != entryPoint()) revert Unauthorized();
+        _onlyEntryPoint();
     _;
     }
```
## [NC-05] Activate the Optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.23",
settings: {
optimizer: {
  enabled: true,
  runs: 1000,
},
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.   