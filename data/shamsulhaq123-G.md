## Gas Optimization

## Summary

|No|Issue|Instance|
|--|-----|--------|
|[G-01]|Use assembly Use assembly for loops|2|
|[G-02]|Unnecessary look up in if condition|4|
|[G-03]|Use assembly to check for address(0) |4|
|[G-04]|Use Custom Errors instead of Revert Strings to save Gas|10|
|[G-05]|Consider using OZ EnumerateSet in place of nested mappings|1|
|[G-06]|Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate|2|
|[G-07]|Emitting constants wastes gas|3|
|[G-08]|Superfluous event fields|1|
|[G-09]|Using mappings instead of arrays to avoid length checks save gas|10|
|[G-10]|address(this) can be stored in state variable that will ultimately cost less|6|
|[G-11]|Call msg.sender directly instead of caching them|6|
|[G-12]|Stack variable cost less while used in emiting event|3|
|[G-13]|Use function instead of modifiers |2|
|[G-14]|Remove or replace unused state variables|11|
|[G-15]|Should use arguments instead of state variable |3|
|[G-16]|Stack variable cost less while used in emiting event|3|
|[G-17]|Using XOR (^) and AND (&) bitwise equivalents for gas optimizations|
|[G-18]|
|[G-19]|
|[G-20]|

## [G-01] Use assembly Use assembly for loops
In the following instances, assembly is used for more gas efficient loops.

```solidity

file: SmartWallet/MultiOwnable.sol

163  for (uint256 i; i < owners.length; i++)
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L163

```solidity

file: SmartWallet/CoinbaseSmartWallet.so


206   for (uint256 i; i < calls.length;) 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L206

## [G-02] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: SmartWallet/MultiOwnable.sol

202    if (isOwnerAddress(msg.sender) || (msg.sender == address(this)))
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L202

```solidity

file: SmartWallet/CoinbaseSmartWallet.sol

253   if (
            functionSelector == MultiOwnable.addOwnerPublicKey.selector
                || functionSelector == MultiOwnable.addOwnerAddress.selector
                || functionSelector == MultiOwnable.removeOwnerAtIndex.selector
                || functionSelector == UUPSUpgradeable.upgradeToAndCall.selector
        )


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L253

```solidity

file: FreshCryptoLib/FCL.sol


51  if (r == 0 || r >= n || s == 0 || s >= n)

79    if (((0 == x) && (0 == y)) || x == p || y == p)
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L51
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L79

## [G-03] Use assembly to check for address(0) 

```solidity

file: SmartWallet/CoinbaseSmartWallet.sol

105  owners[0] = abi.encode(address(0));
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L105

```solidity

file: MagicSpend/MagicSpend.sol

121 if (withdrawRequest.asset != address(0)) 

175   _withdraw(address(0), msg.sender, amount);

335  if (asset == address(0)) 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L121
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L175
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L335

## [G-04] Use Custom Errors instead of Revert Strings to save Gas

```solidity

file: SmartWallet/MultiOwnable.sol

213  assembly ("memory-safe")
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L213

```solidity

file: SmartWallet/ERC1271.sol

23  bytes32 private constant _MESSAGE_TYPEHASH = keccak256("CoinbaseSmartWalletMessage(bytes32 hash)");

104   keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L23
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L104

```solidity

file: SmartWallet/CoinbaseSmartWalletFactory.sol

4 import {LibClone} from "solady/utils/LibClone.sol";
import {CoinbaseSmartWallet} from "./CoinbaseSmartWallet.sol";

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L4

```solidity

file: SmartWallet/CoinbaseSmartWallet.sol

4 import {Receiver} from "solady/accounts/Receiver.sol";
import {UUPSUpgradeable} from "solady/utils/UUPSUpgradeable.sol";
import {SignatureCheckerLib} from "solady/utils/SignatureCheckerLib.sol";
import {UserOperation, UserOperationLib} from "account-abstraction/interfaces/UserOperation.sol";
import {WebAuthn} from "../WebAuthnSol/WebAuthn.sol";

import {ERC1271} from "./ERC1271.sol";
11 import {MultiOwnable} from "./MultiOwnable.sol";

94 assembly ("memory-safe") 

275    assembly ("memory-safe") 

309 assembly ("memory-safe")
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L4-L11
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L94
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L275
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L309

