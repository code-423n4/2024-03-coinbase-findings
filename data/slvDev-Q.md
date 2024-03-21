
## Low Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [L-01] | Large transfers may not work with some `ERC20` tokens | 1 |
| [L-02] | Initializers can be front-run | 1 |
| [L-03] | `internal/private` Function calls within for loops | 3 |
| [L-04] | Prefer skip over `revert` model in loops | 2 |
| [L-05] | Unbounded loop may run out of gas | 2 |
| [L-06] | Lack of index element validation in external/public function | 5 |
| [L-07] | Unsafe solidity low-level call can cause gas grief attack | 1 |
| [L-08] | Solidity version `0.8.20` may not work on other chains due to `PUSH0` | 5 |
| [L-09] | `payable` function does not transfer ETH | 2 |
| [L-10] | Unbounded Gas Consumption on External Calls | 1 |


## NonCritical Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [N-01] | Subtraction in `unchecked` block is unsafe | 1 |
| [N-02] | Non-library/interface files should use fixed compiler versions, not floating ones | 3 |
| [N-03] | Array indices should be referenced via `enum`s rather than via numeric literals | 3 |
| [N-04] | Use custom errors instead of `require()/assert()` | 1 |
| [N-05] | Missing Event Emission After Critical Initialize() Function | 1 |
| [N-06] | Consider returning a `struct` rather than having multiple `return` values | 3 |
| [N-07] | Consider using a `struct` rather than having many function input parameters | 3 |
| [N-08] | Consider using descriptive `constant`s when passing zero as a function argument | 1 |
| [N-09] | Avoid Empty Blocks in Code | 1 |
| [N-10] | Custom error has no error details | 7 |
| [N-11] | Consider using OpenZeppelin's SafeCast for any casting | 4 |
| [N-12] | Consider using named returns | 22 |
| [N-13] | Style guide: Initialisms should be capitalized | 1 |
| [N-14] | Consider using named function arguments | 6 |
| [N-15] | Contract/Library Names Not in `CapWords` Style (CamelCase) | 1 |
| [N-16] | Redundant Implementation of Ownable | 1 |
| [N-17] | Events are missing sender information | 1 |
| [N-18] | Non-constant/non-immutable variables using all capital letters | 5 |
| [N-19] | Leverage Recent Solidity Features with `0.8.23` | 5 |
| [N-20] | High cyclomatic complexity | 1 |
| [N-21] | Overridden function has no body | 1 |
| [N-22] | Contract uses both `require()`/`revert()` as well as custom errors | 4 |
| [N-23] | Solidity Version Too Recent to be Trusted | 2 |
| [N-24] | Inconsistent spacing in comments | 68 |
| [N-25] | Unused `error` definition | 1 |
| [N-26] | Outdated Solidity Version | 5 |
| [N-27] | Ensure Non-Empty Check for Bytes in Function Parameters | 7 |
| [N-28] | Adding a `return` statement when the function defines a named return variable, is redundant | 3 |
| [N-29] | Use `bytes.concat()` instead of `abi.encodePacked()` | 2 |
| [N-30] | Function/Constructor Argument Names Not in mixedCase | 2 |
| [N-31] | Upgrade `openzeppelin` to the Latest Version - 5.0.0 | 1 |
| [N-32] | Event is not properly `indexed` | 3 |
| [N-33] | Consider Adding Emergency-Stop Functionality | 4 |
| [N-34] | `constant` variables not using all capital letters | 9 |
| [N-35] | Non-uppercase/Constant-case Naming for `immutable` Variables | 1 |
| [N-36] | Not using the named return variables anywhere in the function is confusing | 4 |
| [N-37] | Consider moving `msg.sender` checks to a common authorization `modifier` | 2 |
| [N-38] | Style guide: Non-`external`/`public` function names should begin with an underscore | 11 |
| [N-38] | Style guide: Contract does not follow the Solidity style guide's suggested layout ordering | 1 |
| [N-40] | Style guide: Control structures do not follow the Solidity Style Guide | 8 |
| [N-41] | Using Low-Level Call for Transfers | 1 |
| [N-42] | Large numeric literals should use underscores for readability | 1 |
| [N-43] | Long functions should be refactored into multiple, smaller, functions | 1 |
| [N-44] | `public` functions not called by the contract should be declared `external` instead | 1 |
| [N-45] | Complex casting | 3 |
| [N-46] | Style guide: Function ordering does not follow the Solidity style guide | 4 |
| [N-47] | Style guide: Lines are too long | 24 |
| [N-48] | Style guide: Extraneous whitespace | 10 |
| [N-49] | Unused import | 1 |
| [N-50] | Use of `override` is unnecessary | 3 |
| [N-51] | Explicit Visibility Recommended in Variable/Function Definitions | 9 |
| [N-52] | Open TODOs | 1 |


## Low Findings Details

### [L-01] Large transfers may not work with some `ERC20` tokens

Some `IERC20` implementations (e.g `UNI`, `COMP`)
            
- [COMP (Compound Protocol)](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/Governance/Comp.sol#L115-L142)
- [UNI (Uniswap)](https://github.com/Uniswap/governance/blob/eabd8c71ad01f61fb54ed6945162021ee419998e/contracts/Uni.sol#L209-L236)
            
may fail if the valued `transferred` is larger than `uint96`.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

338: SafeTransferLib.safeTransfer(asset, to, amount)
```
[338](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L338)
</details>

### [L-02] Initializers can be front-run

Initializers could be vulnerable to front-running attacks.
This might allow an attacker to set their own values, take ownership of the contract, or force a redeployment in the best-case scenario.
Be cautious of potential front-running risks in the following instances found in the code.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

114: function initialize(bytes[] calldata owners) public payable virtual
```
[114](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114)
</details>

### [L-03] `internal/private` Function calls within for loops

Making function calls or external calls within loops in Solidity can lead to inefficient gas usage, potential bottlenecks, and increased vulnerability to attacks.
Each function call or external call consumes gas, and when executed within a loop, the gas cost multiplies, potentially causing the transaction to run out of gas or exceed block gas limits.
This can result in transaction failure or unpredictable behavior.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

172: _addOwnerAtIndex(owners[i], _getMultiOwnableStorage().nextOwnerIndex++)
172: _getMultiOwnableStorage()
```
[172](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L172) | [172](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L172)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

207: _call(calls[i].target, calls[i].value, calls[i].data)
```
[207](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L207)
</details>

### [L-04] Prefer skip over `revert` model in loops

When dealing with array operations in Solidity, it's often wiser to opt for skipping over reverting.
Instead of halting the entire transaction when a condition isn't met, suggests simply moving on to the next array index.
The reason behind this choice is to reduce the risk of malicious actors intentionally introducing array objects that fail conditional checks within loops, causing group operations to fail.
Unless there's a compelling security or logical justification for reverting, it's recommended to embrace the skip-over approach for a more robust codebase.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

165: revert InvalidOwnerBytesLength(owners[i])
169: revert InvalidEthereumAddressOwner(owners[i])
```
[165](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L165) | [169](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L169)
</details>

### [L-05] Unbounded loop may run out of gas

Unbounded loops in smart contracts pose a risk because they iterate over an unknown number of elements, potentially consuming all available gas for a transaction.
This can result in unintended transaction failures. Gas consumption increases linearly with the number of iterations, and if it surpasses the gas limit, the transaction reverts, wasting the gas spent.
To mitigate this, developers should either set a maximum limit on loop iterations.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

163: for (uint256 i; i < owners.length; i++)
```
[163](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L163)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

206: for (uint256 i; i < calls.length;)
```
[206](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L206)
</details>

### [L-06] Lack of index element validation in external/public function

There's no validation to check whether the index element provided as an argument actually exists in the call. This omission could lead to unintended behavior if an element that does not exist in the call is passed to the function.

The function should validate that the provided index element exists in the call before proceeding.

Without this validation, the function could cause unintended behaviour as it will call an non-existing index element. This could lead to inconsistencies in data and potentially affect the integrity of the call structure.

<details>
<summary><i>5 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

107: _getMultiOwnableStorage().ownerAtIndex[index]
137: _getMultiOwnableStorage().isOwner[account]
146: _getMultiOwnableStorage().ownerAtIndex[index]
```
[107](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L107) | [137](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L137) | [146](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L146)

```solidity
File: src/MagicSpend/MagicSpend.sol

300: _nonceUsed[nonce][account]
300: _nonceUsed[nonce]
```
[300](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L300) | [300](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L300)
</details>

### [L-07] Unsafe solidity low-level call can cause gas grief attack

Using the low-level calls of a solidity address can leave the contract open to gas grief attacks.
These attacks occur when the called contract returns a large amount of data. So when calling an external contract, it is necessary to check the length of the return data before reading/copying it (using returndatasize()).

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

273: (bool success, bytes memory result) = target.call{value: value}(data);
```
[273](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L273)
</details>

### [L-08] Solidity version `0.8.20` may not work on other chains due to `PUSH0`

In Solidity `0.8.20`'s compiler, the default target EVM version has been changed to Shanghai. This version introduces a new op code called `PUSH0`.

However, not all Layer 2 solutions have implemented this op code yet, leading to deployment failures on these chains. To overcome this problem, it is recommended to utilize an earlier EVM version.

<details>
<summary><i>5 issue instances in 5 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L2)

```solidity
File: src/SmartWallet/ERC1271.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L2)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L2)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

