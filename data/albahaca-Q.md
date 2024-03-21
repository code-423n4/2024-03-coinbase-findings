## L001 - Governance functions should be controlled by time locks:

Governance functions (such as upgrading contracts, setting critical parameters) should be controlled using time locks to introduce a delay between a proposal and its execution. This gives users time to exit before a potentially dangerous or malicious operation is applied.

```solidity
File: src/MagicSpend/MagicSpend.sol
222         function entryPointWithdraw(address payable to, uint256 amount) external onlyOwner {

248         function entryPointWithdrawStake(address payable to) external onlyOwner {

212         function entryPointDeposit(uint256 amount) external payable onlyOwner {

239         function entryPointUnlockStake() external onlyOwner {

232         function entryPointAddStake(uint256 amount, uint32 unstakeDelaySeconds) external payable onlyOwner {

203         function ownerWithdraw(address asset, address to, uint256 amount) external onlyOwner {

```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L203:205

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
330         function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L330:330

```solidity
File: src/SmartWallet/MultiOwnable.sol
85          function addOwnerAddress(address owner) public virtual onlyOwner {

93          function addOwnerPublicKey(bytes32 x, bytes32 y) public virtual onlyOwner {

102         function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {

```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L102:110




## L002 - Empty `receive()`/`payable fallback()` function does not authorize requests:

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. `require(msg.sender == address(weth))`). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

```solidity
File: src/MagicSpend/MagicSpend.sol
106         receive() external payable {}
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L106:106

## L003 - Double type casts create complexity within the code:

Double type casting should be avoided in Solidity contracts to prevent unintended consequences and ensure accurate data representation. Performing multiple type casts in succession can lead to unexpected truncation, rounding errors, or loss of precision, potentially compromising the contract's functionality and reliability. Furthermore, double type casting can make the code less readable and harder to maintain, increasing the likelihood of errors and misunderstandings during development and debugging. To ensure precise and consistent data handling, developers should use appropriate data types and avoid unnecessary or excessive type casting, promoting a more robust and dependable contract execution.


```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
if (uint256(bytes32(ownerBytes)) > type(uint160).max) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L302:


```solidity
File: src/SmartWallet/MultiOwnable.sol
if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L168:168

## L004 - Constructor contains no validation:

In Solidity, when values are being assigned in constructors to unsigned or integer variables, it's crucial to ensure the provided values adhere to the protocol's specific operational boundaries as laid out in the project specifications and documentation. If the constructors lack appropriate validation checks, there's a risk of setting state variables with values that could cause unexpected and potentially detrimental behavior within the contract's operations, violating the intended logic of the protocol. This can compromise the contract's security and impact the maintainability and reliability of the system. In order to avoid such issues, it is recommended to incorporate rigorous validation checks in constructors. These checks should align with the project's defined rules and constraints, making use of Solidity's built-in require function to enforce these conditions. If the validation checks fail, the require function will cause the transaction to revert, ensuring the integrity and adherence to the protocol's expected behavior.


```solidity
File: src/MagicSpend/MagicSpend.sol
101         constructor(address _owner) {
102             Ownable._initializeOwner(_owner);
103         }
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L101:103

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
24          constructor(address erc4337) payable {
25              implementation = erc4337;
26          }
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L24:26

## L005 - For loops in `public` or `external` functions should be avoided due to high gas costs and possible DOS:

In Solidity, for loops can potentially cause Denial of Service (DoS) attacks if not handled carefully. DoS attacks can occur when an attacker intentionally exploits the gas cost of a function, causing it to run out of gas or making it too expensive for other users to call. Below are some scenarios where for loops can lead to DoS attacks: Nested for loops can become exceptionally gas expensive and should be used sparingly.


```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
205         function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
206             for (uint256 i; i < calls.length;) {
207                 _call(calls[i].target, calls[i].value, calls[i].data);
208                 unchecked {
209                     ++i;
210                 }
211             }
212         }
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L205:212

## L006 - Function calls within for loops:

Making function calls within loops in Solidity can lead to inefficient gas usage, potential bottlenecks, and increased vulnerability to attacks. Each function call or external call consumes gas, and when executed within a loop, the gas cost multiplies, potentially causing the transaction to run out of gas or exceed block gas limits. This can result in transaction failure or unpredictable behavior.


```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
206             for (uint256 i; i < calls.length;) {
207                 _call(calls[i].target, calls[i].value, calls[i].data);
208                 unchecked {
209                     ++i;
210                 }
211             }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L206:211

```solidity
File: src/SmartWallet/MultiOwnable.sol
163             for (uint256 i; i < owners.length; i++) {
164                 if (owners[i].length != 32 && owners[i].length != 64) {
165                     revert InvalidOwnerBytesLength(owners[i]);
166                 }
167     
168                 if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
169                     revert InvalidEthereumAddressOwner(owners[i]);
170                 }
171     
172                 _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
173             }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L163:173

## L007 - External calls in an unbounded for-loop may result in a DoS:

Consider limiting the number of iterations in for-loops that make external calls.


```solidity
File: src/SmartWallet/MultiOwnable.sol
163             for (uint256 i; i < owners.length; i++) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L163:163

## L008 - Missing checks for address(0x0) in the constructor:

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
25              implementation = erc4337;
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L25:25



## L009 - The additions/multiplications may silently overflow because they're in unchecked blocks with no preceding value checks, which may lead to unexpected results.:
The additions/multiplications may silently overflow because they're in unchecked blocks with no preceding value checks, which may lead to unexpected results.

```solidity
File: src/FreshCryptoLib/FCL.sol
137                     scalar_u = addmod(scalar_u, n - scalar_v, n);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L137:137


## L010 - `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`:

Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). Unless there is a compelling reason, `abi.encode` should be preferred. If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739). If all arguments are strings and or bytes, `bytes.concat()` should be used instead.


```solidity
File: src/SmartWallet/ERC1271.sol
return keccak256(abi.encodePacked("\x19\x01", domainSeparator(), _hashStruct(hash)));
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L122:122

## L011 - Missing checks for address(0x0) when updating address state variables:
issing checks for address(0x0) when updating address state variables

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
25              implementation = erc4337;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L25:25

```solidity
File: src/SmartWallet/ERC1271.sol
53              verifyingContract = address(this);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L53:53


## L012 - Consider bounding input array length:

The functions below take in an unbounded array, and make function calls for entries in the array. While the function will revert if it eventually runs out of gas, it may be a nicer user experience to `require()` that the length of the array is below some reasonable maximum, so that the user doesn't have to use up a full transaction's gas only to see that the transaction reverts.

note: this instence missed from bots

```solidity
File: src/SmartWallet/MultiOwnable.sol
163             for (uint256 i; i < owners.length; i++) {
164                 if (owners[i].length != 32 && owners[i].length != 64) {
165                     revert InvalidOwnerBytesLength(owners[i]);
166                 }
167     
168                 if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
169                     revert InvalidEthereumAddressOwner(owners[i]);
170                 }
171     
172                 _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++);
173             }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L163:173




