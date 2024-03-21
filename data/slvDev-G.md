Initial gas report was saved with `forge snapshot`

Total gas saved per issue represent `Overall gas change` from `forge snapshot --diff`.

More details from `forge snapshot --diff` can be found in Gas Findings Details.

## Gas Findings

|    | Issue | Instances | Total Gas Saved |
|----|-------|:---------:|:---------:|
| [G-01] | Avoid Comparisons of Boolean Expressions to Boolean Literals | 1 | 60 |
| [G-02] | Using `require()` instead of `assert()` safes gas | 1 | 72 |
| [G-03] | Unlimited gas consumption risk due to external call recipients | 1 | - |
| [G-04] | Use assembly to check for `address(0)` | 2 | 116 |
| [G-05] | Use `assembly` for Efficient Event Emission | 3 | 1140 |
| [G-06] | `internal`/`private` functions only called once can be inlined to save gas | 12 | 240 |
| [G-07] | Using `bool`s for storage incurs overhead | 1 | 100 |
| [G-08] | Declare `immutable` as `private` to save gas | 1 | 3000 |
| [G-09] | Use `revert()` to gain maximum gas savings | 22 | 1100 |
| [G-10] | Using `delete` on a `uint/int` variable is cheaper than assigning it to `0` | 1 | 8 |
| [G-11] | Optimize Gas by Using Only Named Returns | 22 | 968 |
| [G-12] | Enable IR-based code generation | 7 | - |


## Gas Findings Details

### [G-01] Avoid Comparisons of Boolean Expressions to Boolean Literals

Direct comparisons of boolean expressions to boolean literals (true or false) can lead to unnecessary gas costs. 
Instead, use the boolean expression directly or its negation in conditional statements for better gas efficiency.

For instance, replace `if (<x> == true)` with `if (<x>)` and `if (<x> == false)` with `if (!<x>)`.

```bash
test_createAccountSetsOwnersCorrectly() (gas: -12 (-0.004%)) 
test_createAccountDeploysToPredeterminedAddress() (gas: -12 (-0.004%)) 
testDeployDeterministicPassValues() (gas: -12 (-0.004%)) 
test_CreateAccount_ReturnsPredeterminedAddress_WhenAccountAlreadyExists() (gas: -24 (-0.008%)) 
Overall gas change: -60 (-0.000%)
```

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