```solidity

file: WebAuthnSol/WebAuthn.sol

4 import {FCL} from "../FreshCryptoLib/FCL.sol";
import {Base64} from "openzeppelin-contracts/contracts/utils/Base64.sol";
6 import {LibString} from "solady/utils/LibString.sol";


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L4-L6

```solidity

file: MagicSpend/MagicSpend.sol

4 import {Ownable} from "solady/auth/Ownable.sol";
import {SignatureCheckerLib} from "solady/utils/SignatureCheckerLib.sol";
import {SafeTransferLib} from "solady/utils/SafeTransferLib.sol";
import {UserOperation} from "account-abstraction/interfaces/UserOperation.sol";
import {IPaymaster} from "account-abstraction/interfaces/IPaymaster.sol";
9 import {IEntryPoint} from "account-abstraction/interfaces/IEntryPoint.sol";


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L4-L9

## [G-05] Consider using OZ EnumerateSet in place of nested mappings
Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).
A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.
OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

```solidity

file: MagicSpend/MagicSpend.sol

37     mapping(uint256 nonce => mapping(address user => bool used)) internal _nonceUsed;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L37

## [G-06] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

```solidity

file: SmartWallet/MultiOwnable.sol

21  mapping(uint256 index => bytes owner) ownerAtIndex;
   
 
24    mapping(bytes account => bool isOwner_) isOwner;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L21-L24

```solidity

file: MagicSpend/MagicSpend.sol

34     mapping(address user => uint256 amount) internal _withdrawableETH;

   
37    mapping(uint256 nonce => mapping(address user => bool used)) internal _nonceUsed;


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L34-L37

## [G-07] Emitting constants wastes gas
Every event parameter costs Glogdata (8 gas) per byte. You can avoid this extra cost, in cases where you're emitting a constant, by creating a new version of the event which doesn't have the parameter (and have users look to the contract's variables for its value instead). Alternatively, in the case of boolean constants, two events can be created - one representing the true case and one representing the false case.

```solidity

file: SmartWallet/MultiOwnable.sol

109   emit RemoveOwner(index, owner);

195 emit AddOwner(index, owner);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L109
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L195

```solidity

file: MagicSpend/MagicSpend.sol

323     emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L323

## [G-08] Superfluous event fields
`block.timestamp` and `block.number` are added to event information by default so adding them manually wastes gas

```solidity

file: MagicSpend/MagicSpend.sol

188  if (block.timestamp > withdrawRequest.expiry)
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L188

## [G-09] Using mappings instead of arrays to avoid length checks save gas
Just by using a mapping, we get a gas saving of 2102 gas. When you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index 

```solidity

file: SmartWallet/MultiOwnable.sol

162  function _initializeOwners(bytes[] memory owners) internal virtual
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L162

```solidity

file: SmartWallet/ERC1271.sol

47     uint256[] memory extensions


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L47

```solidity

file: SmartWallet/CoinbaseSmartWalletFactory.sol

38   function createAccount(bytes[] calldata owners, uint256 nonce)

64   function getAddress(bytes[] calldata owners, uint256 nonce) external view returns (address predicted)

81  function _getSalt(bytes[] calldata owners, uint256 nonce) internal pure returns (bytes32 salt) 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L38
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L64
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L81

```solidity

file: martWallet/CoinbaseSmartWallet.sol

104  bytes[] memory owners = new bytes[](1);

114  function initialize(bytes[] calldata owners) public payable virtual 

205  function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L205

```solidity

file: SmartWallet/CoinbaseSmartWallet.sol

104   bytes[] memory owners = new bytes[](1);

114  function initialize(bytes[] calldata owners) public payable virtual 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114

```solidity

file:  /MagicSpend/MagicSpend.sol#

188 if (block.timestamp > withdrawRequest.expiry) 


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L188

### [G-10] address(this) can be stored in state variable that will ultimately cost less
than every time calculating this specifically when it used multiple times in a contract

```solidity

file: SmartWallet/MultiOwnable.sol

202   if (isOwnerAddress(msg.sender) || (msg.sender == address(this))) 

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L202


```solidity

file: SmartWallet/ERC1271.sol

53   verifyingContract = address(this);

108  address(this)
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L53
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L108

```solidity

file: SmartWallet/CoinbaseSmartWalletFactory.sol

65  predicted = LibClone.predictDeterministicAddress(initCodeHash(), _getSalt(owners, nonce), address(this));

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L65

```solidity