## NC001 - Function ordering does not follow the Solidity style guide:

According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern.

note: missed from bots
```solidity
File: src/MagicSpend/MagicSpend.sol
299         function nonceUsed(address account, uint256 nonce) external view returns (bool) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L299

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
64          function getAddress(bytes[] calldata owners, uint256 nonce) external view returns (address predicted) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L64



## NC002 - Adding a return statement when the function defines a named return variable, is redundant:

If a function defines a named return variable, it is not necessary to explicitly return it. It will automatically be returned at the end of the function.

```solidity
File: src/FreshCryptoLib/FCL.sol
271             return X;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L271


## NC003 - Duplicated `require()`/`revert()` checks should be refactored to a modifier or function:
The compiler will inline the function, which will avoid JUMP instructions usually associated with functions.
```solidity
File: src/FreshCryptoLib/FCL.sol
107                 if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L107


## NC004 - Variable names that consist of all capital letters should be reserved for constant/immutable variables:

If the variable needs to be different based on which class it comes from, a view/pure function should be used instead.

```solidity
File: src/FreshCryptoLib/FCL.sol
83                  uint256 LHS = mulmod(y, y, p); // y^2

84                  uint256 RHS = addmod(mulmod(mulmod(x, x, p), x, p), mulmod(x, a, p), p); // x^3+ax

125             uint256 Y;

127             uint256 H0;

128             uint256 H1;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L128


## NC005 - Large or complicated code bases should implement invariant tests:

Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement invariant fuzzing tests. Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.


## NC006 - Long functions should be refactored into multiple, smaller, functions:


note: this instence missed from bots

```solidity
File: src/FreshCryptoLib/FCL.sol


117         function ecZZ_mulmuladd_S_asm(
118             uint256 Q0,
119             uint256 Q1, //affine rep for input point Q
120             uint256 scalar_u,
121             uint256 scalar_v
122         ) internal view returns (uint256 X) {
123             uint256 zz;
124             uint256 zzz;
125             uint256 Y;
126             uint256 index = 255;
127             uint256 H0;
128             uint256 H1;
129     
130             unchecked {
131                 if (scalar_u == 0 && scalar_v == 0) return 0;
132     
133                 (H0, H1) = ecAff_add(gx, gy, Q0, Q1);
134                 if (
135                     (H0 == 0) && (H1 == 0) //handling Q=-G
136                 ) {
137                     scalar_u = addmod(scalar_u, n - scalar_v, n);
138                     scalar_v = 0;
139                 }
140                 assembly {
141                     for { let T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1)) } eq(T4, 0) {
142                         index := sub(index, 1)
143                         T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
144                     } {}
145                     zz := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
146     
147                     if eq(zz, 1) {
148                         X := gx
149                         Y := gy
150                     }
151                     if eq(zz, 2) {
152                         X := Q0
153                         Y := Q1
154                     }
155                     if eq(zz, 3) {
156                         X := H0
157                         Y := H1
158                     }
159     
160                     index := sub(index, 1)
161                     zz := 1
162                     zzz := 1
163     
164                     for {} gt(minus_1, index) { index := sub(index, 1) } {
165                         // inlined EcZZ_Dbl
166                         let T1 := mulmod(2, Y, p) //U = 2*Y1, y free
167                         let T2 := mulmod(T1, T1, p) // V=U^2
168                         let T3 := mulmod(X, T2, p) // S = X1*V
169                         T1 := mulmod(T1, T2, p) // W=UV
170                         let T4 := mulmod(3, mulmod(addmod(X, sub(p, zz), p), addmod(X, zz, p), p), p) //M=3*(X1-ZZ1)*(X1+ZZ1)
171                         zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
172                         zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
173     
174                         X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
175                         T2 := mulmod(T4, addmod(X, sub(p, T3), p), p) //-M(S-X3)=M(X3-S)
176                         Y := addmod(mulmod(T1, Y, p), T2, p) //-Y3= W*Y1-M(S-X3), we replace Y by -Y to avoid a sub in ecAdd
177     
178                         {
179                             //value of dibit
180                             T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
181     
182                             if iszero(T4) {
183                                 Y := sub(p, Y) //restore the -Y inversion
184                                 continue
185                             } // if T4!=0
186     
187                             if eq(T4, 1) {
188                                 T1 := gx
189                                 T2 := gy
190                             }
191                             if eq(T4, 2) {
192                                 T1 := Q0
193                                 T2 := Q1
194                             }
195                             if eq(T4, 3) {
196                                 T1 := H0
197                                 T2 := H1
198                             }
199                             if iszero(zz) {
200                                 X := T1
201                                 Y := T2
202                                 zz := 1
203                                 zzz := 1
204                                 continue
205                             }
206                             // inlined EcZZ_AddN
207     
208                             //T3:=sub(p, Y)
209                             //T3:=Y
210                             let y2 := addmod(mulmod(T2, zzz, p), Y, p) //R
211                             T2 := addmod(mulmod(T1, zz, p), sub(p, X), p) //P
212     
213                             //special extremely rare case accumulator where EcAdd is replaced by EcDbl, no need to optimize this
214                             //todo : construct edge vector case
215                             if iszero(y2) {
216                                 if iszero(T2) {
217                                     T1 := mulmod(minus_2, Y, p) //U = 2*Y1, y free
218                                     T2 := mulmod(T1, T1, p) // V=U^2
219                                     T3 := mulmod(X, T2, p) // S = X1*V
220     
221                                     T1 := mulmod(T1, T2, p) // W=UV
222                                     y2 := mulmod(addmod(X, zz, p), addmod(X, sub(p, zz), p), p) //(X-ZZ)(X+ZZ)
223                                     T4 := mulmod(3, y2, p) //M=3*(X-ZZ)(X+ZZ)
224     
225                                     zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
226                                     zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
227     
228                                     X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
229                                     T2 := mulmod(T4, addmod(T3, sub(p, X), p), p) //M(S-X3)
230     
231                                     Y := addmod(T2, mulmod(T1, Y, p), p) //Y3= M(S-X3)-W*Y1
232     
233                                     continue
234                                 }
235                             }
236     
237                             T4 := mulmod(T2, T2, p) //PP
238                             let TT1 := mulmod(T4, T2, p) //PPP, this one could be spared, but adding this register spare gas
239                             zz := mulmod(zz, T4, p)
240                             zzz := mulmod(zzz, TT1, p) //zz3=V*ZZ1
241                             let TT2 := mulmod(X, T4, p)
242                             T4 := addmod(addmod(mulmod(y2, y2, p), sub(p, TT1), p), mulmod(minus_2, TT2, p), p)
243                             Y := addmod(mulmod(addmod(TT2, sub(p, T4), p), y2, p), mulmod(Y, TT1, p), p)
244     
245                             X := T4
246                         }
247                     } //end loop
248                     let T := mload(0x40)
249                     mstore(add(T, 0x60), zz)
250                     //(X,Y)=ecZZ_SetAff(X,Y,zz, zzz);
251                     //T[0] = inverseModp_Hard(T[0], p); //1/zzz, inline modular inversion using precompile:
252                     // Define length of base, exponent and modulus. 0x20 == 32 bytes
253                     mstore(T, 0x20)
254                     mstore(add(T, 0x20), 0x20)
255                     mstore(add(T, 0x40), 0x20)
256                     // Define variables base, exponent and modulus
257                     //mstore(add(pointer, 0x60), u)
258                     mstore(add(T, 0x80), minus_2)
259                     mstore(add(T, 0xa0), p)
260     
261                     // Call the precompiled contract 0x05 = ModExp
262                     if iszero(staticcall(not(0), 0x05, T, 0xc0, T, 0x20)) { revert(0, 0) }
263     
264                     //Y:=mulmod(Y,zzz,p)//Y/zzz
265                     //zz :=mulmod(zz, mload(T),p) //1/z
266                     //zz:= mulmod(zz,zz,p) //1/zz
267                     X := mulmod(X, mload(T), p) //X/zz
268                 } //end assembly
269             } //end unchecked
270     
271             return X;
272         }


```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L117-L272