2: pragma solidity ^0.8.0;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L2)

```solidity
File: src/FreshCryptoLib/FCL.sol

2: pragma solidity >=0.8.19 <0.9.0;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L2)
</details>

### [L-09] `payable` function does not transfer ETH

The following functions can be called by any user, who may also send some funds by mistake.
In that case, those funds will be lost (this also applies to delegatecalls, in case they don't use the transferred ETH).

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

114: function initialize(bytes[] calldata owners) public payable virtual {
137: function validateUserOp(UserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)
        public
        payable
        virtual
        onlyEntryPoint
        payPrefund(missingAccountFunds)
        returns (uint256 validationData)
    {
```
[114](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114) | [137](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L137)
</details>

### [L-10] Unbounded Gas Consumption on External Calls

Consider using `addr.call{gas: <amount>}("")` to set a gas limit and prevent potential reversion due to gas consumption.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

273: (bool success, bytes memory result) = target.call{value: value}(data);
```
[273](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L273)
</details>

## NonCritical Findings Details

### [N-01] Subtraction in `unchecked` block is unsafe

Performing subtraction within an unchecked block poses a risk of silent underflow, leading to unintended behavior or vulnerabilities. Without proper value checks, the subtraction could result in values that are lower than expected, potentially impacting the logic and security of the contract.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

137: scalar_u = addmod(scalar_u, n - scalar_v, n);
```
[137](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L137)
</details>

### [N-02] Non-library/interface files should use fixed compiler versions, not floating ones

Floating pragmas may lead to unintended vulnerabilities due to different compiler versions.
It is recommended to lock the Solidity version in pragma statements.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

2: pragma solidity ^0.8.4
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L2)

```solidity
File: src/SmartWallet/ERC1271.sol

2: pragma solidity ^0.8.4
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L2)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

2: pragma solidity ^0.8.4
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L2)
</details>

### [N-03] Array indices should be referenced via `enum`s rather than via numeric literals

Using constant array indexes can make your Solidity code harder to read and maintain.
To improve clarity, consider using commented enum values in place of constant array indexes.

Enums provide a way to define a type that has a few pre-defined values, making your code more self-explanatory and easy to understand.
This can be particularly helpful in large codebases or when working with a team.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

105: owners[0]
```
[105](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L105)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

133: webAuthnAuth.authenticatorData[32]
138: webAuthnAuth.authenticatorData[32]
```
[133](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L133) | [138](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L138)
</details>

### [N-04] Use custom errors instead of `require()/assert()`

Starting from Solidity 0.8.4, custom errors have been introduced to improve readability and clarity in your code. Instead of using `require()/assert()` with strings, custom errors can provide more expressive and understandable error conditions.

This change not only reduces unnecessary clutter in your codebase but also promotes a more streamlined contract structure, ultimately improving the quality of your Solidity code.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

150: assert(mode != PostOpMode.postOpReverted)
```
[150](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L150)
</details>

### [N-05] Missing Event Emission After Critical Initialize() Function

Emitting an initialization event offers clear, on-chain evidence of the contract's initialization state, enhancing transparency and auditability.
This practice aids users and developers in accurately tracking the contract's lifecycle, pinpointing the precise moment of its initialization.
Moreover, it aligns with best practices for event logging in smart contracts, ensuring that significant state changes are both observable and verifiable through emitted events.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

114: function initialize(bytes[] calldata owners) public payable virtual
```
[114](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L114)
</details>

### [N-06] Consider returning a `struct` rather than having multiple `return` values

Functions that return many variables can become difficult to read and maintain.
Using a struct to encapsulate these return values can improve code readability, increase reusability, and reduce the likelihood of errors.
Consider refactoring functions that return more than three variables to use a struct instead.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/ERC1271.sol

36: function eip712Domain()
        external
        view
        virtual
        returns (
            bytes1 fields,
            string memory name,
            string memory version,
            uint256 chainId,
            address verifyingContract,
            bytes32 salt,
            uint256[] memory extensions
        )
```
[36](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L36)

```solidity
File: src/FreshCryptoLib/FCL.sol

318: function ecZZ_Dbl(uint256 x, uint256 y, uint256 zz, uint256 zzz)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
344: function ecZZ_AddN(uint256 x1, uint256 y1, uint256 zz1, uint256 zzz1, uint256 x2, uint256 y2)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
```
[318](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L318) | [344](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L344)
</details>

### [N-07] Consider using a `struct` rather than having many function input parameters

Functions with many parameters can become difficult to read and maintain. 
Using a struct to encapsulate these parameters can improve code readability, increase reusability, and reduce the likelihood of errors.
Consider refactoring functions that take more than three parameters to use a struct instead.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/WebAuthnSol/WebAuthn.sol

104: function verify(bytes memory challenge, bool requireUV, WebAuthnAuth memory webAuthnAuth, uint256 x, uint256 y)
        internal
        view
        returns (bool)
```
[104](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L104)

```solidity
File: src/FreshCryptoLib/FCL.sol

50: function ecdsa_verify(bytes32 message, uint256 r, uint256 s, uint256 Qx, uint256 Qy) internal view returns (bool)
344: function ecZZ_AddN(uint256 x1, uint256 y1, uint256 zz1, uint256 zzz1, uint256 x2, uint256 y2)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
```
[50](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L50) | [344](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L344)
</details>

### [N-08] Consider using descriptive `constant`s when passing zero as a function argument

In instances where utilizing a zero parameter is essential, it is recommended to employ descriptive constants or an enum instead of directly integrating zero within function calls.
This strategy aids in clearly articulating the caller's intention and minimizes the risk of errors.
Emitting zero also not recomended, as it is not clear what the intention is.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

186: _call(address(this), 0, data)
```
[186](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L186)
</details>

