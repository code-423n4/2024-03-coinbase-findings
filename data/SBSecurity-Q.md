### Low Risk

| Count | Title |
| --- | --- |
| [L-01] | Upgrade to EntryPoint 0.7 |
| [L-02] | _initializeOwners is not capped at reasonable bounds  |
| [L-03] | executeBatch is not capped at reasonable bounds  |

| Total Low Risk Issues | 3 |
| --- | --- |

### Non-Critical

| Count | Title |
| --- | --- |
| [NC-01] | Pass only nonce to _validateRequest |
| [NC-02] | MultiOwnable contract need to be marked abstract |
| [NC-03] | Wallets with same owners and same nonce can be deployed |
| [NC-04] | Do not explicitly compare bool variables to bool expression |
| [NC-05] | Typos |

| Total Non-Critical Issues | 5 |
| --- | --- |

## Low Risks

## [L-01] Upgrade to EntryPoint 0.7

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L217-L219

**Issue Description:** CoinbaseSmartWallet uses EntryPoint 0.6, although EntryPoint 0.7 was released in the past months. EntryPoint 0.7 introduce more gas optimization, security and other interesting enhancements. 

Can check here: https://blog.smart-contracts-developer.com/entrypoint-v0-7-0-the-new-era-of-account-abstraction-854f9f82912e

**Recommendation:** Upgrade the CoinbaseSmartWallet to use EntryPoint 0.7

## [L-02] `_initializeOwners` is not capped at reasonable bounds

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L162-L174

**Issue Description:** `_initializeOwners` is not capped and if the array is big can result in out of gas.

```solidity
function _initializeOwners(bytes[] memory owners) internal virtual { 
    for (uint256 i; i < owners.length; i++) {
        if (owners[i].length != 32 && owners[i].length != 64) {
            revert InvalidOwnerBytesLength(owners[i]);
        }

        if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
            revert InvalidEthereumAddressOwner(owners[i]);
        }

        console.log(_getMultiOwnableStorage().nextOwnerIndex);
        _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
    }
}
```

**Recommendation:** Cap the function at reasonable bounds, since then can add owners and always can add if miss someone at initialization.

## [L-03] `executeBatch` is not capped at reasonable bounds

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L205-L212

**Issue Description:** `executeBatch` is not capped and if the array of `Calls` is big can result in out of gas.

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

**Recommendation:** Cap the function at reasonable bounds.

## Non-Critical

## **[N‑01]** Pass only nonce to `_validateRequest`

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L315-L324

**Issue Description:** In `MagicSpend::_validateRequest` only the nonce from `WithdrawRequest` was used.

```solidity
function _validateRequest(address account, WithdrawRequest memory withdrawRequest) internal {
    if (_nonceUsed[withdrawRequest.nonce][account]) {
        revert InvalidNonce(withdrawRequest.nonce);
    }

    _nonceUsed[withdrawRequest.nonce][account] = true;

    // This is emitted ahead of fund transfer, but allows a consolidated code path
    emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce);
}
```

**Recommendation:** In `validatePaymasterUserOp()` and `withdraw()` pass only nonce to `_validateRequest` and move the event in the upper function.

## **[N‑02] MultiOwnable contract need to be marked abstract**

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L32

**Issue Description:** `MultiOwnable` contract is inherited from `CoinbaseSmartWallet` and all its functions are used there as inherited. No one will use `MultiOwnable` independently.

```solidity
contract MultiOwnable {
```

**Recommendation:** Mark `MultiOwnable` abstract.

## **[N‑03] Wallets with same owners and same nonce can be deployed**

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L81-L83

**Issue Description:** When deploy wallets, owners and nonce is passed. Nonce is used to allow multiple accounts with same owners to be deployed, because its used to generate the salt.

```solidity
function createAccount(bytes[] calldata owners, uint256 nonce)
```

But if reorder the owners, another wallet can be deployed with same nonce, since when generate the salt, owners and nonce is used.

```solidity
function _getSalt(bytes[] calldata owners, uint256 nonce) internal pure returns (bytes32 salt) {
    salt = keccak256(abi.encode(owners, nonce));
}
```

**Recommendation:** Change the way salt is generated or limit to pass the nonce from the deployer and user auto-incrementing nonce.

## **[N‑04] Do not explicitly compare bool variables to bool expression**

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L53

**Issue Description:** In `createAccount` there is check if the account is already deployed and if so, it doesn’t deploy it, but just return the address. The if statement explicitly compare the `alreadyDeployed` with `false`.

```solidity
if (alreadyDeployed == false) { // @audit - !alreadyDeployed
    account.initialize(owners);
}
```

**Recommendation:** Remove `== false`, and change to `if (!alreadyDeployed)`.

## **[N‑05] Typos**

**Issue Description:** There are several typos in the codebase:

MultiOwner.sol

```solidity
bytes32 private constant MUTLI_OWNABLE_STORAGE_LOCATION =
        0x97e2c6aad4ce5d562ebfaa00db6b9e0fb66ea5d8162ed5b243f51a2e03086f00; // @audit - MUTLI -> MULTI
```

```solidity
/// @notice Checks if the sender is an owner of this contract or the contract itself.
///
/// @dev Revert if the sender is not an owner fo the contract itself. // @audit - typo | fo -> of
function _checkOwner() internal view virtual {
    if (isOwnerAddress(msg.sender) || (msg.sender == address(this))) {
        return;
    }

    revert Unauthorized();
}
```

[36](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L36), [200](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L200)

ERC1271.sol

```solidity
/// @dev To prevent the same signature from being validated on different accounts owned by the samer signer, // @audit - samer -> same
```

[9](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/ERC1271.sol#L9)