## NC007 - Consider using `block.number` instead of `block.timestamp`:
`block.timestamp` is vulnerable to miner manipulation and creates a potential front-running vulnerability. Consider using `block.number` instead.

```solidity
File: src/MagicSpend/MagicSpend.sol
188             if (block.timestamp > withdrawRequest.expiry) {
```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L188:188


## NC008 - Implement some type of version counter that will be incremented automatically for contract upgrades:

As part of the upgradeability of Proxies , the contract can be upgraded multiple times, where it is a systematic approach to record the version of each upgrade. For instance, you could implement this: uint256 public authorizeUpgradeCounter; function _authorizeUpgrade(address newImplementation) internal { authorizeUpgradeCounter+=1; }


```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
330         function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L330:330



## NC009 - Contract uses both `require()`/`revert()` as well as custom errors:

Consider using just one method in a single file


```solidity
File: src/MagicSpend/MagicSpend.sol
18      contract MagicSpend is Ownable, IPaymaster {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L18:18

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
20      contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271 {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L20:20

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
13      contract CoinbaseSmartWalletFactory {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L13:13

```solidity
File: src/SmartWallet/MultiOwnable.sol
32      contract MultiOwnable {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L32:32


## NC010 - Overridden function has no body:

Consider adding a NatSpec comment describing why the function doesn't need a body and or the purpose it serves.


```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
330         function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L330:330


## NC011 - SPDX-License-Identifier should be on the first line:
The SPDX-License-Identifier is not on the first line of the file, it should always be on the first line.

```solidity
File: src/FreshCryptoLib/FCL.sol
1       //curve order (number of points)
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L1:1



## NC012 - Function names should differ to make the code more readable:

In Solidity, while function overriding allows for functions with the same name to coexist, it is advisable to avoid this practice to enhance code readability and maintainability. Having multiple functions with the same name, even with different parameters or in inherited contracts, can cause confusion and increase the likelihood of errors during development, testing, and debugging. Using distinct and descriptive function names not only clarifies the purpose and behavior of each function, but also helps prevent unintended function calls or incorrect overriding. By adopting a clear and consistent naming convention, developers can create more comprehensible and maintainable smart contracts.

```solidity
File: src/MagicSpend/MagicSpend.sol
    function entryPoint() public pure returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304:306

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
    function entryPoint() public view virtual returns (address) {
        return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
    }

    function _validateSignature(bytes32 message, bytes calldata signature)
        internal
        view
        virtual
        override
        returns (bool)
    {
        SignatureWrapper memory sigWrapper = abi.decode(signature, (SignatureWrapper));
        bytes memory ownerBytes = ownerAtIndex(sigWrapper.ownerIndex);

        if (ownerBytes.length == 32) {
            if (uint256(bytes32(ownerBytes)) > type(uint160).max) {
                // technically should be impossible given owners can only be added with
                // addOwnerAddress and addOwnerPublicKey, but we leave incase of future changes.
                revert InvalidEthereumAddressOwner(ownerBytes);
            }

            address owner;
            assembly ("memory-safe") {
                owner := mload(add(ownerBytes, 32))
            }

            return SignatureCheckerLib.isValidSignatureNow(owner, message, sigWrapper.signatureData);
        }

        if (ownerBytes.length == 64) {
            (uint256 x, uint256 y) = abi.decode(ownerBytes, (uint256, uint256));

            WebAuthn.WebAuthnAuth memory auth = abi.decode(sigWrapper.signatureData, (WebAuthn.WebAuthnAuth));

            return WebAuthn.verify({challenge: abi.encode(message), requireUV: false, webAuthnAuth: auth, x: x, y: y});
        }

        revert InvalidOwnerBytesLength(ownerBytes);
    }

    function _domainNameAndVersion() internal pure override(ERC1271) returns (string memory, string memory) {
        return ("Coinbase Smart Wallet", "1");
    }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L333:335

```solidity
File: src/SmartWallet/ERC1271.sol
    function _validateSignature(bytes32 message, bytes calldata signature) internal view virtual returns (bool);

    function _domainNameAndVersion() internal view virtual returns (string memory name, string memory version);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L143:143



## NC013 - It is standard for all external and public functions to be override from an interface:

This is to ensure the whole API is extracted in an interface


```solidity
File: src/MagicSpend/MagicSpend.sol


106         receive() external payable {}


109         function validatePaymasterUserOp(UserOperation calldata userOp, bytes32, uint256 maxCost)
110             external
111             onlyEntryPoint
112             returns (bytes memory context, uint256 validationData)
113         {
114             WithdrawRequest memory withdrawRequest = abi.decode(userOp.paymasterAndData[20:], (WithdrawRequest));
115             uint256 withdrawAmount = withdrawRequest.amount;
116     
117             if (withdrawAmount < maxCost) {
118                 revert RequestLessThanGasMaxCost(withdrawAmount, maxCost);
119             }
120     
121             if (withdrawRequest.asset != address(0)) {
122                 revert UnsupportedPaymasterAsset(withdrawRequest.asset);
123             }
124     
125             _validateRequest(userOp.sender, withdrawRequest);
126     
127             bool sigFailed = !isValidWithdrawSignature(userOp.sender, withdrawRequest);
128             validationData = (sigFailed ? 1 : 0) | (uint256(withdrawRequest.expiry) << 160);
129     
130             // Ensure at validation that the contract has enough balance to cover the requested funds.
131             // NOTE: This check is necessary to enforce that the contract will be able to transfer the remaining funds
132             //       when `postOp()` is called back after the `UserOperation` has been executed.
133             if (address(this).balance < withdrawAmount) {
134                 revert InsufficientBalance(withdrawAmount, address(this).balance);
135             }
136     
137             // NOTE: Do not include the gas part in withdrawable funds as it will be handled in `postOp()`.
138             _withdrawableETH[userOp.sender] += withdrawAmount - maxCost;
139             context = abi.encode(maxCost, userOp.sender);
140         }


143         function postOp(IPaymaster.PostOpMode mode, bytes calldata context, uint256 actualGasCost)
144             external
145             onlyEntryPoint
146         {
147             // `PostOpMode.postOpReverted` should be impossible.
148             // Only possible cause would be if this contract does not own enough ETH to transfer
149             // but this is checked at the validation step.
150             assert(mode != PostOpMode.postOpReverted);
151     
152             (uint256 maxGasCost, address account) = abi.decode(context, (uint256, address));
153     
154             // Compute the total remaining funds available for the user accout.
155             // NOTE: Take into account the user operation gas that was not consummed.
156             uint256 withdrawable = _withdrawableETH[account] + (maxGasCost - actualGasCost);
157     
158             // Send the all remaining funds to the user accout.
159             delete _withdrawableETH[account];
160             if (withdrawable > 0) {
161                 SafeTransferLib.forceSafeTransferETH(account, withdrawable, SafeTransferLib.GAS_STIPEND_NO_STORAGE_WRITES);
162             }
163         }


169         function withdrawGasExcess() external {
170             uint256 amount = _withdrawableETH[msg.sender];
171             // we could allow 0 value transfers, but prefer to be explicit
172             if (amount == 0) revert NoExcess();
173     
174             delete _withdrawableETH[msg.sender];
175             _withdraw(address(0), msg.sender, amount);
176         }


181         function withdraw(WithdrawRequest memory withdrawRequest) external {
182             _validateRequest(msg.sender, withdrawRequest);
183     
184             if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
185                 revert InvalidSignature();
186             }
187     
188             if (block.timestamp > withdrawRequest.expiry) {
189                 revert Expired();
190             }
191     
192             // reserve funds for gas, will credit user with difference in post op
193             _withdraw(withdrawRequest.asset, msg.sender, withdrawRequest.amount);
194         }


203         function ownerWithdraw(address asset, address to, uint256 amount) external onlyOwner {
204             _withdraw(asset, to, amount);
205         }


212         function entryPointDeposit(uint256 amount) external payable onlyOwner {
213             SafeTransferLib.safeTransferETH(entryPoint(), amount);
214         }


222         function entryPointWithdraw(address payable to, uint256 amount) external onlyOwner {
223             IEntryPoint(entryPoint()).withdrawTo(to, amount);
224         }


232         function entryPointAddStake(uint256 amount, uint32 unstakeDelaySeconds) external payable onlyOwner {
233             IEntryPoint(entryPoint()).addStake{value: amount}(unstakeDelaySeconds);
234         }


239         function entryPointUnlockStake() external onlyOwner {
240             IEntryPoint(entryPoint()).unlockStake();
241         }


248         function entryPointWithdrawStake(address payable to) external onlyOwner {
249             IEntryPoint(entryPoint()).withdrawStake(to);
250         }


260         function isValidWithdrawSignature(address account, WithdrawRequest memory withdrawRequest)
261             public
262             view
263             returns (bool)
264         {
265             return SignatureCheckerLib.isValidSignatureNow(
266                 owner(), getHash(account, withdrawRequest), withdrawRequest.signature
267             );
268         }


279         function getHash(address account, WithdrawRequest memory withdrawRequest) public view returns (bytes32) {
280             return SignatureCheckerLib.toEthSignedMessageHash(
281                 abi.encode(
282                     address(this),
283                     account,
284                     block.chainid,
285                     withdrawRequest.asset,
286                     withdrawRequest.amount,
287                     withdrawRequest.nonce,
288                     withdrawRequest.expiry
289                 )
290             );
291         }


299         function nonceUsed(address account, uint256 nonce) external view returns (bool) {
300             return _nonceUsed[nonce][account];
301         }


304         function entryPoint() public pure returns (address) {
305             return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
306         }


```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304:306

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol


114         function initialize(bytes[] calldata owners) public payable virtual {
115             if (nextOwnerIndex() != 0) {
116                 revert Initialized();
117             }
118     
119             _initializeOwners(owners);
120         }


137         function validateUserOp(UserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)
138             public
139             payable
140             virtual
141             onlyEntryPoint
142             payPrefund(missingAccountFunds)
143             returns (uint256 validationData)
144         {
145             uint256 key = userOp.nonce >> 64;
146     
147             // 0xbf6ba1fc = bytes4(keccak256("executeWithoutChainIdValidation(bytes)"))
148             if (userOp.callData.length >= 4 && bytes4(userOp.callData[0:4]) == 0xbf6ba1fc) {
149                 userOpHash = getUserOpHashWithoutChainId(userOp);
150                 if (key != REPLAYABLE_NONCE_KEY) {
151                     revert InvalidNonceKey(key);
152                 }
153             } else {
154                 if (key == REPLAYABLE_NONCE_KEY) {
155                     revert InvalidNonceKey(key);
156                 }
157             }
158     
159             // Return 0 if the recovered address matches the owner.
160             if (_validateSignature(userOpHash, userOp.signature)) {
161                 return 0;
162             }
163     
164             // Else return 1, which is equivalent to:
165             // `(uint256(validAfter) << (160 + 48)) | (uint256(validUntil) << 160) | (success ? 0 : 1)`
166             // where `validUntil` is 0 (indefinite) and `validAfter` is 0.
167             return 1;
168         }


180         function executeWithoutChainIdValidation(bytes calldata data) public payable virtual onlyEntryPoint {
181             bytes4 selector = bytes4(data[0:4]);
182             if (!canSkipChainIdValidation(selector)) {
183                 revert SelectorNotAllowed(selector);
184             }
185     
186             _call(address(this), 0, data);
187         }


196         function execute(address target, uint256 value, bytes calldata data) public payable virtual onlyEntryPointOrOwner {
197             _call(target, value, data);
198         }


205         function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
206             for (uint256 i; i < calls.length;) {
207                 _call(calls[i].target, calls[i].value, calls[i].data);
208                 unchecked {
209                     ++i;
210                 }
211             }
212         }


217         function entryPoint() public view virtual returns (address) {
218             return 0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789;
219         }


229         function getUserOpHashWithoutChainId(UserOperation calldata userOp)
230             public
231             view
232             virtual
233             returns (bytes32 userOpHash)
234         {
235             return keccak256(abi.encode(UserOperationLib.hash(userOp), entryPoint()));
236         }


241         function implementation() public view returns (address $) {
242             assembly {
243                 $ := sload(_ERC1967_IMPLEMENTATION_SLOT)
244             }
245         }


252         function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
253             if (
254                 functionSelector == MultiOwnable.addOwnerPublicKey.selector
255                     || functionSelector == MultiOwnable.addOwnerAddress.selector
256                     || functionSelector == MultiOwnable.removeOwnerAtIndex.selector
257                     || functionSelector == UUPSUpgradeable.upgradeToAndCall.selector
258             ) {
259                 return true;
260             }
261             return false;
262         }


```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L252:262

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol


38          function createAccount(bytes[] calldata owners, uint256 nonce)
39              public
40              payable
41              virtual
42              returns (CoinbaseSmartWallet account)
43          {
44              if (owners.length == 0) {
45                  revert OwnerRequired();
46              }
47      
48              (bool alreadyDeployed, address accountAddress) =
49                  LibClone.createDeterministicERC1967(msg.value, implementation, _getSalt(owners, nonce));
50      
51              account = CoinbaseSmartWallet(payable(accountAddress));
52      
53              if (alreadyDeployed == false) {
54                  account.initialize(owners);
55              }
56          }


64          function getAddress(bytes[] calldata owners, uint256 nonce) external view returns (address predicted) {
65              predicted = LibClone.predictDeterministicAddress(initCodeHash(), _getSalt(owners, nonce), address(this));
66          }


71          function initCodeHash() public view virtual returns (bytes32 result) {
72              result = LibClone.initCodeHashERC1967(implementation);
73          }


```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L71:73

```solidity
File: src/SmartWallet/MultiOwnable.sol


85          function addOwnerAddress(address owner) public virtual onlyOwner {
86              _addOwner(abi.encode(owner));
87          }


93          function addOwnerPublicKey(bytes32 x, bytes32 y) public virtual onlyOwner {
94              _addOwner(abi.encode(x, y));
95          }


102         function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {
103             bytes memory owner = ownerAtIndex(index);
104             if (owner.length == 0) revert NoOwnerAtIndex(index);
105     
106             delete _getMultiOwnableStorage().isOwner[owner];
107             delete _getMultiOwnableStorage().ownerAtIndex[index];
108     
109             emit RemoveOwner(index, owner);
110         }


117         function isOwnerAddress(address account) public view virtual returns (bool) {
118             return _getMultiOwnableStorage().isOwner[abi.encode(account)];
119         }


127         function isOwnerPublicKey(bytes32 x, bytes32 y) public view virtual returns (bool) {
128             return _getMultiOwnableStorage().isOwner[abi.encode(x, y)];
129         }


136         function isOwnerBytes(bytes memory account) public view virtual returns (bool) {
137             return _getMultiOwnableStorage().isOwner[account];
138         }


145         function ownerAtIndex(uint256 index) public view virtual returns (bytes memory) {
146             return _getMultiOwnableStorage().ownerAtIndex[index];
147         }


152         function nextOwnerIndex() public view virtual returns (uint256) {
153             return _getMultiOwnableStorage().nextOwnerIndex;
154         }


```

https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L152:154


## NC014 - Consider adding formal verification proofs:

Consider using formal verification to mathematically prove that your code does what is intended, and does not have any edge cases with unexpected behavior. The solidity compiler itself has this functionality [built in based off of SMTChecker](https://docs.soliditylang.org/en/latest/smtchecker.html#smtchecker-and-formal-verification).


## NC015 - Assembly code blocks should be thoroughly commented:

In Solidity, assembly blocks are often harder to read and understand due to their low-level nature. Detailed comments can make the code's purpose and functionality clear, aiding in maintenance, debugging, and reviews. Moreover, comments can be crucial for auditors and other developers to understand the contract's security implications, reducing the risk of oversights and vulnerabilities.



```solidity
File: src/FreshCryptoLib/FCL.sol
107                 if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }


140                 assembly {
141                     for { let T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1)) } eq(T4, 0) {
142                         index := sub(index, 1)
143                         T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
144                     } {}
145                     zz := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
146     
147                     if eq(zz, 1) {
148                         X := gx
149                         Y := gy
150                     }
151                     if eq(zz, 2) {
152                         X := Q0
153                         Y := Q1
154                     }
155                     if eq(zz, 3) {
156                         X := H0
157                         Y := H1
158                     }
159     
160                     index := sub(index, 1)
161                     zz := 1
162                     zzz := 1
163     
164                     for {} gt(minus_1, index) { index := sub(index, 1) } {
165                         // inlined EcZZ_Dbl
166                         let T1 := mulmod(2, Y, p) //U = 2*Y1, y free
167                         let T2 := mulmod(T1, T1, p) // V=U^2
168                         let T3 := mulmod(X, T2, p) // S = X1*V
169                         T1 := mulmod(T1, T2, p) // W=UV
170                         let T4 := mulmod(3, mulmod(addmod(X, sub(p, zz), p), addmod(X, zz, p), p), p) //M=3*(X1-ZZ1)*(X1+ZZ1)
171                         zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
172                         zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
173     
174                         X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
175                         T2 := mulmod(T4, addmod(X, sub(p, T3), p), p) //-M(S-X3)=M(X3-S)
176                         Y := addmod(mulmod(T1, Y, p), T2, p) //-Y3= W*Y1-M(S-X3), we replace Y by -Y to avoid a sub in ecAdd
177     
178                         {
179                             //value of dibit
180                             T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
181     
182                             if iszero(T4) {
183                                 Y := sub(p, Y) //restore the -Y inversion
184                                 continue
185                             } // if T4!=0
186     
187                             if eq(T4, 1) {
188                                 T1 := gx
189                                 T2 := gy
190                             }
191                             if eq(T4, 2) {
192                                 T1 := Q0
193                                 T2 := Q1
194                             }
195                             if eq(T4, 3) {
196                                 T1 := H0
197                                 T2 := H1
198                             }
199                             if iszero(zz) {
200                                 X := T1
201                                 Y := T2
202                                 zz := 1
203                                 zzz := 1
204                                 continue
205                             }
206                             // inlined EcZZ_AddN
207     
208                             //T3:=sub(p, Y)
209                             //T3:=Y
210                             let y2 := addmod(mulmod(T2, zzz, p), Y, p) //R
211                             T2 := addmod(mulmod(T1, zz, p), sub(p, X), p) //P
212     
213                             //special extremely rare case accumulator where EcAdd is replaced by EcDbl, no need to optimize this
214                             //todo : construct edge vector case
215                             if iszero(y2) {
216                                 if iszero(T2) {
217                                     T1 := mulmod(minus_2, Y, p) //U = 2*Y1, y free
218                                     T2 := mulmod(T1, T1, p) // V=U^2
219                                     T3 := mulmod(X, T2, p) // S = X1*V
220     
221                                     T1 := mulmod(T1, T2, p) // W=UV
222                                     y2 := mulmod(addmod(X, zz, p), addmod(X, sub(p, zz), p), p) //(X-ZZ)(X+ZZ)
223                                     T4 := mulmod(3, y2, p) //M=3*(X-ZZ)(X+ZZ)
224     
225                                     zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
226                                     zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
227     
228                                     X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
229                                     T2 := mulmod(T4, addmod(T3, sub(p, X), p), p) //M(S-X3)
230     
231                                     Y := addmod(T2, mulmod(T1, Y, p), p) //Y3= M(S-X3)-W*Y1
232     
233                                     continue
234                                 }
235                             }
236     
237                             T4 := mulmod(T2, T2, p) //PP
238                             let TT1 := mulmod(T4, T2, p) //PPP, this one could be spared, but adding this register spare gas
239                             zz := mulmod(zz, T4, p)
240                             zzz := mulmod(zzz, TT1, p) //zz3=V*ZZ1
241                             let TT2 := mulmod(X, T4, p)
242                             T4 := addmod(addmod(mulmod(y2, y2, p), sub(p, TT1), p), mulmod(minus_2, TT2, p), p)
243                             Y := addmod(mulmod(addmod(TT2, sub(p, T4), p), y2, p), mulmod(Y, TT1, p), p)
244     
245                             X := T4
246                         }
247                     } //end loop
248                     let T := mload(0x40)
249                     mstore(add(T, 0x60), zz)
250                     //(X,Y)=ecZZ_SetAff(X,Y,zz, zzz);
251                     //T[0] = inverseModp_Hard(T[0], p); //1/zzz, inline modular inversion using precompile:
252                     // Define length of base, exponent and modulus. 0x20 == 32 bytes
253                     mstore(T, 0x20)
254                     mstore(add(T, 0x20), 0x20)
255                     mstore(add(T, 0x40), 0x20)
256                     // Define variables base, exponent and modulus
257                     //mstore(add(pointer, 0x60), u)
258                     mstore(add(T, 0x80), minus_2)
259                     mstore(add(T, 0xa0), p)
260     
261                     // Call the precompiled contract 0x05 = ModExp
262                     if iszero(staticcall(not(0), 0x05, T, 0xc0, T, 0x20)) { revert(0, 0) }
263     
264                     //Y:=mulmod(Y,zzz,p)//Y/zzz
265                     //zz :=mulmod(zz, mload(T),p) //1/z
266                     //zz:= mulmod(zz,zz,p) //1/zz
267                     X := mulmod(X, mload(T), p) //X/zz
268                 } //end assembly


