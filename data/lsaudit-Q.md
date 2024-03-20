# [1] `MultiOwnable.sol` misses functionality which allows to return the current number of owners

**File:** `MultiOwnable.sol`

[File: src/SmartWallet/MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L179)
```solidity
179:     function _addOwner(bytes memory owner) internal virtual {
180:         _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
181:     }
```

Whenever new owner is being added, `_getMultiOwnableStorage().nextOwnerIndex` is being increased (line 180).
However, removing the owner - does not decrease `_getMultiOwnableStorage().nextOwnerIndex`. While this issue is not a security per se (removing the owner should not decrease `_getMultiOwnableStorage().nextOwnerIndex`) - it's important to notice, that `MultiOwnable.sol` misses a functionality which returns the number of owners.

`_getMultiOwnableStorage().nextOwnerIndex` is being increases every time new owner is being added - however, it cannot be used to estimate how many owners are currently set (because removing the owner does not decrease `_getMultiOwnableStorage().nextOwnerIndex`). Protocol does not implement any other function which might return the current number of owners.

Our recommendation is to implement additional counter - which will allow to follow how many current owners are being set. This counter needs to be increased every time new owner is being added - and decreased - whenever owner is being removed.


# [2] Incorrect comment in `ERC1271.sol`

**File:** `ERC1271.sol`

[File: src/SmartWallet/ERC1271.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L154)
```solidity
154:     /// @return `true` is the signature is valid, else `false`.
155:     function _validateSignature(bytes32 message, bytes calldata signature) internal view virtual returns (bool);
```

According to NatSpec, function should return `true` when the signature is valid - or `false` - otherwise.
Function `_validateSignature` is being overridden in `CoinbaseSmartWallet.sol` - which is ERC-4337. According to EIP-4337: `If the account does not support signature aggregation, it MUST validate the signature is a valid signature of the userOpHash, and SHOULD return SIG_VALIDATION_FAILED (and not revert) on signature mismatch. Any other error MUST revert`. Let's focus on the last part of this sentence: `Any other error MUST revert`.
This sentence straightforwardly implies, that `_validateSignature` returns `true` when the signature is valid, returns `false` when signature is not valid and reverts on any other error.
The NatSpec of `_validSignature` misses the information about the revert. When we examine the `_validateSignature` in `CoinbaseSmartWallet.sol` - we can indeed notice, that this function might revert with either `InvalidEthereumAddressOwner()` or `InvalidOwnerBytesLength()` errors. This case should be mentioned in the NatSpec.

Our recommendation is to change above NatSpec to:

```
/// @return `true` is the signature is valid, return `false` when the signature is not valid, revert at any other error.
```


# [3] Change `+=` operator to avoid potential revert

**File:** `MagicSpend.sol`

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L138)
```solidity
138:         _withdrawableETH[userOp.sender] += withdrawAmount - maxCost;
```

When `maxCost < withdrawAmount`, function will revert due to underflow. The underflow might occurs, because Solidity firstly tries to calculate the `withdrawAmount - maxCost` (which might underflow) and then adds and assigns the result to `_withdrawableETH[userOp.sender]`.
To avoid potential revert, it's better to avoid using `+=` operator.

```
_withdrawableETH[userOp.sender] = _withdrawableETH[userOp.sender] + withdrawAmount - maxCost;
```

In the above scenario, Solidity will firstly add `withdrawAmount` to `_withdrawableETH[userOp.sender]` and then subtract `maxCost`.


# [4] Function `nonceUsed()` should be changed to `isNonceUsed()`

**File:** `MagicSpend.sol`

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L299)
```solidity
299:     function nonceUsed(address account, uint256 nonce) external view returns (bool) {
300:         return _nonceUsed[nonce][account];
301:     }
```

Function `nonceUsed()` verifies if the nonce has been already used (`true` - when nonce is used, `false` - otherwise). This suggests, that the more appropriate function name would be `isNonceUsed()`.

# [5] Create additional comment in `createAccount()` to fully describe function behavior

**File:** `CoinbaseSmartWalletFactory.sol`

