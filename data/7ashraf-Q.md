
# Quality Assurance Report

## Summary

| Issue Number | Issue Title                                       | Number of Instances |
|--------------|---------------------------------------------------|---------------------|
| L-01         | Check if `msg.value` is the same as the passed `amount` | 2                   |
| L-02         | Avoid hardcoded strings and addresses            | 3                   |
| L-04         | More safety is advised when removing owners      | 1                   |
| N-01         | Remove un-necessary checks                        | 1                   |
| N-02         | Consider emitting an event for the following functions | 2                   |
| N-03         | Add comments to explain assembly code             | 3                   |
| N-04         | Inaccurate variable emission                     | 1                   |



## [L-01] Check if `msg.value` is the same as the passed `amount`
### Instances
* [MagicSpend.sol#212](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L212)
```solidity
    function entryPointDeposit(uint256 amount) external payable onlyOwner {
        SafeTransferLib.safeTransferETH(entryPoint(), amount);
    }
```
* [CoinbaseSmartWallet.sol#273](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L273)
```solidity
        (bool success, bytes memory result) = target.call{value: value}(data);

```

## [L-02] Avoid hardcoded strings and addresses
Using hardcoded addresses or strings should be avoidable and may overcomplicate things in case of a contract upgrade, rather store contract addresses in a variable and add a setter to change the address in case of an upgrade
### Instances
* [MagicSpend.sol#304](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304C1-L306C6)
```solidity
    function entryPoint() public pure returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }
```
* [CoinbaseSmartWallet.sol#218](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L218)
```solidity
    function entryPoint() public view virtual returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }
```
* [CoinbaseSmartWallet.sol#334](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L334)
```solidity
        return ("Coinbase Smart Wallet", "1");

```
## [L-04] More safety is advised when removing owners
### Instances
* [MultiOwnable.sol#102](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L102C1-L110C6)
```solidity
    function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {
        bytes memory owner = ownerAtIndex(index);
        if (owner.length == 0) revert NoOwnerAtIndex(index);

        delete _getMultiOwnableStorage().isOwner[owner];
        delete _getMultiOwnableStorage().ownerAtIndex[index];

        emit RemoveOwner(index, owner);
    }

```
### Mitigation
* It is recommended to not allow the user to remove himself
* It is recommended to check if the system has at least only one owner to avoid the removal of all owners




## [N-01] Remove unnecessary checks
### Instances
* [MagicSpend.sol#150](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L150)
```solidity
        assert(mode != PostOpMode.postOpReverted);

```

## [N-02] Consider emitting an event for the following functions
### Instances
* [MagicSpend.sol#169](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L169)
```solidity
function withdrawGasExcess() external
```
* [MagicSpend.sol#203](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L203)
```solidity
function ownerWithdraw(address asset, address to, uint256 amount) external onlyOwner
```
## [N-03] Add comments to explain assembly code
### Instances
* [CoinbaseSmartWallet.sol#94](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L94C1-L99C10)
```solidity
        assembly ("memory-safe") {
            if missingAccountFunds {
                // Ignore failure (it's EntryPoint's job to verify, not the account's).
                pop(call(gas(), caller(), missingAccountFunds, codesize(), 0x00, codesize(), 0x00))
            }
        }
```
* [CoinbaseSmartWallet.sol#275](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L275C1-L278C1)
```solidity
            assembly ("memory-safe") {
                revert(add(result, 32), mload(result))
            }
```
* [CoinbaseSmartWallet.sol#309](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L309C1-L311C14)
```solidity
           assembly ("memory-safe") {
                owner := mload(add(ownerBytes, 32))
            }

```

## [N-04] Inaccurate variable emission
### Instances
* [CoinbaseSmartWallet.sol#324](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L324)
```solidity
        revert InvalidOwnerBytesLength(ownerBytes);

```
### Mitigation
Change to `ownerBytes.length`