### [N-09] Avoid Empty Blocks in Code

Empty blocks in code can lead to confusion and the introduction of errors when the code is later modified.
They should be removed, or the block should do something useful, such as emitting an event or reverting. 

For contracts meant to be extended, the contract should be abstract and the function signatures added without any default implementation. 

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

106: receive() external payable {}
```
[106](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L106)
</details>

### [N-10] Custom error has no error details

Take advantage of custom error's return value property. 

An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, providing a serious advantage in debugging and examining the revert details of dapps such as Tenderly.

<details>
<summary><i>7 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

40: error Unauthorized()
```
[40](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L40)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

18: error OwnerRequired()
```
[18](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L18)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

46: error Initialized()
```
[46](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L46)

```solidity
File: src/MagicSpend/MagicSpend.sol

53: error InvalidSignature()
56: error Expired()
83: error NoExcess()
90: error UnexpectedPostOpRevertedMode()
```
[53](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L53) | [56](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L56) | [83](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L83) | [90](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L90)
</details>

### [N-11] Consider using OpenZeppelin's SafeCast for any casting

OpenZeppelin's has `SafeCast` library that provides functions to safely cast. Recommended to use it instead of casting directly in any case.

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

168: uint256(bytes32(owners[i]))
```
[168](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L168)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

302: uint256(bytes32(ownerBytes))
```
[302](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L302)

```solidity
File: src/FreshCryptoLib/FCL.sol

61: uint256(message)
```
[61](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L61)

```solidity
File: src/MagicSpend/MagicSpend.sol

128: uint256(withdrawRequest.expiry)
```
[128](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L128)
</details>

### [N-12] Consider using named returns

Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas.
The cases below are where there currently is at most one return statement, which is ideal for named returns.

<details>
<summary><i>22 issue instances in 6 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

117: function isOwnerAddress(address account) public view virtual returns (bool)
127: function isOwnerPublicKey(bytes32 x, bytes32 y) public view virtual returns (bool)
136: function isOwnerBytes(bytes memory account) public view virtual returns (bool)
145: function ownerAtIndex(uint256 index) public view virtual returns (bytes memory)
152: function nextOwnerIndex() public view virtual returns (uint256)
```
[117](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L117) | [127](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L127) | [136](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L136) | [145](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L145) | [152](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L152)

```solidity
File: src/SmartWallet/ERC1271.sol

90: function replaySafeHash(bytes32 hash) public view virtual returns (bytes32)
100: function domainSeparator() public view returns (bytes32)
121: function _eip712Hash(bytes32 hash) internal view virtual returns (bytes32)
133: function _hashStruct(bytes32 hash) internal view virtual returns (bytes32)
155: function _validateSignature(bytes32 message, bytes calldata signature) internal view virtual returns (bool)
```
[90](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L90) | [100](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L100) | [121](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L121) | [133](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L133) | [155](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L155)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

217: function entryPoint() public view virtual returns (address)
252: function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool)
291: function _validateSignature(bytes32 message, bytes calldata signature)
        internal
        view
        virtual
        override
        returns (bool)
333: function _domainNameAndVersion() internal pure override(ERC1271) returns (string memory, string memory)
```
[217](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L217) | [252](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L252) | [291](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291) | [333](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L333)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

104: function verify(bytes memory challenge, bool requireUV, WebAuthnAuth memory webAuthnAuth, uint256 x, uint256 y)
        internal
        view
        returns (bool)
```
[104](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L104)

```solidity
File: src/FreshCryptoLib/FCL.sol

50: function ecdsa_verify(bytes32 message, uint256 r, uint256 s, uint256 Qx, uint256 Qy) internal view returns (bool)
78: function ecAff_isOnCurve(uint256 x, uint256 y) internal pure returns (bool)
274: function ecAff_add(uint256 x0, uint256 y0, uint256 x1, uint256 y1) internal view returns (uint256, uint256)
```
[50](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L50) | [78](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L78) | [274](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L274)

```solidity
File: src/MagicSpend/MagicSpend.sol

260: function isValidWithdrawSignature(address account, WithdrawRequest memory withdrawRequest)
        public
        view
        returns (bool)
279: function getHash(address account, WithdrawRequest memory withdrawRequest) public view returns (bytes32)
299: function nonceUsed(address account, uint256 nonce) external view returns (bool)
304: function entryPoint() public pure returns (address)
```
[260](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L260) | [279](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L279) | [299](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L299) | [304](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L304)
</details>

### [N-13] Style guide: Initialisms should be capitalized

According to the Solidity [style guide](https://docs.soliditylang.org/en/latest/style-guide.html#naming-styles) initialisms such as "NFT", "ERC", etc should be capitalized.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

24: address erc4337
```
[24](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L24)
</details>

### [N-14] Consider using named function arguments

When calling functions in external contracts with multiple arguments, consider using named function parameters, rather than positional ones.

<details>
<summary><i>6 issue instances in 3 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

313: SignatureCheckerLib.isValidSignatureNow(owner, message, sigWrapper.signatureData)
```
[313](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L313)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

116: webAuthnAuth.clientDataJSON.slice(webAuthnAuth.typeIndex, webAuthnAuth.typeIndex + 21)
123: webAuthnAuth.clientDataJSON.slice(
            webAuthnAuth.challengeIndex, webAuthnAuth.challengeIndex + expectedChallenge.length
        )
160: FCL.ecdsa_verify(messageHash, webAuthnAuth.r, webAuthnAuth.s, x, y)
```
[116](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L116) | [123](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L123) | [160](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L160)

```solidity
File: src/MagicSpend/MagicSpend.sol

223: IEntryPoint(entryPoint()).withdrawTo(to, amount)
265: SignatureCheckerLib.isValidSignatureNow(
            owner(), getHash(account, withdrawRequest), withdrawRequest.signature
        )
```
[223](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L223) | [265](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L265)
</details>

### [N-15] Contract/Library Names Not in `CapWords` Style (CamelCase)

Contracts and libraries should be named using the `CapWords` style for better readability and consistency with Solidity style guidelines.
Examples of good practice include: SimpleToken, SmartBank, CertificateHashRepository, Player, Congress, Owned.
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#contract-and-library-names)

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

3: library FCL {
```
[3](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L3)
</details>

### [N-16] Redundant Implementation of Ownable

Detected custom implementation of `Ownable` pattern.
It is recommended to use standardized implementations from libraries like OpenZeppelin or Solady.
These versions have been optimized and usage-hardened, thus re-inventing this is unnecessary and could lead to potential security risks.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

11: import {MultiOwnable} from "./MultiOwnable.sol";
```
[11](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L11)
</details>

### [N-17] Events are missing sender information

Events should include the sender information when emitted in `public` or `external` functions for better traceability.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

/// @audit public function `removeOwnerAtIndex()` emits an event without sender information
108: emit RemoveOwner(index, owner);
```
[108](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L108)
</details>

### [N-18] Non-constant/non-immutable variables using all capital letters

Variable names that consist of all capital letters should be reserved for constant/immutable variables. If the variable needs to be different based on which class it comes from, a view/pure function should be used instead.

<details>
<summary><i>5 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

61: uint256 LHS = mulmod(y, y, p); // y^2
62: uint256 RHS = addmod(mulmod(mulmod(x, x, p), x, p), mulmod(x, a, p), p); // x^3+ax
103: uint256 Y;
105: uint256 H0;
106: uint256 H1;
```
[61](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L61) | [62](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L62) | [103](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L103) | [105](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L105) | [106](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L106)
</details>

### [N-19] Leverage Recent Solidity Features with `0.8.23`

Key enhancements include the use of push0 for placing 0 on the stack for EVM versions starting from "Shanghai", making your code simpler and more straightforward.

Additionally, the re-implementation of the UnusedAssignEliminator and UnusedStoreEliminator in the Solidity optimizer provides the ability to remove unused assignments in deeply nested loops.
This results in a cleaner, more efficient contract code, reducing clutter and potential points of confusion during code review or debugging.
It's recommended to make full use of these features and optimizations to enhance the robustness and readability of your smart contracts.

<details>
<summary><i>5 issue instances in 5 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L2)

