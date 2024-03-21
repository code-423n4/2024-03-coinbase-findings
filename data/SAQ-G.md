## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Internal functions only called once can be inlined to save gas | 4 | - |
| [G-02] | Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas | 1 | - |
| [G-03] | Using fixed bytes is cheaper than using string | 1 | - |
| [G-04] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 1 | - |
| [G-05] | Before transfer of  some functions, we should check some variables for possible gas save | 1 | - |
| [G-06] | Don't compare boolean expressions to boolean literals | 1 | - |
| [G-07] | Using storage instead of memory for structs/arrays saves gas | 1 | - |
| [G-08] | `require()` Should Be Used Instead Of `assert()` | 1 | - |
| [G-09] | Use hardcode address instead address(this) | 3 | - |
| [G-10] | Use assembly in place of abi.decode to extract calldata values more efficiently | 5 | - |


## Gas Optimizations  

## [G-01] Internal functions only called once can be inlined to save gas



```solidity
file: /src/FreshCryptoLib/FCL.sol

78     function ecAff_isOnCurve(uint256 x, uint256 y) internal pure returns (bool) {

94    function FCL_nModInv(uint256 u) internal view returns (uint256 result) {

117    function ecZZ_mulmuladd_S_asm(
        uint256 Q0,
        uint256 Q1, //affine rep for input point Q
        uint256 scalar_u,
        uint256 scalar_v
    ) internal view returns (uint256 X) {  

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L78


```solidity
file: /src/SmartWallet/ERC1271.sol

121    function _eip712Hash(bytes32 hash) internal view virtual returns (bytes32) {

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L121


## [G-02]  Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas  


```solidity
file: /src/SmartWallet/CoinbaseSmartWallet.sol

43    uint256 public constant REPLAYABLE_NONCE_KEY = 8453;

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L43


## [G-03] Using fixed bytes is cheaper than using string


```solidity
file: /src/SmartWallet/ERC1271.sol

43            string memory version,

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L43


## [G-04] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant


While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor. 


```solidity
file: /src/WebAuthnSol/WebAuthn.sol

55    bytes32 private constant EXPECTED_TYPE_HASH = keccak256('"type":"webauthn.get"');

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L55


## [G-05] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything


```solidity
file: /src/MagicSpend/MagicSpend.sol

213        SafeTransferLib.safeTransferETH(entryPoint(), amount);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L213


## [G-06] Don't compare boolean expressions to boolean literals


`if (<x> == true)` => `if (<x>)`, `if (<x> == false)` => `if (!<x>)`

```solidity
file: /src/SmartWallet/CoinbaseSmartWalletFactory.sol

53        if (alreadyDeployed == false) {

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L53C1-L53C40


## [G-07] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /src/SmartWallet/CoinbaseSmartWallet.sol

104        bytes[] memory owners = new bytes[](1);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104C1-L104C48


## [G-08] `require()` Should Be Used Instead Of `assert()`
`
```solidity
file:/src/MagicSpend/MagicSpend.sol

150        assert(mode != PostOpMode.postOpReverted);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L150C1-L150C51


## [G-09] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address


```solidity
file: /src/SmartWallet/CoinbaseSmartWalletFactory.sol

65        predicted = LibClone.predictDeterministicAddress(initCodeHash(), _getSalt(owners, nonce), address(this));

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L65C1-L65C114


```solidity
file: /src/SmartWallet/ERC1271.sol

53        verifyingContract = address(this);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L53


```solidity
file: /src/SmartWallet/CoinbaseSmartWallet.sol

186        _call(address(this), 0, data);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L186C1-L186C39


## [G-10] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity
file: /src/SmartWallet/CoinbaseSmartWallet.sol

298        SignatureWrapper memory sigWrapper = abi.decode(signature, (SignatureWrapper));

317            (uint256 x, uint256 y) = abi.decode(ownerBytes, (uint256, uint256));

319            WebAuthn.WebAuthnAuth memory auth = abi.decode(sigWrapper.signatureData, (WebAuthn.WebAuthnAuth));

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L298C1-L298C88


```solidity
file: /src/MagicSpend/MagicSpend.sol

114        WithdrawRequest memory withdrawRequest = abi.decode(userOp.paymasterAndData[20:], (WithdrawRequest));

152        (uint256 maxGasCost, address account) = abi.decode(context, (uint256, address));

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L114C1-L114C110