141                     for { let T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1)) } eq(T4, 0) {


141                     for { let T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1)) } eq(T4, 0) {
142                         index := sub(index, 1)
143                         T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
144                     } {}


144                     } {}


147                     if eq(zz, 1) {
148                         X := gx
149                         Y := gy
150                     }


151                     if eq(zz, 2) {
152                         X := Q0
153                         Y := Q1
154                     }


155                     if eq(zz, 3) {
156                         X := H0
157                         Y := H1
158                     }


164                     for {} gt(minus_1, index) { index := sub(index, 1) } {


164                     for {} gt(minus_1, index) { index := sub(index, 1) } {


164                     for {} gt(minus_1, index) { index := sub(index, 1) } {
165                         // inlined EcZZ_Dbl
166                         let T1 := mulmod(2, Y, p) //U = 2*Y1, y free
167                         let T2 := mulmod(T1, T1, p) // V=U^2
168                         let T3 := mulmod(X, T2, p) // S = X1*V
169                         T1 := mulmod(T1, T2, p) // W=UV
170                         let T4 := mulmod(3, mulmod(addmod(X, sub(p, zz), p), addmod(X, zz, p), p), p) //M=3*(X1-ZZ1)*(X1+ZZ1)
171                         zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
172                         zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
173     
174                         X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
175                         T2 := mulmod(T4, addmod(X, sub(p, T3), p), p) //-M(S-X3)=M(X3-S)
176                         Y := addmod(mulmod(T1, Y, p), T2, p) //-Y3= W*Y1-M(S-X3), we replace Y by -Y to avoid a sub in ecAdd
177     
178                         {
179                             //value of dibit
180                             T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
181     
182                             if iszero(T4) {
183                                 Y := sub(p, Y) //restore the -Y inversion
184                                 continue
185                             } // if T4!=0
186     
187                             if eq(T4, 1) {
188                                 T1 := gx
189                                 T2 := gy
190                             }
191                             if eq(T4, 2) {
192                                 T1 := Q0
193                                 T2 := Q1
194                             }
195                             if eq(T4, 3) {
196                                 T1 := H0
197                                 T2 := H1
198                             }
199                             if iszero(zz) {
200                                 X := T1
201                                 Y := T2
202                                 zz := 1
203                                 zzz := 1
204                                 continue
205                             }
206                             // inlined EcZZ_AddN
207     
208                             //T3:=sub(p, Y)
209                             //T3:=Y
210                             let y2 := addmod(mulmod(T2, zzz, p), Y, p) //R
211                             T2 := addmod(mulmod(T1, zz, p), sub(p, X), p) //P
212     
213                             //special extremely rare case accumulator where EcAdd is replaced by EcDbl, no need to optimize this
214                             //todo : construct edge vector case
215                             if iszero(y2) {
216                                 if iszero(T2) {
217                                     T1 := mulmod(minus_2, Y, p) //U = 2*Y1, y free
218                                     T2 := mulmod(T1, T1, p) // V=U^2
219                                     T3 := mulmod(X, T2, p) // S = X1*V
220     
221                                     T1 := mulmod(T1, T2, p) // W=UV
222                                     y2 := mulmod(addmod(X, zz, p), addmod(X, sub(p, zz), p), p) //(X-ZZ)(X+ZZ)
223                                     T4 := mulmod(3, y2, p) //M=3*(X-ZZ)(X+ZZ)
224     
225                                     zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
226                                     zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
227     
228                                     X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
229                                     T2 := mulmod(T4, addmod(T3, sub(p, X), p), p) //M(S-X3)
230     
231                                     Y := addmod(T2, mulmod(T1, Y, p), p) //Y3= M(S-X3)-W*Y1
232     
233                                     continue
234                                 }
235                             }
236     
237                             T4 := mulmod(T2, T2, p) //PP
238                             let TT1 := mulmod(T4, T2, p) //PPP, this one could be spared, but adding this register spare gas
239                             zz := mulmod(zz, T4, p)
240                             zzz := mulmod(zzz, TT1, p) //zz3=V*ZZ1
241                             let TT2 := mulmod(X, T4, p)
242                             T4 := addmod(addmod(mulmod(y2, y2, p), sub(p, TT1), p), mulmod(minus_2, TT2, p), p)
243                             Y := addmod(mulmod(addmod(TT2, sub(p, T4), p), y2, p), mulmod(Y, TT1, p), p)
244     
245                             X := T4
246                         }
247                     } //end loop


178                         {
179                             //value of dibit
180                             T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1))
181     
182                             if iszero(T4) {
183                                 Y := sub(p, Y) //restore the -Y inversion
184                                 continue
185                             } // if T4!=0
186     
187                             if eq(T4, 1) {
188                                 T1 := gx
189                                 T2 := gy
190                             }
191                             if eq(T4, 2) {
192                                 T1 := Q0
193                                 T2 := Q1
194                             }
195                             if eq(T4, 3) {
196                                 T1 := H0
197                                 T2 := H1
198                             }
199                             if iszero(zz) {
200                                 X := T1
201                                 Y := T2
202                                 zz := 1
203                                 zzz := 1
204                                 continue
205                             }
206                             // inlined EcZZ_AddN
207     
208                             //T3:=sub(p, Y)
209                             //T3:=Y
210                             let y2 := addmod(mulmod(T2, zzz, p), Y, p) //R
211                             T2 := addmod(mulmod(T1, zz, p), sub(p, X), p) //P
212     
213                             //special extremely rare case accumulator where EcAdd is replaced by EcDbl, no need to optimize this
214                             //todo : construct edge vector case
215                             if iszero(y2) {
216                                 if iszero(T2) {
217                                     T1 := mulmod(minus_2, Y, p) //U = 2*Y1, y free
218                                     T2 := mulmod(T1, T1, p) // V=U^2
219                                     T3 := mulmod(X, T2, p) // S = X1*V
220     
221                                     T1 := mulmod(T1, T2, p) // W=UV
222                                     y2 := mulmod(addmod(X, zz, p), addmod(X, sub(p, zz), p), p) //(X-ZZ)(X+ZZ)
223                                     T4 := mulmod(3, y2, p) //M=3*(X-ZZ)(X+ZZ)
224     
225                                     zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
226                                     zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
227     
228                                     X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
229                                     T2 := mulmod(T4, addmod(T3, sub(p, X), p), p) //M(S-X3)
230     
231                                     Y := addmod(T2, mulmod(T1, Y, p), p) //Y3= M(S-X3)-W*Y1
232     
233                                     continue
234                                 }
235                             }
236     
237                             T4 := mulmod(T2, T2, p) //PP
238                             let TT1 := mulmod(T4, T2, p) //PPP, this one could be spared, but adding this register spare gas
239                             zz := mulmod(zz, T4, p)
240                             zzz := mulmod(zzz, TT1, p) //zz3=V*ZZ1
241                             let TT2 := mulmod(X, T4, p)
242                             T4 := addmod(addmod(mulmod(y2, y2, p), sub(p, TT1), p), mulmod(minus_2, TT2, p), p)
243                             Y := addmod(mulmod(addmod(TT2, sub(p, T4), p), y2, p), mulmod(Y, TT1, p), p)
244     
245                             X := T4
246                         }


