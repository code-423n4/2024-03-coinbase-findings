## Funds will be stuck in MagicSpend if withdrawRequest is expired 

## Links

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L181-L194

## Vulnerability details

In order to withdraw funds an EOA or SCW will need to validate a signature in order to finalize the withdrawal. However if there is downtime on the blockchain side which might occur, the signature might become expired and unable user to withdraw.

## Impact

User will be denied withdrawal

## POC

In the [withdraw()](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L184-L190) function we can see the logic below.

```solidity
        if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
            revert InvalidSignature();
        }

        if (block.timestamp > withdrawRequest.expiry) {
            revert Expired();
        }
```
If there is any downtime on the blockchain or even block stuffing depending on the expiry, the withdrawal process will revert. 

## Mitigation Route

Remove this check: 
```
if (block.timestamp > withdrawRequest.expiry) {
            revert Expired();
```

## Typos

## Links

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L174

## Typo:

```solidity
ID befor validatin
```