[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L2)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L2)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

2: pragma solidity ^0.8.0;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L2)

```solidity
File: src/FreshCryptoLib/FCL.sol

2: pragma solidity >=0.8.19 <0.9.0;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L2)
</details>

### [N-20] High cyclomatic complexity

Functions with high cyclomatic complexity are harder to understand, test, and maintain.
Consider breaking down these blocks into more manageable units, by splitting things into utility functions,
by reducing nesting, and by using early returns.

[Learn More About Cyclomatic Complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/WebAuthnSol/WebAuthn.sol

/// @audit function `verify` has a cyclomatic complexity of 6
104: function verify(bytes memory challenge, bool requireUV, WebAuthnAuth memory webAuthnAuth, uint256 x, uint256 y)
        internal
        view
        returns (bool)
    {
```
[104](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L104)
</details>

### [N-21] Overridden function has no body

Overridden functions with empty bodies.
Consider adding a NatSpec comment explaining why there's no implementation.
This helps future code maintainers understand the design choice.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

330: function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
```
[330](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L330)
</details>

### [N-22] Contract uses both `require()`/`revert()` as well as custom errors

The contract uses both `require()/revert()` and `custom errors` for error handling.

It's recommended to use a consistent approach for error handling in a single file.

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

32: contract MultiOwnable {
```
[32](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L32)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

13: contract CoinbaseSmartWalletFactory {
```
[13](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L13)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

20: contract CoinbaseSmartWallet is MultiOwnable, UUPSUpgradeable, Receiver, ERC1271 {
```
[20](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L20)

```solidity
File: src/MagicSpend/MagicSpend.sol

18: contract MagicSpend is Ownable, IPaymaster {
```
[18](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L18)
</details>

### [N-23] Solidity Version Too Recent to be Trusted

The current Solidity version used in the code is too recent to be trusted.
Using a newer version might introduce unrecovered bugs. It is recommended to use version 0.8.21.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

2: pragma solidity 0.8.23;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L2)

```solidity
File: src/MagicSpend/MagicSpend.sol

2: pragma solidity 0.8.23;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L2)
</details>

### [N-24] Inconsistent spacing in comments

Some lines use // x and some use //x. The instances below point out the usages that don't follow the majority, within each file:

<details>
<summary><i>68 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

5: //*******************************Constants*******************************************************/
8: //curve prime field modulus
10: //short weierstrass first coefficient
12: //short weierstrass second coefficient
14: //generating point affine coordinates
17: //curve order (number of points)
26: //******************Core Function ecdsa_verify********************************************************************/
50: //**************************************************************************************/
52: //***************************Supporting Functions***********************************************************/
97: uint256 Q1, //affine rep for input point Q
113: (H0 == 0) && (H1 == 0) //handling Q=-G
144: let T1 := mulmod(2, Y, p) //U = 2*Y1, y free
148: let T4 := mulmod(3, mulmod(addmod(X, sub(p, zz), p), addmod(X, zz, p), p), p) //M=3*(X1-ZZ1)*(X1+ZZ1)
149: zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
150: zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
152: X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
153: T2 := mulmod(T4, addmod(X, sub(p, T3), p), p) //-M(S-X3)=M(X3-S)
154: Y := addmod(mulmod(T1, Y, p), T2, p) //-Y3= W*Y1-M(S-X3), we replace Y by -Y to avoid a sub in ecAdd
157: //value of dibit
161: Y := sub(p, Y) //restore the -Y inversion
186: //T3:=sub(p, Y)
187: //T3:=Y
188: let y2 := addmod(mulmod(T2, zzz, p), Y, p) //R
189: T2 := addmod(mulmod(T1, zz, p), sub(p, X), p) //P
191: //special extremely rare case accumulator where EcAdd is replaced by EcDbl, no need to optimize this
192: //todo : construct edge vector case
195: T1 := mulmod(minus_2, Y, p) //U = 2*Y1, y free
200: y2 := mulmod(addmod(X, zz, p), addmod(X, sub(p, zz), p), p) //(X-ZZ)(X+ZZ)
201: T4 := mulmod(3, y2, p) //M=3*(X-ZZ)(X+ZZ)
203: zzz := mulmod(T1, zzz, p) //zzz3=W*zzz1
204: zz := mulmod(T2, zz, p) //zz3=V*ZZ1, V free
206: X := addmod(mulmod(T4, T4, p), mulmod(minus_2, T3, p), p) //X3=M^2-2S
207: T2 := mulmod(T4, addmod(T3, sub(p, X), p), p) //M(S-X3)
209: Y := addmod(T2, mulmod(T1, Y, p), p) //Y3= M(S-X3)-W*Y1
215: T4 := mulmod(T2, T2, p) //PP
216: let TT1 := mulmod(T4, T2, p) //PPP, this one could be spared, but adding this register spare gas
218: zzz := mulmod(zzz, TT1, p) //zz3=V*ZZ1
225: } //end loop
228: //(X,Y)=ecZZ_SetAff(X,Y,zz, zzz);
229: //T[0] = inverseModp_Hard(T[0], p); //1/zzz, inline modular inversion using precompile:
235: //mstore(add(pointer, 0x60), u)
242: //Y:=mulmod(Y,zzz,p)//Y/zzz
243: //zz :=mulmod(zz, mload(T),p) //1/z
244: //zz:= mulmod(zz,zz,p) //1/zz
245: X := mulmod(X, mload(T), p) //X/zz
246: } //end assembly
247: } //end unchecked
284: uint256 zzzInv = FCL_pModInv(zzz); //1/zzz
285: y1 = mulmod(y, zzzInv, p); //Y/zzz
286: uint256 _b = mulmod(zz, zzzInv, p); //1/z
287: zzzInv = mulmod(_b, _b, p); //1/zz
288: x1 = mulmod(x, zzzInv, p); //X/zz
303: P0 := mulmod(2, y, p) //U = 2*Y1
307: P2 := mulmod(P2, zz, p) //zz3=V*ZZ1
308: zz := mulmod(3, mulmod(addmod(x, sub(p, zz), p), addmod(x, zz, p), p), p) //M=3*(X1-ZZ1)*(X1+ZZ1)
309: P0 := addmod(mulmod(zz, zz, p), mulmod(minus_2, P3, p), p) //X3=M^2-2S
310: x := mulmod(zz, addmod(P3, sub(p, P0), p), p) //M(S-X3)
311: P3 := mulmod(P1, zzz, p) //zzz3=W*zzz1
312: P1 := addmod(x, sub(p, mulmod(P1, y, p)), p) //Y3= M(S-X3)-W*Y1
336: P0 := mulmod(x2, x2, p) //PP = P^2
337: P1 := mulmod(P0, x2, p) //PPP = P*PP
338: P2 := mulmod(zz1, P0, p) ////ZZ3 = ZZ1*PP
339: P3 := mulmod(zzz1, P1, p) ////ZZZ3 = ZZZ1*PPP
340: zz1 := mulmod(x1, P0, p) //Q = X1*PP
341: P0 := addmod(addmod(mulmod(y2, y2, p), sub(p, P1), p), mulmod(minus_2, zz1, p), p) //R^2-PPP-2*Q
342: P1 := addmod(mulmod(addmod(zz1, sub(p, P0), p), y2, p), mulmod(y1, P1, p), p) //R*(Q-X3)
344: //end assembly
345: } //end unchecked
```
[5](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L5) | [8](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L8) | [10](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L10) | [12](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L12) | [14](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L14) | [17](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L17) | [26](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L26) | [50](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L50) | [52](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L52) | [97](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L97) | [113](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L113) | [144](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L144) | [148](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L148) | [149](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L149) | [150](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L150) | [152](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L152) | [153](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L153) | [154](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L154) | [157](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L157) | [161](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L161) | [186](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L186) | [187](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L187) | [188](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L188) | [189](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L189) | [191](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L191) | [192](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L192) | [195](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L195) | [200](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L200) | [201](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L201) | [203](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L203) | [204](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L204) | [206](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L206) | [207](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L207) | [209](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L209) | [215](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L215) | [216](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L216) | [218](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L218) | [225](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L225) | [228](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L228) | [229](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L229) | [235](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L235) | [242](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L242) | [243](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L243) | [244](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L244) | [245](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L245) | [246](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L246) | [247](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L247) | [284](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L284) | [285](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L285) | [286](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L286) | [287](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L287) | [288](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L288) | [303](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L303) | [307](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L307) | [308](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L308) | [309](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L309) | [310](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L310) | [311](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L311) | [312](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L312) | [336](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L336) | [337](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L337) | [338](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L338) | [339](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L339) | [340](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L340) | [341](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L341) | [342](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L342) | [344](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L344) | [345](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L345)
</details>

