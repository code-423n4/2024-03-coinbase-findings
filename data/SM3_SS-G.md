
# Gas optimization 

| Number | Issue | Instences |
|--------|-------|------------|
|[G-01]| Use assembly to emit events | 3 |
|[G-02]| Pre-calculate keccak256 for constant variables | 2 |
|[G-03]| Remove local-testing related code in MagicSpend.sol | 1 |
|[G-04]| Optimize address(0) Checks Using Assembly | 4 |
|[G-05]| Check Arguments Early | 4 |
|[G-06]| Use named returns for local variables of pure functions where it is possible | 11 |
|[G-07]| Shorten arrays with inline assembly | 1 |
|[G-08]| Redundant Bool Comparison | 1 |
|[G-09]| Using bytes32 is cheaper than using string. | 2 |
|[G-10]| Use hardcoded address instead of address(this) | 8 |


## [G-01] Use assembly to emit events

This detector checks for instances where a contract uses assembly code to emit events. While it’s technically possible to emit events using inline assembly in Solidity, it’s generally discouraged due to readability concerns and potential for errors. Events should usually be emitted using higher-level Solidity constructs.

```solidity
file: blob/main/src/MagicSpend/MagicSpend.sol

323   emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L323


```solidity 
file: blob/main/src/SmartWallet/MultiOwnable.sol

109  emit RemoveOwner(index, owner);

195  emit AddOwner(index, owner);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L109

## [G-02] Pre-calculate keccak256 for constant variables

```solidity
file: blob/main/src/SmartWallet/ERC1271.sol

23   bytes32 private constant _MESSAGE_TYPEHASH = keccak256("CoinbaseSmartWalletMessage(bytes32 hash)");

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L23

```solidity
file: blob/main/src/WebAuthnSol/WebAuthn.sol

55  bytes32 private constant EXPECTED_TYPE_HASH = keccak256('"type":"webauthn.get"');

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L55

## [G-03] Remove local-testing related code in MagicSpend.sol

```solidity
file: blob/main/src/MagicSpend/MagicSpend.sol

147  // `PostOpMode.postOpReverted` should be impossible.
        // Only possible cause would be if this contract does not own enough ETH to transfer
        // but this is checked at the validation step.
        assert(mode != PostOpMode.postOpReverted);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L147-L151

## [G-04] Optimize address(0) Checks Using Assembly

Solidity provides address(0) as a constant for the zero address. While it is generally safe to use, explicit checks against address(0) in your code might be slightly more gas efficient if implemented in inline assembly, due to reduced overhead.

```solidity
file: blob/main/src/MagicSpend/MagicSpend.sol