182                             if iszero(T4) {
183                                 Y := sub(p, Y) //restore the -Y inversion
184                                 continue
185                             } // if T4!=0


187                             if eq(T4, 1) {
188                                 T1 := gx
189                                 T2 := gy
190                             }


191                             if eq(T4, 2) {
192                                 T1 := Q0
193                                 T2 := Q1
194                             }


195                             if eq(T4, 3) {
196                                 T1 := H0
197                                 T2 := H1
198                             }


199                             if iszero(zz) {
200                                 X := T1
201                                 Y := T2
202                                 zz := 1
203                                 zzz := 1
204                                 continue
205                             }


215                             if iszero(y2) {
216                                 if iszero(T2) {
217                                     T1 := mulmod(minus_2, Y, p) //U = 2*Y1, y free
218                                     T2 := mulmod(T1, T1, p) // V=U^2
219                                     T3 := mulmod(X, T2, p) // S = X1*V
220     
221                                     T1 := mulmod(T1, T2, p) // W=UV
222                                     y2 := mulmod(addmod(X, zz, p), addmod(X, sub(p, zz), p), p) //(X-ZZ)(X+ZZ)
223                                     T4 := mulmod(3, y2, p) //M=3*(X-ZZ)(X+ZZ)
224     
225                                     zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
226                                     zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
227     
228                                     X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
229                                     T2 := mulmod(T4, addmod(T3, sub(p, X), p), p) //M(S-X3)
230     
231                                     Y := addmod(T2, mulmod(T1, Y, p), p) //Y3= M(S-X3)-W*Y1
232     
233                                     continue
234                                 }
235                             }