### [N-25] Unused `error` definition

Custom errors that aren't utilized within the contract add unnecessary complexity to the codebase. 
Cleaning up and removing these unused custom errors can enhance the readability and maintainability of the contract.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/MagicSpend/MagicSpend.sol

/// @audit Custom Error `UnexpectedPostOpRevertedMode` declared but never used
90: error UnexpectedPostOpRevertedMode();
```
[90](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L90)
</details>

### [N-26] Outdated Solidity Version

The current Solidity version used in the contract is outdated.
Consider using a more recent version for improved features and security.

0.8.4: bytes.concat() instead of abi.encodePacked(,)

0.8.12: string.concat() instead of abi.encodePacked(,)

0.8.13: Ability to use using for with a list of free functions

0.8.14:
    ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases. Override Checker: Allow changing data location for parameters only when overriding external functions.

0.8.15:
    Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays. Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

0.8.16:
    Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

0.8.17:
    Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

<details>
<summary><i>5 issue instances in 5 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L2)

```solidity
File: src/SmartWallet/ERC1271.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L2)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

2: pragma solidity ^0.8.4;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L2)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

2: pragma solidity ^0.8.0;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L2)

```solidity
File: src/FreshCryptoLib/FCL.sol

2: pragma solidity >=0.8.19 <0.9.0;
```
[2](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L2)
</details>

### [N-27] Ensure Non-Empty Check for Bytes in Function Parameters

To avoid mistakenly accepting empty bytes as valid parameters, it is advisable to implement checks for non-empty bytes within functions.
This ensures that empty bytes are not treated as valid inputs, preventing potential issues.

<details>
<summary><i>7 issue instances in 3 files:</i></summary>

```solidity
File: src/SmartWallet/ERC1271.sol

69: function isValidSignature(bytes32 hash, bytes calldata signature) public view virtual returns (bytes4 result) {
69: function isValidSignature(bytes32 hash, bytes calldata signature) public view virtual returns (bytes4 result) {
90: function replaySafeHash(bytes32 hash) public view virtual returns (bytes32) {
```
[69](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L69) | [69](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L69) | [90](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L90)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

