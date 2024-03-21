## L-01 The initialize function in the `CoinbaseSmartWallet` contract is public and does not have any access control modifiers.

This means that anyone can call it and initialize the contract, potentially overriding its state.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114-L120

```solidity
    function initialize(bytes[] calldata owners) public payable virtual {
        if (nextOwnerIndex() != 0) {
            revert Initialized();
        }

        _initializeOwners(owners);
    }
```

## L-02 `CoinbaseSmartWallet::_validateSignature` return a boolean value instead `revert`

If the length `ownerBytes.length` is not 32 and 64 it should return false instead of `revert`.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291-L325

```solidity
        revert InvalidOwnerBytesLength(ownerBytes);
```

`_validateSignature` function should return a `boolean` value instead of `revert`.

**This could potentially lead to transaction reverts instead of returning the correct validation result.**

## L-03 The `CoinbaseSmartWalletFactory::createAccount` function does not properly handle the case when the account is already deployed at the predicted address.

When the `createAccount` function is called with a set of `owners` and a `nonce`, it first checks if the account is already deployed at the predicted address by calling `LibClone.createDeterministicERC1967`. If the account is not yet deployed, it initializes the account with the given `owners`. However, if the account is already deployed at the predicted address and `different owners`, the function does not initialize the account with the given owners, which could lead to unintended consequences.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L38-L56

```solidity
        (bool alreadyDeployed, address accountAddress) =
            LibClone.createDeterministicERC1967(msg.value, implementation, _getSalt(owners, nonce));

        account = CoinbaseSmartWallet(payable(accountAddress));

        if (alreadyDeployed == false) {
            account.initialize(owners);
        }
    }
```

If the account has different owners, the function will revert with an error.

## L-04. `MultiOwnable::removeOwnerAtIndex` function revert in certain cases

The `removeOwnerAtIndex` function is designed to remove an owner from the given `index`. However, it does not check if the owner being removed is the last owner of the contract. If the last owner is removed, the `nextOwnerIndex` will not be updated, which could lead to unintended consequences.

After the last owner (B) is removed, the `nextOwnerIndex` remains the same (1), which could cause issues when adding new owners.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L102-L110

```diff
+    // If the removed owner was the last owner, update the nextOwnerIndex
+    if (index == _getMultiOwnableStorage().nextOwnerIndex - 1) {
+        _getMultiOwnableStorage().nextOwnerIndex--;
+    }
}
```

With this update, the `removeOwnerAtIndex` function will now properly handle the case when the last owner is removed by updating the `nextOwnerIndex`. This should prevent the contract from reverting in this scenario.

## L-05 Lack of `event` and `emit`

Some functions do not `emit events` when they modify the `contract's state`.

Emitting events is important for tracking changes in the contract's state and can help with off-chain data indexing and monitoring.

The `entryPointDeposit`, `entryPointWithdraw`, `entryPointAddStake`, `entryPointUnlockStake`, and `entryPointWithdrawStake` functions in the `MagicSpend` contract do not emit events when they modify the contract's state.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L212

```solidity
    function entryPointDeposit(uint256 amount) external payable onlyOwner {
        SafeTransferLib.safeTransferETH(entryPoint(), amount);
    }


        function entryPointWithdraw(address payable to, uint256 amount) external onlyOwner {
        IEntryPoint(entryPoint()).withdrawTo(to, amount);
    }

        function entryPointAddStake(uint256 amount, uint32 unstakeDelaySeconds) external payable onlyOwner {
        IEntryPoint(entryPoint()).addStake{value: amount}(unstakeDelaySeconds);
    }

        function entryPointUnlockStake() external onlyOwner {
        IEntryPoint(entryPoint()).unlockStake();
    }

        function entryPointWithdrawStake(address payable to) external onlyOwner {
        IEntryPoint(entryPoint()).withdrawStake(to);
    }
```

## L-06 `CoinbaseSmartWalletFactory::getAddress` function return incorrect `address` when the `owners` array is empty

The `getAddress` function is designed to return the predicted account deployment address for a given set of `owners` and a `nonce`. However, it does not properly handle the case when the `owners` array is empty. In this case, the function will return an incorrect address, which could lead to unintended consequences.

1. Call the `getAddress` function with an empty `owners` array and a nonce.

The `getAddress` function will return an incorrect address, which could cause the whole protocol to fail if this address is used for account deployment or other purposes.

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L64-L66

```solidity
    function getAddress(bytes[] calldata owners, uint256 nonce) external view returns (address predicted) {
        predicted = LibClone.predictDeterministicAddress(initCodeHash(), _getSalt(owners, nonce), address(this));
    }
```

To fix this issue, you can update the `getAddress` function as follows:

```diff

+    require(owners.length > 0, "Owners array cannot be empty");

    predicted = LibClone.predictDeterministicAddress(initCodeHash(), _getSalt(owners, nonce), address(this));
```

With this update, the `getAddress` function will now properly handle the case when the `owners` array is empty by reverting the transaction with an error message. This should prevent the contract from returning an incorrect address.

## L-07. `CoinbaseSmartWallet::executeBatch` function not handle when the batch of calls contains a call to the `executeBatch` function itself

The `executeBatch` function allows an owner or the EntryPoint to execute a batch of calls from the smart wallet contract to other contracts. However, it does not properly handle the case when the batch of calls contains a call to the `executeBatch` function itself. This could lead to an infinite loop and cause the transaction to revert due to the gas limit.

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

1. Create a batch of calls that includes a call to the `executeBatch` function itself.
2. Call the `executeBatch` function in the `CoinbaseSmartWallet` contract with the batch of calls.

The transaction will revert due to the `gas limit`, as the `executeBatch` function will keep calling itself in an infinite loop.