file: MagicSpend/MagicSpend.sol

133 if (address(this).balance < withdrawAmount) {
            revert InsufficientBalance(withdrawAmount, address(this).balance);
}

282 address(this),
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L133
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L282

### [G-11] Call msg.sender directly instead of caching them

In the instance below, instead of caching `msg.sender` and incurring unnecessary stack manipulation, we can call `msg.sender` directly.

```solidity

file: SmartWallet/MultiOwnable.sol

202   if (isOwnerAddress(msg.sender) || (msg.sender == address(this))) 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L202

```solidity

file: MagicSpend/MagicSpend.sol

94  if (msg.sender != entryPoint()) revert Unauthorized();

170  uint256 amount = _withdrawableETH[msg.sender];

174  delete _withdrawableETH[msg.sender];
175  _withdraw(address(0), msg.sender, amount);

182  _validateRequest(msg.sender, withdrawRequest);

184  if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) 

193  _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L94
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L170
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L174-L175 
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L182-L184
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L193


## [G-12] Stack variable cost less while used in emiting event
Even if the variable is going to be used only one time, caching a state variable and use its cache in an emit would help you reduce the cost by at least

```solidity

file: SmartWallet/MultiOwnable.sol

109   emit RemoveOwner(index, owner);

195 emit AddOwner(index, owner);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L109
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L195

```solidity

file: MagicSpend/MagicSpend.sol

323     emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L323

## [G-13] Use function instead of modifiers 

```solidity

file: SmartWallet/MultiOwnable.sol

77 modifier onlyOwner() virtual

193  modifier onlyEntryPoint() virtual 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L77
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L193

## [G-14] Remove or replace unused state variables

```solidity

file: SmartWallet/MultiOwnable.sol

162  function _initializeOwners(bytes[] memory owners) internal virtual
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L162

```solidity

file: SmartWallet/ERC1271.sol

47     uint256[] memory extensions


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L47

```solidity

file: SmartWallet/CoinbaseSmartWalletFactory.sol

38   function createAccount(bytes[] calldata owners, uint256 nonce)

64   function getAddress(bytes[] calldata owners, uint256 nonce) external view returns (address predicted)

81  function _getSalt(bytes[] calldata owners, uint256 nonce) internal pure returns (bytes32 salt) 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L38
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L64
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L81

```solidity

file: martWallet/CoinbaseSmartWallet.sol

104  bytes[] memory owners = new bytes[](1);

114  function initialize(bytes[] calldata owners) public payable virtual 

205  function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L205

```solidity

file: SmartWallet/CoinbaseSmartWallet.sol

104   bytes[] memory owners = new bytes[](1);

114  function initialize(bytes[] calldata owners) public payable virtual 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L104
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114

```solidity

file:  /MagicSpend/MagicSpend.sol#

188 if (block.timestamp > withdrawRequest.expiry) 


```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L188

## [G-15] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas  

```solidity

file: SmartWallet/MultiOwnable.sol

109   emit RemoveOwner(index, owner);

195 emit AddOwner(index, owner);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L109
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L195

```solidity

file: MagicSpend/MagicSpend.sol

323     emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L323

## [G-16] Stack variable cost less while used in emiting event
Even if the variable is going to be used only one time, caching a state variable and use its cache in an emit would help you reduce the cost by at least

```solidity

file: SmartWallet/MultiOwnable.sol

109   emit RemoveOwner(index, owner);

195 emit AddOwner(index, owner);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L109
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L195

```solidity

file: MagicSpend/MagicSpend.sol

323     emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce);

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L323

## [G-17] Using XOR (^) and AND (&) bitwise equivalents for gas optimizations                                      
Given 4 variables a, b, c and d represented as such:

0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d
To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

```solidity

file: SmartWallet/MultiOwnable.sol

104   if (owner.length == 0) revert NoOwnerAtIndex(index);

168  if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) 

202    if (isOwnerAddress(msg.sender) || (msg.sender == address(this)))
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L104
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L168
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L202


```solidity

file: SmartWallet/CoinbaseSmartWalletFactory.sol

44  if (owners.length == 0) 

53       if (alreadyDeployed == false) 
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L44
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L53


```solidity

file: WebAuthnSol/WebAuthn.sol

158   if (success && valid) return abi.decode(ret, (uint256)) == 1;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L158