216                                 if iszero(T2) {
217                                     T1 := mulmod(minus_2, Y, p) //U = 2*Y1, y free
218                                     T2 := mulmod(T1, T1, p) // V=U^2
219                                     T3 := mulmod(X, T2, p) // S = X1*V
220     
221                                     T1 := mulmod(T1, T2, p) // W=UV
222                                     y2 := mulmod(addmod(X, zz, p), addmod(X, sub(p, zz), p), p) //(X-ZZ)(X+ZZ)
223                                     T4 := mulmod(3, y2, p) //M=3*(X-ZZ)(X+ZZ)
224     
225                                     zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
226                                     zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
227     
228                                     X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
229                                     T2 := mulmod(T4, addmod(T3, sub(p, X), p), p) //M(S-X3)
230     
231                                     Y := addmod(T2, mulmod(T1, Y, p), p) //Y3= M(S-X3)-W*Y1
232     
233                                     continue
234                                 }


262                     if iszero(staticcall(not(0), 0x05, T, 0xc0, T, 0x20)) { revert(0, 0) }


324                 assembly {
325                     P0 := mulmod(2, y, p) //U = 2*Y1
326                     P2 := mulmod(P0, P0, p) // V=U^2
327                     P3 := mulmod(x, P2, p) // S = X1*V
328                     P1 := mulmod(P0, P2, p) // W=UV
329                     P2 := mulmod(P2, zz, p) //zz3=V*ZZ1
330                     zz := mulmod(3, mulmod(addmod(x, sub(p, zz), p), addmod(x, zz, p), p), p) //M=3*(X1-ZZ1)*(X1+ZZ1)
331                     P0 := addmod(mulmod(zz, zz, p), mulmod(minus_2, P3, p), p) //X3=M^2-2S
332                     x := mulmod(zz, addmod(P3, sub(p, P0), p), p) //M(S-X3)
333                     P3 := mulmod(P1, zzz, p) //zzz3=W*zzz1
334                     P1 := addmod(x, sub(p, mulmod(P1, y, p)), p) //Y3= M(S-X3)-W*Y1
335                 }


