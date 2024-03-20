### Report 1:
#### Invalid Verification due to Absence of Sensitive Verification Steps
The Code provided below shows how verify(...) function is implemented in the WebAuthn contract, The problem is that Steps 1-10 , 13 - 15 & 18 were all skipped on purpose, the fact that the protocol is aware does not justify such implementation exceptions, necessary corrections should be made by Protocol to avoid invalid Verifications
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L130-L142
```solidity
 function verify(bytes memory challenge, bool requireUV, WebAuthnAuth memory webAuthnAuth, uint256 x, uint256 y)
        internal
        view
        returns (bool)
    {
        if (webAuthnAuth.s > P256_N_DIV_2) {
            // guard against signature malleability
            return false;
        }
>>>
        // 11. Verify that the value of C.type is the string webauthn.get.
        // bytes("type":"webauthn.get").length = 21
        string memory _type = webAuthnAuth.clientDataJSON.slice(webAuthnAuth.typeIndex, webAuthnAuth.typeIndex + 21);
        if (keccak256(bytes(_type)) != EXPECTED_TYPE_HASH) {
            return false;
        }

        // 12. Verify that the value of C.challenge equals the base64url encoding of options.challenge.
        bytes memory expectedChallenge = bytes(string.concat('"challenge":"', Base64.encodeURL(challenge), '"'));
        string memory actualChallenge = webAuthnAuth.clientDataJSON.slice(
            webAuthnAuth.challengeIndex, webAuthnAuth.challengeIndex + expectedChallenge.length
        );
        if (keccak256(bytes(actualChallenge)) != keccak256(expectedChallenge)) {
            return false;
        }

>>>        // Skip 13., 14., 15.

        // 16. Verify that the UP bit of the flags in authData is set.
        if (webAuthnAuth.authenticatorData[32] & AUTH_DATA_FLAGS_UP != AUTH_DATA_FLAGS_UP) {
            return false;
        }

        // 17. If user verification is required for this assertion, verify that the User Verified bit of the flags in authData is set.
        if (requireUV && (webAuthnAuth.authenticatorData[32] & AUTH_DATA_FLAGS_UV) != AUTH_DATA_FLAGS_UV) {
            return false;
        }

>>>        // skip 18.

        // 19. Let hash be the result of computing a hash over the cData using SHA-256.
        bytes32 clientDataJSONHash = sha256(bytes(webAuthnAuth.clientDataJSON));

        ...

        return FCL.ecdsa_verify(messageHash, webAuthnAuth.r, webAuthnAuth.s, x, y);
    }
```
###  Report 2:
#### Withdrawal Would not Revert at Expiry
The code provided shows how withdraw(...) function is implemented in the MagicSpend contract, The problem is that due to an oversight in implementation when Block.timestamp is exactly at expiry point the function would not revert when it should, reversion should take effect at the expiry point not after it, necessary adjustment should be done as provided below.
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L188
```solidity
  function withdraw(WithdrawRequest memory withdrawRequest) external {
        _validateRequest(msg.sender, withdrawRequest);

        if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
            revert InvalidSignature();
        }

---        if (block.timestamp > withdrawRequest.expiry) {
+++        if (block.timestamp >= withdrawRequest.expiry) {
            revert Expired();
        }

        // reserve funds for gas, will credit user with difference in post op
        _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
    }

```
###  Report 3:
#### post Operation can take effect at Zero Gas Cost
The function provided below shows how post operation is handled in the MagicSpend function, the problem is that no implementation was done to prevent actualGasCost from being zero meaning the function would go through with zero cost which could be taken advantage of to avoid gas payment in MagicSpend.sol contract
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L143-L156
```solidity
>>>  function postOp(IPaymaster.PostOpMode mode, bytes calldata context, uint256 actualGasCost)
        external
        onlyEntryPoint
    {
        // `PostOpMode.postOpReverted` should be impossible.
        // Only possible cause would be if this contract does not own enough ETH to transfer
        // but this is checked at the validation step.
        assert(mode != PostOpMode.postOpReverted);

        (uint256 maxGasCost, address account) = abi.decode(context, (uint256, address));

        // Compute the total remaining funds available for the user accout.
        // NOTE: Take into account the user operation gas that was not consummed.
>>>        uint256 withdrawable = _withdrawableETH[account] + (maxGasCost - actualGasCost);

        // Send the all remaining funds to the user accout.
        delete _withdrawableETH[account];
        if (withdrawable > 0) {
            SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);
        }
    }
```
###  Report 4:
#### Incomplete Function Selector Validation
The code provided below from the CoinbaseSmartWallet contract shows one of the ways execution is handled in the contract, the problem is that the function wrongly assumes data length would always be equal to or greater than 4, in a situation it is less than 4, a wrong function selector would be used in the function implementation which would break the contract functionality, necessary validation should be done as provided below and Protocol should make necessary implementation for excess data length
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L181
```solidity
   function executeWithoutChainIdValidation(bytes calldata data) public payable virtual onlyEntryPoint {
+++       require ( data.length >= 4 , "Error Message" )
>>>        bytes4 selector = bytes4(data[0:4]);
        if (!canSkipChainIdValidation(selector)) {
            revert SelectorNotAllowed(selector);
        }

        _call(address(this), 0, data);
    }
```
###  Report 5:
#### Unimplemented Function
Unimplemented Function which would return empty string name and string version in the ERC1271.sol, This report would not have been a problem since ERC1271 is an abstract contract but the problem is from the fact that validation was not done at [L51](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L51) & [L101](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L101) where the function is used to ensure the name and version strings are not actually empty
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L143
```solidity
 /// @notice Returns the domain name and version to use when creating EIP-712 signatures.
    ///
    /// @dev MUST be defined by the implementation.
    ///
    /// @return name The user readable name of signing domain.
    /// @return version The current major version of the signing domain.
    function _domainNameAndVersion() internal view virtual returns (string memory name, string memory version);
```