121   if (withdrawRequest.asset != address(0)) {

175   _withdraw(address(0), msg.sender, amount);

335   if (asset == address(0)) {

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L121

```solidity
file: blob/main/src/SmartWallet/CoinbaseSmartWallet.sol

105 owners[0] = abi.encode(address(0));

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L105

## [G-05] Check Arguments Early

Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.




```solidity
file:  blob/main/src/MagicSpend/MagicSpend.sol

188   if (block.timestamp > withdrawRequest.expiry) {
            revert Expired();
        }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L188-L190

```solidity
file: blob/main/src/WebAuthnSol/WebAuthn.sol

138  if (requireUV && (webAuthnAuth.authenticatorData[32] & AUTH_DATA_FLAGS_UV) != AUTH_DATA_FLAGS_UV) {
            return false;
        }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L138-L140

```solidity
file: blob/main/src/MagicSpend/MagicSpend.sol

117  if (withdrawAmount < maxCost) {
            revert RequestLessThanGasMaxCost(withdrawAmount, maxCost);
        }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L117-L119

```solidity
file: blob/main/src/WebAuthnSol/WebAuthn.sol

133 if (webAuthnAuth.authenticatorData[32] & AUTH_DATA_FLAGS_UP != AUTH_DATA_FLAGS_UP) {
            return false;
        }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L133-L135

## [G-06] Use named returns for local variables of pure functions where it is possible


```solidity
file: blob/main/src/SmartWallet/ERC1271.sol

133   function _hashStruct(bytes32 hash) internal view virtual returns (bytes32) {
        return keccak256(abi.encode(_MESSAGE_TYPEHASH, hash));
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L133-L135

```solidity
file:  blob/main/src/MagicSpend/MagicSpend.sol

304    function entryPoint() public pure returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304-L306

```solidity
file: blob/main/src/SmartWallet/MultiOwnable.sol

117  function isOwnerAddress(address account) public view virtual returns (bool) {
        return _getMultiOwnableStorage().isOwner[abi.encode(account)];
    }

127  function isOwnerPublicKey(bytes32 x, bytes32 y) public view virtual returns (bool) {
        return _getMultiOwnableStorage().isOwner[abi.encode(x, y)];
    }

136  function isOwnerBytes(bytes memory account) public view virtual returns (bool) {
        return _getMultiOwnableStorage().isOwner[account];
    }

145  function ownerAtIndex(uint256 index) public view virtual returns (bytes memory) {
        return _getMultiOwnableStorage().ownerAtIndex[index];
    }

152  function nextOwnerIndex() public view virtual returns (uint256) {
        return _getMultiOwnableStorage().nextOwnerIndex;
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L117-L120

```solidity
file: blob/main/src/SmartWallet/CoinbaseSmartWallet.sol

217  function entryPoint() public view virtual returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }

229  function getUserOpHashWithoutChainId(UserOperation calldata userOp)
        public
        view
        virtual
        returns (bytes32 userOpHash)
    {
        return keccak256(abi.encode(UserOperationLib.hash(userOp), entryPoint()));
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L217-L219

```solidity
file: blob/main/src/SmartWallet/CoinbaseSmartWallet.sol

333  function _domainNameAndVersion() internal pure override(ERC1271) returns (string memory, string memory) {
        return ("Coinbase Smart Wallet", "1");
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L333-L335

```solidity
file: blob/main/src/MagicSpend/MagicSpend.sol

299  function nonceUsed(address account, uint256 nonce) external view returns (bool) {
        return _nonceUsed[nonce][account];
    }

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L299-L301

## [G-07] Shorten arrays with inline assembly

Issue Description - When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating storage.

Proposed Optimization - Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new array.

Estimated Gas Savings - Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending on original length.

```solidity
file: blob/main/src/SmartWallet/CoinbaseSmartWallet.

104   bytes[] memory owners = new bytes[](1);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104

## [G-08] Redundant Bool Comparison

This detector checks for instances where a boolean expression is compared to a boolean literal (true or false), which is redundant.

```solidity
file: blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol

53  if (alreadyDeployed == false) {

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L53

## [G-09] Using bytes32 is cheaper than using string.

Storing and manipulating data in bytes32 is more gas-efficient than using string, which involves dynamic storage allocation. bytes32 is a fixed-size type, whereas string can have variable length, resulting in higher gas costs.


```solidity
file: blob/main/src/WebAuthnSol/WebAuthn.sol

116  string memory _type = webAuthnAuth.clientDataJSON.slice(webAuthnAuth.typeIndex, webAuthnAuth.typeIndex + 21);

123  string memory actualChallenge = webAuthnAuth.clientDataJSON.slice(

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L116

## [G-10] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences


```solidity
file: blob/main/src/MagicSpend/MagicSpend.sol

133  if (address(this).balance < withdrawAmount) {

134  revert InsufficientBalance(withdrawAmount, address(this).balance);

282  address(this),

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L133

```solidity
file: blob/main/src/SmartWallet/CoinbaseSmartWallet.sol

186  _call(address(this), 0, data);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L186


```solidity
file: blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol

73   address(this)

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L73


```solidity
file: blob/main/src/SmartWallet/ERC1271.sol

53   verifyingContract = address(this);

108  address(this)

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L53

```solidity
file:  blob/main/src/SmartWallet/MultiOwnable.sol

202    if (isOwnerAddress(msg.sender) || (msg.sender == address(this))) {

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L202