53: if (alreadyDeployed == false) {
```
[53](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L53)
</details>

### [G-02] Using `require()` instead of `assert()` safes gas

Since `assert()` can't contain any message, it is recommended to use `require()` without message instead.
Using `require()` saves 33 gas per call.
```solidity
    require(false); // 133 gas
    assert(false); // 166 gas 
```

```bash
test_transfersExcess(uint256,uint256,uint256,uint256) (gas: -33 (-0.029%)) 
test_RevertsIfPostOpFailed(uint256,uint256,uint256) (gas: -39 (-0.040%)) 
Overall gas change: -72 (-0.000%)
```


<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

150: assert(mode != PostOpMode.postOpReverted)
```
[150](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L150)
</details>

### [G-03] Unlimited gas consumption risk due to external call recipients

When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted.
To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

273: (bool success, bytes memory result) = target.call{value: value}(data);
```
[273](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L273)
</details>

### [G-04] Use assembly to check for `address(0)`

The usage of inline assembly to check if variable is the zero can save gas compared to traditional `require` or `if` statement checks. 

The assembly check uses the `extcodesize` operation which is generally cheaper in terms of gas.

[More information can be found here.](https://medium.com/@kalexotsu/solidity-assembly-checking-if-an-address-is-0-efficiently-d2bfe071331)

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

121: if (withdrawRequest.asset != address(0)) {
            revert UnsupportedPaymasterAsset(withdrawRequest.asset);
335: if (asset == address(0)) {
            SafeTransferLib.safeTransferETH(to, amount);
```
[121](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L121) | [335](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L335)
</details>

### [G-05] Use `assembly` for Efficient Event Emission

To efficiently emit events, consider utilizing assembly by making use of scratch space and the free memory pointer.
This approach can potentially avoid the costs associated with memory expansion.

However, it's crucial to cache and restore the free memory pointer for safe optimization.
Good examples of such practices can be found in well-optimized [Solady's codebases](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167).
Please review your code and consider the potential gas savings of this approach.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

109: emit RemoveOwner(index, owner)
195: emit AddOwner(index, owner)
```
[109](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L109) | [195](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L195)

```solidity
File: src/MagicSpend/MagicSpend.sol

323: emit MagicSpendWithdrawal(account, withdrawRequest.asset, withdrawRequest.amount, withdrawRequest.nonce)
```
[323](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L323)
</details>

### [G-06] `internal`/`private` functions only called once can be inlined to save gas

`internal` functions that are only called once should be inlined to save gas. 
Not inlining such functions costs an extra 20 to 40 gas due to the additional `JUMP` instructions and stack operations required for function calls.

<details>
<summary><i>12 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

/// @audit function `_checkOwner()` called only once
201: function _checkOwner() internal view virtual {
```
[201](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L201)

```solidity
File: src/SmartWallet/ERC1271.sol

/// @audit function `_eip712Hash()` called only once
121: function _eip712Hash(bytes32 hash) internal view virtual returns (bytes32) {
/// @audit function `_hashStruct()` called only once
133: function _hashStruct(bytes32 hash) internal view virtual returns (bytes32) {
```
[121](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L121) | [133](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L133)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

/// @audit function `_validateSignature()` called only once
291: function _validateSignature(bytes32 message, bytes calldata signature)
        internal
        view
        virtual
        override
        returns (bool)
    {
```
[291](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291)

```solidity
File: src/FreshCryptoLib/FCL.sol

/// @audit function `ecAff_isOnCurve()` called only once
56: function ecAff_isOnCurve(uint256 x, uint256 y) internal pure returns (bool) {
/// @audit function `FCL_nModInv()` called only once
72: function FCL_nModInv(uint256 u) internal view returns (uint256 result) {
/// @audit function `ecZZ_mulmuladd_S_asm()` called only once
95: function ecZZ_mulmuladd_S_asm(
        uint256 Q0,
        uint256 Q1, //affine rep for input point Q
        uint256 scalar_u,
        uint256 scalar_v
    ) internal view returns (uint256 X) {
/// @audit function `ecAff_add()` called only once
252: function ecAff_add(uint256 x0, uint256 y0, uint256 x1, uint256 y1) internal view returns (uint256, uint256) {
/// @audit function `ecZZ_SetAff()` called only once
279: function ecZZ_SetAff(uint256 x, uint256 y, uint256 zz, uint256 zzz)
        internal
        view
        returns (uint256 x1, uint256 y1)
    {
/// @audit function `ecZZ_Dbl()` called only once
296: function ecZZ_Dbl(uint256 x, uint256 y, uint256 zz, uint256 zzz)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
    {
/// @audit function `ecZZ_AddN()` called only once
322: function ecZZ_AddN(uint256 x1, uint256 y1, uint256 zz1, uint256 zzz1, uint256 x2, uint256 y2)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
    {
/// @audit function `FCL_pModInv()` called only once
352: function FCL_pModInv(uint256 u) internal view returns (uint256 result) {
```
[56](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L56) | [72](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L72) | [95](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L95) | [252](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L252) | [279](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L279) | [296](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L296) | [322](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L322) | [352](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L352)
</details>

### [G-07] Using `bool`s for storage incurs overhead

Utilizing booleans for storage is less gas-efficient compared to using types that consume a full word like uint256.
Every write operation on a boolean necessitates an extra SLOAD operation to read the slot's current value, modify the boolean bits, and then write back.
This additional step is the compiler's measure against contract upgrades and pointer aliasing.

To enhance gas efficiency, consider using `uint256(0)` for false and `uint256(1)` for true, bypassing the extra Gwarmaccess (100 gas) incurred by the SLOAD.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

37: mapping(uint256 nonce => mapping(address user => bool used)) internal _nonceUsed;
```
[37](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L37)
</details>

### [G-08] Declare `immutable` as `private` to save gas

Using `private` instead of `public` for immutables saves gas.

The compiler doesn't need to create non-payable getter functions for deployment calldata, store the bytes of the value outside of where it's used, or add another entry to the method ID table, saving 3406-3606 gas in deployment.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

15: address public immutable implementation;
```
[15](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L15)
</details>

### [G-09] Use `revert()` to gain maximum gas savings

If you dont need Error messages, or you want gain maximum gas savings - `revert()` is a cheapest way to revert transaction in terms of gas.
```solidity
    revert(); // 117 gas 
    require(false); // 132 gas
    revert CustomError(); // 157 gas
    assert(false); // 164 gas
    revert("Custom Error"); // 406 gas
    require(false, "Custom Error"); // 421 gas
```


<details>
<summary><i>22 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

104: revert NoOwnerAtIndex(index)
165: revert InvalidOwnerBytesLength(owners[i])
169: revert InvalidEthereumAddressOwner(owners[i])
190: revert AlreadyOwner(owner)
206: revert Unauthorized()
```
[104](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L104) | [165](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L165) | [169](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L169) | [190](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L190) | [206](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L206)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

45: revert OwnerRequired()
```
[45](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L45)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

67: revert Unauthorized()
116: revert Initialized()
151: revert InvalidNonceKey(key)
155: revert InvalidNonceKey(key)
183: revert SelectorNotAllowed(selector)
305: revert InvalidEthereumAddressOwner(ownerBytes)
324: revert InvalidOwnerBytesLength(ownerBytes)
```
[67](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L67) | [116](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L116) | [151](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L151) | [155](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L155) | [183](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L183) | [305](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L305) | [324](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L324)

```solidity
File: src/MagicSpend/MagicSpend.sol

150: assert(mode != PostOpMode.postOpReverted)
94: revert Unauthorized()
118: revert RequestLessThanGasMaxCost(withdrawAmount, maxCost)
122: revert UnsupportedPaymasterAsset(withdrawRequest.asset)
134: revert InsufficientBalance(withdrawAmount, address(this).balance)
172: revert NoExcess()
185: revert InvalidSignature()
189: revert Expired()
317: revert InvalidNonce(withdrawRequest.nonce)
```
[150](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L150) | [94](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L94) | [118](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L118) | [122](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L122) | [134](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L134) | [172](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L172) | [185](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L185) | [189](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L189) | [317](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L317)
</details>

### [G-10] Using `delete` on a `uint/int` variable is cheaper than assigning it to `0`

```solidity
function test() external {
    delete foo; // 5148 gas
    foo = 0; // 5156 gas
}
```

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

116: scalar_v = 0
```
[138](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L13)
</details>


Consider using `receive()` function instead of a specific `deposit()` (or similar) function.
If there are several functions in the contract that can receive Ether, it is recommended to use `receive()` for the most frequently used function.
```solidity
function deposit() external payable { // 5401 gas
    // your logic
}

### [G-16] Optimize Ether Transfers with `receive()` Function
receive() external payable {  // 5356 gas
    // your logic
}
```

The `receive()` or `fallback()` function can handle incoming Ether transfers directly, providing more gas-efficient way to manage deposits.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

212: function entryPointDeposit(uint256 amount) external payable onlyOwner
232: function entryPointAddStake(uint256 amount, uint32 unstakeDelaySeconds) external payable onlyOwner
```
[212](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L212) | [232](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L232)
</details>

### [G-11] Optimize Gas by Using Only Named Returns

The Solidity compiler can generate more efficient bytecode when using named returns.
It's recommended to replace anonymous returns with named returns for potential gas savings.

Example:
```solidity
/// 985 gas cost
function add(uint256 x, uint256 y) public pure returns (uint256) {
    return x + y;
}
/// 941 gas cost
function addNamed(uint256 x, uint256 y) public pure returns (uint256 res) {
    res = x + y;
}
```

<details>
<summary><i>22 issue instances in 6 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

117: function isOwnerAddress(address account) public view virtual returns (bool) {
127: function isOwnerPublicKey(bytes32 x, bytes32 y) public view virtual returns (bool) {
136: function isOwnerBytes(bytes memory account) public view virtual returns (bool) {
145: function ownerAtIndex(uint256 index) public view virtual returns (bytes memory) {
152: function nextOwnerIndex() public view virtual returns (uint256) {
```
[117](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L117) | [127](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L127) | [136](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L136) | [145](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L145) | [152](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L152)

```solidity
File: src/SmartWallet/ERC1271.sol

90: function replaySafeHash(bytes32 hash) public view virtual returns (bytes32) {
100: function domainSeparator() public view returns (bytes32) {
121: function _eip712Hash(bytes32 hash) internal view virtual returns (bytes32) {
133: function _hashStruct(bytes32 hash) internal view virtual returns (bytes32) {
155: function _validateSignature(bytes32 message, bytes calldata signature) internal view virtual returns (bool);
```
[90](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L90) | [100](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L100) | [121](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L121) | [133](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L133) | [155](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L155)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

217: function entryPoint() public view virtual returns (address) {
252: function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
291: function _validateSignature(bytes32 message, bytes calldata signature)
        internal
        view
        virtual
        override
        returns (bool)
    {
333: function _domainNameAndVersion() internal pure override(ERC1271) returns (string memory, string memory) {
```
[217](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L217) | [252](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L252) | [291](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291) | [333](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L333)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

104: function verify(bytes memory challenge, bool requireUV, WebAuthnAuth memory webAuthnAuth, uint256 x, uint256 y)
        internal
        view
        returns (bool)
    {
```
[104](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L104)

```solidity
File: src/FreshCryptoLib/FCL.sol

27: function ecdsa_verify(bytes32 message, uint256 r, uint256 s, uint256 Qx, uint256 Qy) internal view returns (bool) {
56: function ecAff_isOnCurve(uint256 x, uint256 y) internal pure returns (bool) {
251: function ecAff_add(uint256 x0, uint256 y0, uint256 x1, uint256 y1) internal view returns (uint256, uint256) {
```
[27](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L27) | [56](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L56) | [251](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L251)

```solidity
File: src/MagicSpend/MagicSpend.sol

260: function isValidWithdrawSignature(address account, WithdrawRequest memory withdrawRequest)
        public
        view
        returns (bool)
    {
279: function getHash(address account, WithdrawRequest memory withdrawRequest) public view returns (bytes32) {
299: function nonceUsed(address account, uint256 nonce) external view returns (bool) {
304: function entryPoint() public pure returns (address) {
```
[260](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L260) | [279](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L279) | [299](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L299) | [304](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304)
</details>

### [G-12] Enable IR-based code generation

The `--via-ir` command line option activates the IR-based code generator in Solidity, which is designed to enable powerful optimization passes that can span across functions. The end result may be a contract that requires less gas to execute its functions.

We recommend you enable this feature, run tests, and benchmark the gas usage of your contract to evaluate if it leads to any tangible gas savings. Experimenting with this feature could lead to a more gas-efficient contract.

[Solidity Documentation](https://docs.soliditylang.org/en/v0.8.20/ir-breaking-changes.html#solidity-ir-based-codegen-changes).

<details>
<summary><i>7 issue instances in 1 files:</i></summary>

```solidity

File: src/SmartWallet/MultiOwnable.sol
File: src/SmartWallet/ERC1271.sol
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol
File: src/SmartWallet/CoinbaseSmartWallet.sol
File: src/WebAuthnSol/WebAuthn.sol
File: src/FreshCryptoLib/FCL.sol
File: src/MagicSpend/MagicSpend.sol
```
[1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L1) | [1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L1) | [1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L1) | [1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L1) | [1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L1) | [1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L1) | [1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L1)
</details>
