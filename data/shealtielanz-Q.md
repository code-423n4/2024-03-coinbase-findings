# L-1: MultiOwnable.sol is missing two functions as specified in the documentation,  making the `canSkipChainIdValidation()` checks to not be complete.
In the documentation, there are 6 functions that allow for cross-chain replayability.
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/README.md

![Screenshot from 2024-03-21 19-40-02](https://github.com/shealtielanz/chainlinktQA/assets/126171088/969630c3-9136-4006-8b81-d5719a571eab)

However, only four of these functions were implemented in the MultiOwnable.sol
Implemented and checked in the  `canSkipChainIdValidation()`:
https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L252C1-L262C6
```solidity
    function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
        if (
            functionSelector == MultiOwnable.addOwnerPublicKey.selector
                || functionSelector == MultiOwnable.addOwnerAddress.selector
                || functionSelector == MultiOwnable.removeOwnerAtIndex.selector
                || functionSelector == UUPSUpgradeable.upgradeToAndCall.selector
        ) {
            return true;
        }
        return false;
    }
```

## Mitigation

Implement the missing functions and add them to the `canSkipChainIdValidation()`

# L-2:  Make use of `non-reentrant` guards on functions sending eth or other tokens. 
In MagicSpend.sol & CoinBaseSmartContract.sol There are different functions that send ETh and other tokens out of the contract, however this function can be prone to re-entrancy it is best to add non-reentrant modifier to the following functions.
- withdrawGasExcess() -> https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L169
- withdraw() -> https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L181
- ownerWithdraw() -> https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L203
- executeBatch() -> https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L205
- execute() -> https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L196
- executeWithoutChainIdValidation() -> https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L180C14-L180C46

# Mitigation
Add a non-reentrant modifier.

# L-3: Griefing as an attacker can front-run the call to create the SCW Account leading to bad user UX.
Anyone can call createAccount() in the Factory, this allows malicious actors to front run the call to the contract since it uses create2 to create the contract before the User Operation and create it before the User does thereby grieving the users, sometimes the address might be hard to find If it has already been created, this can lead to bad user UX and annoyance.
https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L48C1-L49C101
```solidity
   function createAccount(bytes[] calldata owners, uint256 nonce)
        public
        payable
        virtual
        returns (CoinbaseSmartWallet account)
    {
        if (owners.length == 0) {
            revert OwnerRequired();
        }

        (bool alreadyDeployed, address accountAddress) = //@audit frontrun call to creat2 
            LibClone.createDeterministicERC1967(msg.value, implementation, _getSalt(owners, nonce));//@audit erc 4337 specifies the use of create2 over create also use signature during creation.

        account = CoinbaseSmartWallet(payable(accountAddress));

        if (alreadyDeployed == false) {
            account.initialize(owners); //@audit-ok this should revert.
        }
    }
```

# Mitigation 
- Allow only the Entry Point to call the function.
- Use a unique key by the caller during the creation of the contract or a hash to avoid this.
# L-4: Solady's `isValidSignatureNow()` is prone to signature malleability.
The contracts use `isValidSignatureNow()` by Solady to check for signature validity allowing for signature malleability as the function doesn't check `s` against the other side of the curve.
https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L313
```solidity
            return SignatureCheckerLib.isValidSignatureNow(owner, message, sigWrapper.signatureData);
```
https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L265
```solidity
        return SignatureCheckerLib.isValidSignatureNow(
            owner(), getHash(account, withdrawRequest), withdrawRequest.signature
        );

```
This allows for validation of invalid signature as attackers can manipulated the signature to grieve the signer.

## Mitigation
Use Openzeppelin library to avoid this

# L-5: Newly created accounts should depend on the user's signature.

# L-6: Anyone can call `upgradeToAndCall()`
The `upgradeToAndCall()` in CoinBaseSmartContract.sol can be called by anyone, allowing malicious actors to be able to perform malicious activities like upgrades and grieving the users:

```solidity
 /// @dev Upgrades the proxy's implementation to `newImplementation`.
    /// Emits a {Upgraded} event.
    ///
    /// Note: Passing in empty `data` skips the delegatecall to `newImplementation`.
    function upgradeToAndCall(address newImplementation, bytes calldata data)
        public
        payable
        virtual
        onlyProxy
    {
..SNIP..
}
```
## Mitigation
Add a modifier to ensure only the owners can call this function.

# L-7: The SCW account check the user ops nonce.
As per vitalik Buterin Article [here](https://medium.com/infinitism/erc-4337-account-abstraction-without-ethereum-protocol-changes-d75c9d94dc4a) :
> validateUserOp, which takes a UserOperation as input. This function is supposed to verify the signature and nonce on the UserOperation, pay the fee and increment the nonce if verification succeeds, and throw an exception if verification fails.

The `validateUserOp()` doesn't check against the user nonce.
https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L137C1-L168C6
```solidity
    function validateUserOp(UserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)
        public
        payable
        virtual
        onlyEntryPoint
        payPrefund(missingAccountFunds)
        returns (uint256 validationData)
    {
        uint256 key = userOp.nonce >> 64;

        if (userOp.callData.length >= 4 && bytes4(userOp.callData[0:4]) == 0xbf6ba1fc) {
            userOpHash = getUserOpHashWithoutChainId(userOp);
            if (key != REPLAYABLE_NONCE_KEY) {
                revert InvalidNonceKey(key);
            }
        } else {
            if (key == REPLAYABLE_NONCE_KEY) {
                revert InvalidNonceKey(key);
            }
        }

.
        if (_validateSignature(userOpHash, userOp.signature)) {
            return 0;
        }

        return 1;
    }
```

# Mitigation 
Validate the user nonce.

# Info-1: solady `safeTransfer()` doesn't check contracts existence.
Solady `safeTransfer()` doesn't check against the existence of the contract in an address and can be harmful to the contract when integrating with other contracts
https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L338
```solidity
         SafeTransferLib.safeTransfer(asset, to, amount); //@audit 
        }
```