180: function executeWithoutChainIdValidation(bytes calldata data) public payable virtual onlyEntryPoint {
196: function execute(address target, uint256 value, bytes calldata data) public payable virtual onlyEntryPointOrOwner {
252: function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
```
[180](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L180) | [196](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L196) | [252](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L252)

```solidity
File: src/MagicSpend/MagicSpend.sol

143: function postOp(IPaymaster.PostOpMode mode, bytes calldata context, uint256 actualGasCost)
        external
        onlyEntryPoint
    {
```
[143](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L143)
</details>

### [N-28] Adding a `return` statement when the function defines a named return variable, is redundant

Functions in Solidity can use named return variables which can enhance readability and reduce the complexity of the code. When using named return variables, a return statement without any parameters is implicitly added to the end of the function.
This means that explicitly writing return in a function that uses named return variables is redundant.
This redundancy could lead to confusion and misinterpretation of the function's behavior, particularly for developers who are new to the Solidity language or to the codebase.

Moreover, explicitly stating the return statement with the named variables might give the impression that the function could have different exit points or that it might return different values, which is not the case here.
Therefore, removing these redundant return statements can lead to cleaner, more maintainable code with reduced cognitive load for developers.
This helps in understanding the flow and the logic of the function at a quick glance and aids in faster debugging and fewer mistakes.

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

/// @audit - Redundant `return X;` when function has named return variables
95: function ecZZ_mulmuladd_S_asm(
        uint256 Q0,
        uint256 Q1, //affine rep for input point Q
        uint256 scalar_u,
        uint256 scalar_v
    ) internal view returns (uint256 X) {
/// @audit - Redundant `return (P0, P1, P2, P3);` when function has named return variables
295: function ecZZ_Dbl(uint256 x, uint256 y, uint256 zz, uint256 zzz)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
    {
/// @audit - Redundant `return (P0, P1, P2, P3);` when function has named return variables
322: function ecZZ_AddN(uint256 x1, uint256 y1, uint256 zz1, uint256 zzz1, uint256 x2, uint256 y2)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
    {
```
[95](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L95) | [295](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L295) | [322](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L322)
</details>

### [N-29] Use `bytes.concat()` instead of `abi.encodePacked()`

Solidity version 0.8.4 introduces `bytes.concat()` for concatenating byte arrays, which is more efficient than using `abi.encodePacked()`.
It is recommended to use `bytes.concat()` instead of `abi.encodePacked()` and upgrade to at least Solidity version 0.8.4 if required.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/ERC1271.sol

122: return keccak256(abi.encodePacked("\x19\x01", domainSeparator(), _hashStruct(hash)));
```
[122](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L122)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

148: bytes32 messageHash = sha256(abi.encodePacked(webAuthnAuth.authenticatorData, clientDataJSONHash));
```
[148](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L148)
</details>

### [N-30] Function/Constructor Argument Names Not in mixedCase

Underscore before of after function argument names is a common convention in Solidity NOT a documentation requirement.

Function arguments should use mixedCase for better readability and consistency with Solidity style guidelines. 
Examples of good practice include: initialSupply, account, recipientAddress, senderAddress, newOwner. 
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-argument-names)

Rule exceptions
- Allow constant variable name/symbol/decimals to be lowercase (ERC20).
- Allow `_` at the beginning of the mixedCase match for `private variables` and `unused parameters`.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

95: function ecZZ_mulmuladd_S_asm(
        uint256 Q0,
        uint256 Q1, //affine rep for input point Q
        uint256 scalar_u,
        uint256 scalar_v
    ) internal view returns (uint256 X) {
```
[95](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L95)

```solidity
File: src/MagicSpend/MagicSpend.sol

101: constructor(address _owner) {
```
[101](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L101)
</details>

### [N-31] Upgrade `openzeppelin` to the Latest Version - 5.0.0

These contracts import contracts from @openzeppelin/contracts but they are not using the latest version.
For more information, please visit: [OpenZeppelin GitHub Releases](https://github.com/OpenZeppelin/openzeppelin-contracts/releases)
It is recommended to always use the latest version to take advantage of updates and security fixes.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/WebAuthnSol/WebAuthn.sol

5: import {Base64} from "openzeppelin-contracts/contracts/utils/Base64.sol";
```
[5](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L5)
</details>

### [N-32] Event is not properly `indexed`

Index event fields make the field more quickly accessible to off-chain tools that parse events.
This is especially useful when it comes to filtering based on an address. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields).
Where applicable, each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question.
If there are fewer than three applicable fields, all of the applicable fields should be indexed.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

68: event AddOwner(uint256 indexed index, bytes owner);
74: event RemoveOwner(uint256 indexed index, bytes owner);
```
[68](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L68) | [74](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L74)

```solidity
File: src/MagicSpend/MagicSpend.sol

45: event MagicSpendWithdrawal(address indexed account, address indexed asset, uint256 amount, uint256 nonce);
```
[45](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L45)
</details>

### [N-33] Consider Adding Emergency-Stop Functionality

Smart contracts that hold significant value, interact with external contracts, or have complex logic should include an emergency-stop mechanism for added security. This allows pausing certain contract functionalities in case of emergencies, mitigating potential damages.

This contract seems to lack such a mechanism. Implementing an emergency stop can enhance contract security and reliability.

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

1: No emergency stop pattern found
```
[1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L1)

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

1: No emergency stop pattern found
```
[1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L1)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

1: No emergency stop pattern found
```
[1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L1)

```solidity
File: src/MagicSpend/MagicSpend.sol

1: No emergency stop pattern found
```
[1](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L1)
</details>

### [N-34] `constant` variables not using all capital letters

Variable names for `constant` variables should consist of all capital letters.

<details>
<summary><i>9 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

9: uint256 constant p = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFF;
11: uint256 constant a = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFC;
13: uint256 constant b = 0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B;
15: uint256 constant gx = 0x6B17D1F2E12C4247F8BCE6E563A440F277037D812DEB33A0F4A13945D898C296;
16: uint256 constant gy = 0x4FE342E2FE1A7F9B8EE7EB4A7C0F9E162BCE33576B315ECECBB6406837BF51F5;
18: uint256 constant n = 0xFFFFFFFF00000000FFFFFFFFFFFFFFFFBCE6FAADA7179E84F3B9CAC2FC632551;
20: uint256 constant minus_2 = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFD;
22: uint256 constant minus_2modn = 0xFFFFFFFF00000000FFFFFFFFFFFFFFFFBCE6FAADA7179E84F3B9CAC2FC63254F;
24: uint256 constant minus_1 = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF;
```
[9](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L9) | [11](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L11) | [13](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L13) | [15](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L15) | [16](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L16) | [18](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L18) | [20](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L20) | [22](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L22) | [24](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L24)
</details>

### [N-35] Non-uppercase/Constant-case Naming for `immutable` Variables

For better readability and adherence to common naming conventions, variable names declared as immutable should be written in all uppercase letters.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

15: address public immutable implementation;
```
[15](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L15)
</details>

### [N-36] Not using the named return variables anywhere in the function is confusing

Declaring a named return variable but returning a different value within the function is an inconsistent use of named return variables.
This practice can lead to confusion and misinterpretation of the function's intended behavior, as it contradicts the purpose of declaring named return variables, which is to enhance readability and reduce complexity in the code.

This inconsistency might signal an error in the function logic where the declared return variable is not being used as intended.
It is recommended to revisit the function's logic to ensure that the named return variables are being utilized correctly, or to adjust the return statements to reflect the actual values being returned, fostering better clarity and coherence in the code.

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: src/SmartWallet/ERC1271.sol

/// @audit - `return 0xffffffff;` statement in functions but `result` are declared
69: function isValidSignature(bytes32 hash, bytes calldata signature) public view virtual returns (bytes4 result) {
```
[69](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L69)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

/// @audit - `return 0;` statement in functions but `validationData` are declared
/// @audit - `return 1;` statement in functions but `validationData` are declared
137: function validateUserOp(UserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)
        public
        payable
        virtual
        onlyEntryPoint
        payPrefund(missingAccountFunds)
        returns (uint256 validationData)
    {
/// @audit - `return keccak256(abi.encode(UserOperationLib.hash(userOp), entryPoint()));` statement in functions but `userOpHash` are declared
229: function getUserOpHashWithoutChainId(UserOperation calldata userOp)
        public
        view
        virtual
        returns (bytes32 userOpHash)
    {
```
[137](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L137) | [229](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L229)

```solidity
File: src/FreshCryptoLib/FCL.sol

/// @audit - `if (scalar_u == 0 && scalar_v == 0) return 0;` statement in functions but `X` are declared
/// @audit - `return (y == 0);` statement in functions but `flag` are declared
271: function ecAff_IsZero(uint256, uint256 y) internal pure returns (bool flag) {
/// @audit - `return (x2, y2, 1, 1);` statement in functions but `P0, P1, P2, P3` are declared
```
[271](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L271)
</details>

### [N-37] Consider moving `msg.sender` checks to a common authorization `modifier`

Functions that are only allowed to be called by a specific actor should use a modifier to check if the caller is the specified actor (e.g., owner, specific role, etc.).
Using `require` to check `msg.sender` in the function body is less efficient and less clear.

Consider refactoring these `require` statements into a modifier for better readability, organization, and gas efficiency.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

202: if (isOwnerAddress(msg.sender) || (msg.sender == address(this))) {
```
[202](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L202)

```solidity
File: src/MagicSpend/MagicSpend.sol

183: if (!isValidWithdrawSignature(msg.sender, withdrawRequest)) {
```
[183](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L183)
</details>

### [N-38] Style guide: Non-`external`/`public` function names should begin with an underscore

In Solidity, it is suggested to use a leading underscore for non-public (private and internal) function names.
This convention helps to distinguish clearly between public/external and non-public functions, improving code clarity.
By adhering to this convention, developers can mitigate potential misinterpretation or errors, thus making the code more comprehensible and easier to maintain.

<details>
<summary><i>11 issue instances in 2 files:</i></summary>

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
72: function FCL_nModInv(uint256 u) internal view returns (uint256 result) {
95: function ecZZ_mulmuladd_S_asm(
        uint256 Q0,
        uint256 Q1, //affine rep for input point Q
        uint256 scalar_u,
        uint256 scalar_v
    ) internal view returns (uint256 X) {
251: function ecAff_add(uint256 x0, uint256 y0, uint256 x1, uint256 y1) internal view returns (uint256, uint256) {
271: function ecAff_IsZero(uint256, uint256 y) internal pure returns (bool flag) {
279: function ecZZ_SetAff(uint256 x, uint256 y, uint256 zz, uint256 zzz)
        internal
        view
        returns (uint256 x1, uint256 y1)
    {
295: function ecZZ_Dbl(uint256 x, uint256 y, uint256 zz, uint256 zzz)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
    {
322: function ecZZ_AddN(uint256 x1, uint256 y1, uint256 zz1, uint256 zzz1, uint256 x2, uint256 y2)
        internal
        pure
        returns (uint256 P0, uint256 P1, uint256 P2, uint256 P3)
    {
352: function FCL_pModInv(uint256 u) internal view returns (uint256 result) {
```
[27](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L27) | [56](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L56) | [72](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L72) | [95](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L95) | [251](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L251) | [271](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L271) | [279](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L279) | [295](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L295) | [322](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L322) | [352](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L352)
</details>

### [N-39] Style guide: Contract does not follow the Solidity style guide's suggested layout ordering

Adhering to a recommended order in Solidity contracts enhances code readability and maintenance.
[More information in Documentation](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout)
It's recommended to use the following order:
1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

/// @audit `event` declared after `error`
68: event AddOwner(uint256 indexed index, bytes owner);
```
[68](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L68)
</details>

### [N-40] Style guide: Control structures do not follow the Solidity Style Guide

Following best practices for control structures in solidity code is vital for readability and maintainability. The control structures in contracts, libraries, functions, and structs should adhere to the following standards:

- Braces denoting the body should open on the same line as the declaration and close on their own line at the same indentation level as the beginning of the declaration. 
- A single space should precede the opening brace. 
- Control structures such as 'if', 'else', 'while', and 'for' should also follow these spacing and brace placement recommendations.

It is advised to revisit the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) sections in documentation to ensure conformity with these best practices, fostering cleaner and more maintainable code.

<details>
<summary><i>8 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

/// @audit `Return or revert statement should be on new line.`
104: if (owner.length == 0) revert NoOwnerAtIndex(index);
/// @audit `Return or revert statement should be on new line.`
190: if (isOwnerBytes(owner)) revert AlreadyOwner(owner);
```
[104](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L104) | [190](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L190)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

/// @audit `Return or revert statement should be on new line.`
158: if (success && valid) return abi.decode(ret, (uint256)) == 1;
```
[158](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L158)

```solidity
File: src/FreshCryptoLib/FCL.sol

/// @audit `Return or revert statement should be on new line.`
109: if (scalar_u == 0 && scalar_v == 0) return 0;
/// @audit `Return or revert statement should be on new line.`
255: if (ecAff_IsZero(x0, y0)) return (x1, y1);
/// @audit `Return or revert statement should be on new line.`
257: if (ecAff_IsZero(x1, y1)) return (x0, y0);
```
[109](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L109) | [255](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L255) | [257](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L257)

```solidity
File: src/MagicSpend/MagicSpend.sol

/// @audit `Return or revert statement should be on new line.`
94: if (msg.sender != entryPoint()) revert Unauthorized();
/// @audit `Return or revert statement should be on new line.`
172: if (amount == 0) revert NoExcess();
```
[94](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L94) | [172](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L172)
</details>

### [N-41] Using Low-Level Call for Transfers

Directly using low-level calls for Ether transfers can introduce vulnerabilities and obscure the intent of your transfers.
Adopt modern Solidity best practices by switching to recognized libraries like `SafeTransferLib.safeTransferETH` or `Address.sendValue`.
This ensures safer transactions, enhances code clarity, and aligns with the standards of the Solidity community.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

273: (bool success, bytes memory result) = target.call{value: value}(data);
```
[273](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L273)
</details>

### [N-42] Large numeric literals should use underscores for readability

Large numeric literals are often hard to read and prone to mistakes.
To improve readability and reduce the likelihood of errors, it is recommended to separate the digits of large numeric literals using underscores.
For example, instead of '1000000', use '1_000_000'.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

43: uint256 public constant REPLAYABLE_NONCE_KEY = 8453;
```
[43](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L43)
</details>

### [N-43] Long functions should be refactored into multiple, smaller, functions

Functions that span many lines can be hard to understand and maintain.
It is often beneficial to refactor long functions into multiple smaller functions.
This improves readability, makes testing easier, and can even lead to gas optimization if it eliminates the need for variables that would otherwise have to be stored in memory.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

 /// @audit 112 lines (excluding comments)
95: function ecZZ_mulmuladd_S_asm(
        uint256 Q0,
        uint256 Q1, //affine rep for input point Q
        uint256 scalar_u,
        uint256 scalar_v
    ) internal view returns (uint256 X) {
```
[95](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L95)
</details>

### [N-44] `public` functions not called by the contract should be declared `external` instead

Contracts are allowed to override their parents' functions and change the visibility from `external` to `public`.
If a `public` function is not called internally within the contract, it should be declared as `external` to save gas.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

241: function implementation() public view returns (address $) {
```
[241](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L241)
</details>

### [N-45] Complex casting

Complex casting in Solidity contracts can introduce both readability issues and potential for overflows. 
Whenever multiple casts are found, consider whether the complexity is necessary.
If so, it is strongly recommended to add inline comments to explain why these casts are needed and to assure that no overflows are introduced.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

34: ///      Computed from: keccak256(abi.encode(uint256(keccak256("coinbase.storage.MultiOwnable")) - 1)) & ~bytes32(uint256(0xff))
168: if (owners[i].length == 32 && uint256(bytes32(owners[i])) > type(uint160).max) {
```
[34](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L34) | [168](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L168)

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

302: if (uint256(bytes32(ownerBytes)) > type(uint160).max) {
```
[302](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L302)
</details>

### [N-46] Style guide: Function ordering does not follow the Solidity style guide

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.
But there are contracts in the project that do not comply with this.
Functions should be grouped according to their visibility and ordered:
- constructor 
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private.

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWalletFactory.sol

/// @audit `public` function `createAccount()` declared before `external` function `getAddress()`
38: function createAccount(bytes[] calldata owners, uint256 nonce)
        public
        payable
        virtual
        returns (CoinbaseSmartWallet account)
    {
64: function getAddress(bytes[] calldata owners, uint256 nonce) external view returns (address predicted) {
```
[38](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L38) | [64](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWalletFactory.sol#L64)

```solidity
File: src/MagicSpend/MagicSpend.sol

/// @audit `public` function `getHash()` declared before `external` function `nonceUsed()`
279: function getHash(address account, WithdrawRequest memory withdrawRequest) public view returns (bytes32) {
299: function nonceUsed(address account, uint256 nonce) external view returns (bool) {
```
[279](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L279) | [299](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/MagicSpend.sol#L299)
</details>

### [N-47] Style guide: Lines are too long

It is generally recommended that lines in the source code should not exceed 80-120 characters.
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

<details>
<summary><i>24 issue instances in 4 files:</i></summary>

```solidity
File: src/SmartWallet/MultiOwnable.sol

34: ///      Computed from: keccak256(abi.encode(uint256(keccak256("coinbase.storage.MultiOwnable")) - 1)) & ~bytes32(uint256(0xff))
```
[34](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/MultiOwnable.sol#L34)

```solidity
File: src/SmartWallet/ERC1271.sol

115: /// @dev Implements encode(domainSeparator : 𝔹²⁵⁶, message : 𝕊) = "\x19\x01" || domainSeparator || hashStruct(message).
125: /// @notice Returns the EIP-712 `hashStruct` result of the `CoinbaseSmartWalletMessage(bytes32 hash)` data structure.
```
[115](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L115) | [125](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/ERC1271.sol#L125)

```solidity
File: src/WebAuthnSol/WebAuthn.sol

66: ///           a well-formed assertion with the user present bit set. If `requireUV` is set, checks that the authenticator
67: ///           enforced user verification. User verification should be required if, and only if, options.userVerification
69: ///         - Verifies that the client JSON is of type "webauthn.get", i.e. the client was responding to a request to
72: ///         - Verifies that (r, s) constitute a valid signature over both the authenicatorData and client JSON, for public
76: ///         - Does NOT verify that the origin in the `clientDataJSON` matches the Relying Party's origin: tt is considered
78: ///           enforced by most high quality authenticators properly, particularly the iCloud Keychain and Google Password
80: ///         - Does NOT verify That `topOrigin` in `clientDataJSON` is well-formed: We assume it would never be present, i.e.
81: ///           the credentials are never used in a cross-origin/iframe context. The website/app set up should disallow
82: ///           cross-origin usage of the credentials. This is the default behaviour for created credentials in common settings.
83: ///         - Does NOT verify that the `rpIdHash` in `authenticatorData` is the SHA-256 hash of the RP ID expected by the Relying
84: ///           Party: this means that we rely on the authenticator to properly enforce credentials to be used only by the correct RP.
85: ///           This is generally enforced with features like Apple App Site Association and Google Asset Links. To protect from
86: ///           edge cases in which a previously-linked RP ID is removed from the authorised RP IDs, we recommend that messages
88: ///         - Does NOT verify the credential backup state: this assumes the credential backup state is NOT used as part of Relying
90: ///         - Does NOT verify the values of the client extension outputs: this assumes that the Relying Party does not use client
92: ///         - Does NOT verify the signature counter: signature counters are intended to enable risk scoring for the Relying Party.
94: ///         - Does NOT verify the attestation object: this assumes that response.attestationObject is NOT present in the response,
137: // 17. If user verification is required for this assertion, verify that the User Verified bit of the flags in authData is set.
147: // 20. Using credentialPublicKey, verify that sig is a valid signature over the binary concatenation of authData and hash.
```
[66](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L66) | [67](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L67) | [69](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L69) | [72](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L72) | [76](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L76) | [78](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L78) | [80](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L80) | [81](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L81) | [82](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L82) | [83](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L83) | [84](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L84) | [85](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L85) | [86](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L86) | [88](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L88) | [90](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L90) | [92](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L92) | [94](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L94) | [137](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L137) | [147](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L147)

```solidity
File: src/FreshCryptoLib/FCL.sol

148: let T4 := mulmod(3, mulmod(addmod(X, sub(p, zz), p), addmod(X, zz, p), p), p) //M=3*(X1-ZZ1)*(X1+ZZ1)
191: //special extremely rare case accumulator where EcAdd is replaced by EcDbl, no need to optimize this
```
[148](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L148) | [191](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L191)
</details>

### [N-48] Style guide: Extraneous whitespace

See the [whitespace](https://docs.soliditylang.org/en/v0.8.24/style-guide.html#whitespace-in-expressions) section of the Solidity Style Guide

<details>
<summary><i>10 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
85: if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
119: for { let T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1)) } eq(T4, 0) {
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
142: for {} gt(minus_1, index) { index := sub(index, 1) } {
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
240: if iszero(staticcall(not(0), 0x05, T, 0xc0, T, 0x20)) { revert(0, 0) }
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
365: if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
85: if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
119: for { let T4 := add(shl(1, and(shr(index, scalar_v), 1)), and(shr(index, scalar_u), 1)) } eq(T4, 0) {
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
142: for {} gt(minus_1, index) { index := sub(index, 1) } {
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
240: if iszero(staticcall(not(0), 0x05, T, 0xc0, T, 0x20)) { revert(0, 0) }
/// @audit `Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.`
365: if iszero(staticcall(not(0), 0x05, pointer, 0xc0, pointer, 0x20)) { revert(0, 0) }
```
[85](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L85) | [119](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L119) | [142](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L142) | [240](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L240) | [365](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L365) | [85](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L85) | [119](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L119) | [142](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L142) | [240](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L240) | [365](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L365)
</details>

### [N-49] Unused import

The contract contains import statements for libraries or other contracts that are not utilized within the code.
Excessive or unused imports can clutter the codebase, leading to inefficiency and potential confusion.
Consider removing any imports that are not essential to the contract's functionality.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/WebAuthnSol/WebAuthn.sol

/// @audit imported `Base64` not used)
5: import {Base64} from "openzeppelin-contracts/contracts/utils/Base64.sol";
```
[5](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/WebAuthnSol/WebAuthn.sol#L5)
</details>

### [N-50] Use of `override` is unnecessary

In Solidity version 0.8.8 and later, the use of the `override` keyword becomes superfluous when a function is overriding solely from an interface and the function isn't present in multiple base contracts.
Previously, the `override` keyword was required as an explicit indication to the compiler. However, this is no longer the case, and the extraneous use of the keyword can make the code less clean and more verbose.

Solidity documentation on [Function Overriding](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding).

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: src/SmartWallet/CoinbaseSmartWallet.sol

291: function _validateSignature(bytes32 message, bytes calldata signature)
        internal
        view
        virtual
        override
        returns (bool)
    {
330: function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
333: function _domainNameAndVersion() internal pure override(ERC1271) returns (string memory, string memory) {
```
[291](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L291) | [330](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L330) | [333](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/SmartWallet/CoinbaseSmartWallet.sol#L333)
</details>

### [N-51] Explicit Visibility Recommended in Variable/Function Definitions

In Solidity, variable/function visibility is crucial for controlling access and protecting against unwanted modifications. 
While Solidity functions default to `internal` visibility, it is best practice to explicitly state the visibility for better code readability and avoiding confusion.

The missing visibility could lead to a false sense of security in contract design and potential vulnerabilities.

<details>
<summary><i>9 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

9: uint256 constant p = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFF;
11: uint256 constant a = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFC;
13: uint256 constant b = 0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B;
15: uint256 constant gx = 0x6B17D1F2E12C4247F8BCE6E563A440F277037D812DEB33A0F4A13945D898C296;
16: uint256 constant gy = 0x4FE342E2FE1A7F9B8EE7EB4A7C0F9E162BCE33576B315ECECBB6406837BF51F5;
18: uint256 constant n = 0xFFFFFFFF00000000FFFFFFFFFFFFFFFFBCE6FAADA7179E84F3B9CAC2FC632551;
20: uint256 constant minus_2 = 0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFD;
22: uint256 constant minus_2modn = 0xFFFFFFFF00000000FFFFFFFFFFFFFFFFBCE6FAADA7179E84F3B9CAC2FC63254F;
24: uint256 constant minus_1 = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF;
```
[9](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L9) | [11](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L11) | [13](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L13) | [15](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L15) | [16](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L16) | [18](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L18) | [20](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L20) | [22](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L22) | [24](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L24)
</details>

### [N-52] Open TODOs

An open TODO is present in the code. It is recommended to avoid open TODOs
as they may indicate programming errors that still need to be fixed.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/FreshCryptoLib/FCL.sol

192: //todo : construct edge vector case
```
[192](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/FreshCryptoLib/FCL.sol#L192)
</details>