According to [EIP-4337](https://eips.ethereum.org/EIPS/eip-4337#first-time-account-creation):

```
it’s expected to return the wallet address even if the wallet has already been created. This is to make it easier for clients to query the address without knowing if the wallet has already been deployed, by simulating a call to entryPoint.getSenderAddress(), which calls the factory under the hood.
```

[File: src/SmartWallet/CoinbaseSmartWalletFactory.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L48)
```solidity
48:         (bool alreadyDeployed, address accountAddress) =
49:             LibClone.createDeterministicERC1967(msg.value, implementation, _getSalt(owners, nonce));
```

Since function `createAccount()` uses external library from Solady: `LibClone.createDeterministicERC1967()` - it's a good coding practice to explicitly mention in the comment section - that `LibClone.createDeterministicERC1967()` won't revert.
This might be confirmed when we'll take a look at the `LibClone.createDeterministicERC1967()` implementation:

```
source: https://github.com/Vectorized/solady/blob/c6738e40225288842ce890cd265a305684e52c3d/src/utils/LibClone.sol#L829
    /// Note: This method is intended for use in ERC4337 factories,
    /// which are expected to NOT revert if the proxy is already deployed.
    function createDeterministicERC1967(uint256 value, address implementation, bytes32 salt)
```

Our recommendation is to add additional comment, before calling `LibClone.createDeterministicERC1967()` - that this function is expected to NOT revert if the proxy is already deployed.


# [6] Protocol does not support the latest Entry Point

**Files:** `MagicSpend.sol`, `CoinbaseSmartWallet.sol`

[File: src/SmartWallet/CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L217)
```solidity
217:     function entryPoint() public view virtual returns (address) {
218:         return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789; 
219:     }
```

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304)
```solidity
304:     function entryPoint() public pure returns (address) {
305:         return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
306:     }
```

The `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789` address points to Entry Point 0.6.0 - while the latest Entry Point version is [0.7.0](https://github.com/eth-infinitism/account-abstraction/releases/tag/v0.7.0).

# [7] Implement ability to update Entry Point address

**Files:** `MagicSpend.sol`, `CoinbaseSmartWallet.sol`

[File: src/SmartWallet/CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L217)
```solidity
217:     function entryPoint() public view virtual returns (address) {
218:         return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789; 
219:     }
```

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304)
```solidity
304:     function entryPoint() public pure returns (address) {
305:         return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
306:     }
```

The Entry Point address points to the constant value (`0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789`) and cannot be changed. If Entry Point address changes (e.g., because of a bug) - it won't be possible to update it without upgrading the contract.
Our recommendation is to store Entry Point address in a state variable and create an `onlyOwner` function which would allow to update it when needed.

# [8] Improve the code readability by using `getAddress()` in `createAccount()`

**File:** `CoinbaseSmartWalletFactory.sol`

According to [EIP-4337](https://eips.ethereum.org/EIPS/eip-4337#first-time-account-creation):

```
it’s expected to return the wallet address even if the wallet has already been created. This is to make it easier for clients to query the address without knowing if the wallet has already been deployed, by simulating a call to entryPoint.getSenderAddress(), which calls the factory under the hood.
```

[File: src/SmartWallet/CoinbaseSmartWalletFactory.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L48)
```solidity
48:         (bool alreadyDeployed, address accountAddress) =
49:             LibClone.createDeterministicERC1967(msg.value, implementation, _getSalt(owners, nonce)); 
50: 
51:         account = CoinbaseSmartWallet(payable(accountAddress));
```

Even though this requirement is fulfilled, the code readability would be much more improved when function `getAddress()` would be utilized.
For the reference, please take a look how it's done in `eth-infinitism/account-abstraction`:

```
source: https://github.com/eth-infinitism/account-abstraction/blob/7af70c8993a6f42973f520ae0752386a5032abe7/contracts/samples/SimpleAccountFactory.sol#L28

    function createAccount(address owner,uint256 salt) public returns (SimpleAccount ret) {
        address addr = getAddress(owner, salt);
        uint256 codeSize = addr.code.length;
        if (codeSize > 0) {
            return SimpleAccount(payable(addr));
        }
        ret = SimpleAccount(payable(new ERC1967Proxy{salt : bytes32(salt)}(
                address(accountImplementation),
                abi.encodeCall(SimpleAccount.initialize, (owner))
            )));
    }
```

Function utilizes `getAddress()` to get the address of the contract, and then - it checks if the contract is indeed deployed. Separating this logic will improve the code readability, thus it's highly recommended.

# [9] Typos

**Files:** `MultiOwnable.sol`, `MagicSpend.sol`, `CoinbaseSmartWallet.sol`, `ERC1271.sol`

[File: src/SmartWallet/MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L10)
```solidity
10:     /// @dev Mapping of indices to raw owner bytes, used to idenfitied owners by their
```

`idenfitied` should be changed to `identified`.

[File: src/SmartWallet/MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L52)
```solidity
52:     /// @notice Thrown when trying to intialize the contracts owners if a provided owner is neither
```

[File: src/SmartWallet/MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L58)
```solidity
58:     /// @notice Thrown when trying to intialize the contracts owners if a provided owner is 32 bytes
```

intialize should be changed to `initialize`.

[File: src/SmartWallet/MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L161)
```solidity
161:     /// @param owners The intiial list of owners to register.
```

`intial` should be changed to `initial`.

[File: src/SmartWallet/MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L200)
```solidity
200:     /// @dev Revert if the sender is not an owner fo the contract itself.
```

`fo` should be changed to `of`.

[File: src/SmartWallet/ERC1271.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L9)
```solidity
9: /// @dev To prevent the same signature from being validated on different accounts owned by the samer signer,
```

`samer` should be changed to `same`.

[File: src/SmartWallet/ERC1271.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L78)
```solidity
78:     /// @notice Wrapper around `_eip712Hash()` to produce a replay-safe hash fron the given `hash`.
```

`fron` should be changed to `from`.

[File: src/SmartWallet/CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L122)
```solidity
122:     /// @notice Custom implemenentation of the ERC-4337 `validateUserOp` method. The EntryPoint will
```

`implemenentation` should be changed to `implementation`.

[File: src/SmartWallet/CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L130)
```solidity
130:     /// @dev Reverts if the signature verification fails (except for the case mentionned earlier).
```

`mentionned` should be changed to `mentioned`.

[File: src/SmartWallet/CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L173)
```solidity
173:     /// @dev Reverts if the given call is not authorized to skip the chain ID validtion.
174:     /// @dev `validateUserOp()` will recompute the `userOpHash` without the chain ID befor validatin
```

`validtion` should be changed to `validation`.
`befor validatin` should be changed to `before validating`.

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L55)
```solidity
55:     /// @notice Thrown when trying to use a withdraw request after its expiry has been reched.
```