354                 assembly {
355                     y1 := sub(p, y1)
356                     y2 := addmod(mulmod(y2, zzz1, p), y1, p)
357                     x2 := addmod(mulmod(x2, zz1, p), sub(p, x1), p)
358                     P0 := mulmod(x2, x2, p) //PP = P^2
359                     P1 := mulmod(P0, x2, p) //PPP = P*PP
360                     P2 := mulmod(zz1, P0, p) ////ZZ3 = ZZ1*PP
361                     P3 := mulmod(zzz1, P1, p) ////ZZZ3 = ZZZ1*PPP
362                     zz1 := mulmod(x1, P0, p) //Q = X1*PP
363                     P0 := addmod(addmod(mulmod(y2, y2, p), sub(p, P1), p), mulmod(minus_2, zz1, p), p) //R^2-PPP-2*Q
364                     P1 := addmod(mulmod(addmod(zz1, sub(p, P0), p), y2, p), mulmod(y1, P1, p), p) //R*(Q-X3)
365                 }


375             assembly {
376                 let pointer := mload(0x40)
377                 // Define length of base, exponent and modulus. 0x20 == 32 bytes
378                 mstore(pointer, 0x20)
379                 mstore(add(pointer, 0x20), 0x20)
380                 mstore(add(pointer, 0x40), 0x20)
381                 // Define variables base, exponent and modulus
382                 mstore(add(pointer, 0x60), u)
383                 mstore(add(pointer, 0x80), minus_2)
384                 mstore(add(pointer, 0xa0), p)
385     
386                 // Call the precompiled contract 0x05 = ModExp
387                 if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }
388                 result := mload(pointer)
389             }