`reched` should be changed to `reached`.

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L63)
```solidity
63:     /// @notice Thrown during validation in the context of ERC4337, when the withraw reques amount is insufficient
```

`withraw reques` should be changed to `withdraw request`.

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L76)
```solidity
76:     ///         requested amount (exluding the `maxGasCost` set by the Entrypoint).
```

`exluding` should be changed to `excluding`.

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L87)
```solidity
87:     /// @dev This should only really occur if for unknown reasons the transfer of the withdrwable
```

`withdrwable` should be changed to `withdrawable`.

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L154)
```solidity
154:         // Compute the total remaining funds available for the user accout.
155:         // NOTE: Take into account the user operation gas that was not consummed.
```

`accout` should be changed to `account`.
`consummed` should be changed to `consumed`.

# [10] Incorrect punctuation

**Files:** `MagicSpend.sol`, `CoinbaseSmartWallet.sol`, `ERC1271.sol`

According to American style guides, `i.e.` and `e.g.` should be followed by comma, while according to British style guides, there's no need to follow `i.e.` and `e.g.` by comma.
The current code-base mixes those styles in the comments, thus it's recommended to stick to one. Since there are more instances of the style without a comma:

```
./SmartWallet/CoinbaseSmartWallet.sol:    /// @notice Sends to the EntryPoint (i.e. `msg.sender`) the missing funds for this transaction.
./MagicSpend/MagicSpend.sol:    ///      funds to the user account failed (i.e. this contract's ETH balance is insufficient or
./WebAuthnSol/WebAuthn.sol:    ///         - Verifies that the client JSON is of type "webauthn.get", i.e. the client was responding to a request to
./WebAuthnSol/WebAuthn.sol:    ///           i.e. the RP does not intend to verify an attestation.
./SmartWallet/CoinbaseSmartWallet.sol:    /// @dev Subclass MAY override this modifier for better funds management (e.g. send to the
./SmartWallet/CoinbaseSmartWallet.sol:    ///      allows making a "simulation call" without a valid signature. Other failures (e.g. nonce
./SmartWallet/CoinbaseSmartWallet.sol:    ///      to be replayed for all accounts sharing the same address across chains. E.g. This may be
```

our recommendation is to implement a fix (change `e.g.,` to `e.g.` and `i.e.,` to `i.e`) in the following instances:

[File: src/MagicSpend/MagicSpend.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L168)
```solidity
168:     ///      use cases (e.g., swap or mint).
```

[File: src/SmartWallet/CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L170)
```solidity
170:     /// @notice Execute the given call from this account to this account (i.e., self call).
```

[File: src/SmartWallet/ERC1271.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L62)
```solidity
62:     ///      cross-account-replay layer on the given `hash` (i.e., verification is run on the replay-safe
```

# [11] Incorrect grammar

**Files:** `MultiOwnable.sol`, `CoinbaseSmartWallet.sol`

[File: src/SmartWallet/MultiOwnable.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L13)
```solidity
13:     ///      Some uses —-such as signature validation for secp256r1 public key owners—-
14:     ///      requires the caller to assert which owner signed. To economize calldata,
```

`Some uses (...) requires` should be changed to `Some uses (...) require`.

[File: src/SmartWallet/CoinbaseSmartWallet.sol](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L176)
```solidity
176:     ///      to be replayed for all accounts sharing the same address across chains. E.g. This may be
```

`E.g. This` should be changed to `E.g., this` (lower-cased letter).