387                 if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L387:387

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
95                  if missingAccountFunds {
96                      // Ignore failure (it's EntryPoint's job to verify, not the account's).
97                      pop(call(gas(), caller(), missingAccountFunds, codesize(), 0x00, codesize(), 0x00))
98                  }


242             assembly {
243                 $ := sload(_ERC1967_IMPLEMENTATION_SLOT)
244             }


275                 assembly ("memory-safe") {
276                     revert(add(result, 32), mload(result))
277                 }


309                 assembly ("memory-safe") {
310                     owner := mload(add(ownerBytes, 32))
311                 }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L309:311

```solidity
File: src/SmartWallet/MultiOwnable.sol
213             assembly ("memory-safe") {
214                 $.slot := MUTLI_OWNABLE_STORAGE_LOCATION
215             }
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L213:



## NC016 - Add inline comments for unnamed variables:

`function foo(address x, address)` -> `function foo(address x, address /* y */)`

```solidity
File: src/MagicSpend/MagicSpend.sol
109         function validatePaymasterUserOp(UserOperation calldata userOp, bytes32, uint256 maxCost)
110             external
111             onlyEntryPoint
112             returns (bytes memory context, uint256 validationData)
113         {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L109:113

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
330         function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L330:330


## NC017 - Consider adding emergency-stop functionality:
Adding a way to quickly halt protocol functionality in an emergency, rather than having to pause individual contracts one-by-one, will make in-progress hack mitigation faster and much less stressful.


```solidity
File: src/MagicSpend/MagicSpend.sol
18      contract MagicSpend is Ownable, IPaymaster {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L18:18

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
20      contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271 {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L20:20

```solidity
File: src/SmartWallet/MultiOwnable.sol
32      contract MultiOwnable {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L32:32



## NC018 - If statement control structures do not comply with best practices:

If statements which include a single line do not need to have curly brackets, however according to the Solidiity style guide the line of code executed upon the if statement condition being met should still be on the next line, not on the same line as the if statement declaration.

```solidity
File: src/FreshCryptoLib/FCL.sol
131                 if (scalar_u == 0 && scalar_v == 0) return 0;

278             if (ecAff_IsZero(x0, y0)) return (x1, y1);

279             if (ecAff_IsZero(x1, y1)) return (x0, y0);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L279:279

```solidity
File: src/MagicSpend/MagicSpend.sol
94              if (msg.sender != entryPoint()) revert Unauthorized();

172             if (amount == 0) revert NoExcess();
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L172:172

```solidity
File: src/SmartWallet/MultiOwnable.sol
104             if (owner.length == 0) revert NoOwnerAtIndex(index);

190             if (isOwnerBytes(owner)) revert AlreadyOwner(owner);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L190:190

```solidity
File: src/WebAuthnSol/WebAuthn.sol
158             if (success && valid) return abi.decode(ret, (uint256)) == 1;
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L158:158


## NC019 - It is best practice to use linear inheritance:

In Solidity, complex inheritance structures can obfuscate code understanding, introducing potential security risks. Multiple inheritance, especially with overlapping function names or state variables, can cause unintentional overrides or ambiguous behavior. Resolution: Strive for linear and simple inheritance chains. Avoid diamond or circular inheritance patterns. Clearly document the purpose and relationships of base contracts, ensuring that overrides are intentional. Tools like Remix or Hardhat can visualize inheritance chains, assisting in verification. Keeping inheritance streamlined aids in better code readability, reduces potential errors, and ensures smoother audits and upgrades.


```solidity
File: src/MagicSpend/MagicSpend.sol
18      contract MagicSpend is Ownable, IPaymaster {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L18:18

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
20      contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271 {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L20:20


## NC020 - Use a struct to encapsulate multiple function parameters:

Using a struct to encapsulate multiple parameters in Solidity functions can significantly enhance code readability and maintainability. Instead of passing a long list of arguments, which can be error-prone and hard to manage, a struct allows grouping related data into a single, coherent entity. This approach simplifies function signatures and makes the code more organized. It also enhances code clarity, as developers can easily understand the relationship between the parameters. Moreover, it aids in future code modifications and expansions, as adding or modifying a parameter only requires changes in the struct definition, rather than in every function that uses these parameters.


```solidity
File: src/FreshCryptoLib/FCL.sol
50          function ecdsa_verify(bytes32 message, uint256 r, uint256 s, uint256 Qx, uint256 Qy) internal view returns (bool) {


344         function ecZZ_AddN(uint256 x1, uint256 y1, uint256 zz1, uint256 zzz1, uint256 x2, uint256 y2)
345             internal
346             pure
347             returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
348         {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L344:348

```solidity
File: src/WebAuthnSol/WebAuthn.sol
104         function verify(bytes memory challenge, bool requireUV, WebAuthnAuth memory webAuthnAuth, uint256 x, uint256 y)
105             internal
106             view
107             returns (bool)
108         {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L104:108





## NC021 - Defining All External/Public Functions in Contract Interfaces:

It is preferable to have all the external and public function in an interface to make using them easier by developers. This helps ensure the whole API is extracted in a interface.

```solidity
File: src/MagicSpend/MagicSpend.sol
260         function isValidWithdrawSignature(address account, WithdrawRequest memory withdrawRequest)

279         function getHash(address account, WithdrawRequest memory withdrawRequest) public view returns (bytes32) {

304         function entryPoint() public pure returns (address) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304:304

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
114         function initialize(bytes[] calldata owners) public payable virtual {


137         function validateUserOp(UserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)


180         function executeWithoutChainIdValidation(bytes calldata data) public payable virtual onlyEntryPoint {


196         function execute(address target, uint256 value, bytes calldata data) public payable virtual onlyEntryPointOrOwner {


205         function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {


217         function entryPoint() public view virtual returns (address) {


229         function getUserOpHashWithoutChainId(UserOperation calldata userOp)


241         function implementation() public view returns (address $) {


252         function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L252:252



## NC022 - Avoid mutating function parameters:

Function parameters in Solidity are passed by value, meaning they are essentially local copies. Mutating them can lead to confusion and errors because the changes don't persist outside the function. By keeping function parameters immutable, you ensure clarity in code behavior, preventing unintended side-effects. If you need to modify a value based on a parameter, use a local variable inside the function, leaving the original parameter unaltered. By adhering to this practice, you maintain a clear distinction between input data and the internal processing logic, improving code readability and reducing the potential for bugs.

note: this insteance missed from bots
```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol
149                 userOpHash = getUserOpHashWithoutChainId(userOp);
```
https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L149:149
