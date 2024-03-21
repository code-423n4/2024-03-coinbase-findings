![logo](https://i.imgur.com/rPuzEya.png)

# **🛠️ Analysis -** Coinbase

## Overview

Coinbase Smart Wallet contracts are focused mainly on `ERC4337` and making it compliant with the Coinbase infrastructure. With his custom `Paymaster` implementation, the flow where sponsoring transactions from the executor’s off-chain wallet is already possible. The end goal of the product is to completely remove the need for EOAs, thus attracting non-technical users without exposing them to the underlying complexity. `WebAuthnSol` is a crucial contract validating the signatures provided off-chain by passkey owners (passwords, biometrics, and PINs).

**Key contracts of Coinbase Smart Wallet for this audit are:**

- `CoinbaseSmartWallet`: ERC4337-compliant smart wallet supporting multiple owners and two types of signatures
- `CoinbaseSmartWalletFactory`: ERC4337-compliant smart wallet factory, used to deploy `CoinbaseSmartWallet` deterministically using the UUPS pattern.
- `WebAuthn`: Library for verifying WebAuthn Authentication assertions (off-chain chain biometric data validation).
- `MagicSpend`: ERC4337-compliant Paymaster contract, used to sponsor user transactions from `Coinbase` wallet.

## **System Overview**

We can observe 3 main parts in the Coinbase Smart Wallet system:

- `CoinbaseSmartWallet` - Constructed from:
    - `MultiOwnable` - Contract that makes wallet to support multiple owners and their management, also ensures that storage is consistent throughout the upgrades, follows `ERC-7201`.
    - `UUPSUpgradeable` -  `Solady` implementation contract makes it possible to deploy many instances in a gas-efficient manner and upgrade the deployable implementation when needed.
    - `Receiver` - `Solady` fallback extension to support token transfers with callbacks (ERC721, ERC1155).
    - `ERC1271` - Signature validation contract, defining the domain and version of the contract to prevent signature replays.

Is a smart wallet implementation with added functionality to have repayable admin transactions across different chains.

- `CoinbaseSmartWalletFactory` - Standard factory contract that can precompute address and deploy them through `CREATE2` when needed.
- `MagicSpend` - Paymaster implementation to sponsor smart wallet transactions and allowing users to deposit and withdraw from `EntryPoint` .
- `WebAuthn` - Biometric validation contract, used to verify and backup through various options (fingerprints, Face ID, password).

## Approach Taken in Evaluating WiseLending

| Stage | Action | Details | Information |
| --- | --- | --- | --- |
| 1 | Compile and Run Test | [Installation](https://github.com/code-423n4/2024-03-coinbase?tab=readme-ov-file#tests) | Easy setup, standard test execution, |
| 2 | Documentation review | [Documentation](https://github.com/code-423n4/2024-03-coinbase/blob/main/src/MagicSpend/README.md) | Provides general use-case details as well as structures used and most important functions. |
| 3 | Contest details | [Audit Details](https://github.com/code-423n4/2024-03-coinbase?tab=readme-ov-file#smart-wallet-audit-details) | Contains simple contract explanations, and external helper links. Gives additional context and potential attack ideas. |
| 4 | Diagramming | Excalidraw | Drawing diagrams through the entire process of codebase evaluation. |
| 5 | Test Suits | [Tests](https://github.com/code-423n4/2024-03-coinbase?tab=readme-ov-file#tests) | Consisting mainly of happy path tests, without going beyond the code and testing complex scenarios. |
| 6 | Manual Code Review | [Scope](https://github.com/code-423n4/2024-03-coinbase?tab=readme-ov-file#scope) | Reviewed all the contracts in scope. |
| 7 | Special focus on Areas of Concern | [Areas of concern](https://github.com/code-423n4/2024-03-coinbase/blob/main/bot-report.md) | Observing the known issues and bot report.  |

## Architecture Recommendations

### CoinbaseSmartWallet:

Developers did a great job, maximally sticking to the EIP4337, without introducing unnecessary complexity. Additions, such as multi-owners and signatures without `chainId` are well implemented, using `assembly` where possible. `EntryPoint` address is hardcoded, but this does not pose any problem since the wallet is upgradeable.

All access modifiers are applied properly, owner management is as described. Signatures cannot be replayed, containing all the crucial information hashed, only exception is for admins.

Notes for improvement:

- Replace setting owners at 1st index to `address(0)` with `disableInitializer()` in the constructor as it is more widely used:

```solidity
constructor() {
      // Implementation should not be initializable (does not affect proxies that use their storage).
      bytes[] memory owners = new bytes[](1);
      owners[0] = abi.encode(address(0));
      _initializeOwners(owners); //disableInitializer()
  }
```

- Fix typos:

```solidity
bytes32 private constant MUTLI_OWNABLE_STORAGE_LOCATION //MULTI
```

- Some functions can be simplified, such as `canSkipChainIdValidation` to return the result without explicitly use true/false.
- `MultiOwner` can be made abstract since there is no purpose to be deployed separately.
- Functions to [deposit and withdraw](https://github.com/eth-infinitism/account-abstraction/blob/674b1f51164e641a18d5c141895bab9c96a68e9d/contracts/samples/SimpleAccount.sol#L118-L138) from `EntryPoint` can be added instead of relying on users to call them on behalf of the smart wallet.
- `executeBatch` is not capped to reasonable amount of transactions potentially leading to out of gas exceptions.

### CoinbaseSmartWalletFactory:

Standard implementation of ERC4337 factory, has all the needed functions to precompute and deploy the wallet when needed, as well as support just in time ETH transfers for new wallets.

Salt creation can be more specific as now same nonce and different owners combination will deploy new wallet every time, eventually misleading non-technical users.

Notes for improvement:

- Owners array can be sorted to prevent deployment of duplicate wallets.

### MagicSpend:

Simple paymaster implementation, contains all the needed functions to operate effectively and integrate with `EntryPoint`, paying the smart wallet transactions.

Notes for improvement:

- Remove `asset` from the `WithdrawRequest`, since only ETH is supported as payment token for now.

### WebAuthn:

1:1 with Daimo’s implementation, with some modified steps. Following the official `w3` [documentation](https://www.w3.org/TR/webauthn-2/#sctn-verifying-assertion).

Notes for improvement:

- In `verify()` remove step 17 as the `requireUV` is always passed as false and no need to check for the user verification flag.

```solidity
// 17. If user verification is required for this assertion, verify that the User Verified bit of the flags in authData is set.
if (requireUV && (webAuthnAuth.authenticatorData[32] & AUTH_DATA_FLAGS_UV) != AUTH_DATA_FLAGS_UV) {
    return false;
}
```

## Codebase Quality Analysis

We consider the quality of `Coinbase Smart Wallet` codebase as a exceptional, without introducing unnecessary complexity and error prone code.

Contracts strictly follow the desired standards (4337, 1271, 712).

Unique feature represented by `WebAuthn` and `FCL`, used to solve the problem of the signatures is hard to be understood but on the other hand, there is no way to be changed due to the increased gas cost.

Tests are not well developed, with validating mainly the happy path and not testing for use cases that might cause harm to the contracts in scope. They are modular with a good structure and setup, but due to not utilizing `CoinbaseSmartWalletFactory` in `SmartWalletTestBase` to deploy accounts, tests for UUPS upgrades were failing, because of using a concrete implementation instead of a proxy.

| Codebase Quality Categories | Comments |
| --- | --- |
| Code Maintainability and Reliability | The Coinbase Smart Wallet contracts demonstrate good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, and conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space. |
| Code Comments | Comments provided high-level context for certain constructs, lower-level logic flows and variables explained within the code itself. As an auditor, it is easier to analyze code sections with the provided information.  |
| Documentation | Technical documentation is provided on the Coinbase Smart Wallet page and in their Github and a decent amount of attack ideas, as well as, main invariants are on the contest page. |
| Code Structure and Formatting | The codebase contracts are well-structured and formatted. Their consist of consistent comments and notes making the review easier. |

## Test analysis

The overall test coverage is high, most of them covering only happy path scenarios. Not all the functions were tested. For example `upgradeAndCall` called from `EntryPoint` contract was missing.

It was failing because `onlyProxy` modifier was reverting since the `account` was not deployed through factory, test was indicating passed but when the verbosity was increased it appeared. 

Consider updating the `setUp` function in `SmartWalletTestBase` in order to properly test the functionality and avoid scenarios where concrete implementations work, but proxies don’t.

```solidity
function setUp() public virtual {
    CoinbaseSmartWalletFactory factory = new CoinbaseSmartWalletFactory(address(new CoinbaseSmartWallet()));
    owners.push(abi.encode(signer));
    owners.push(passkeyOwner);
    account = factory.createAccount(owners, 1);
}
```

It will be beneficial for the project if end-to-end tests are introduced validating example use cases of the wallet-paymaster integration.

## **Systemic & Centralization Risks**

Regarding the centralization risks involved in the `CoinbaseSmartWallet` contract, it is important to mention that owners can remove each other freely, potentially leading to scenarios where if one of them gets compromised it can handily remove other owners and steal the assets held in the wallet. 

Functions allowed to have replayable signatures are well protected with proper access modifiers applied.

Important `_authorizeUpgrade` function is overriden with applied `onlyOwner` modifier, but the risk of a one of the admins to upgrade the wallet to a non-ERC1967 compliant contract is presented which can have serious security implications.

Another inconsistency can be the non-sequential nonce tracking in `MagicSpend` allowing bundlers to execute multiple `userOp` transactions of the same sender in their desired order, possibly extracting value from them.

Notes for improvements:

- Threshold for admin actions can be introduced, although it will require significant amount of development it can increase the security of the wallet, in exchange for making it less robust.
    - Middle ground can be to apply it only on the `upgradeToAndCall` function from `UUPSUpgradeable`

## Team Findings

| Severity | Findings |
| --- | --- |
| High | 0 |
| Medium | 1 |
| Low/NC | 8 |

## New insights and learning from this audit

Learned about:

- [ERC4337](https://eips.ethereum.org/EIPS/eip-4337#abstract)
    - [Resources](https://github.com/4337Mafia/awesome-account-abstraction)
    - [Ambire](https://github.com/AmbireTech)
    - [Infinitism](https://github.com/eth-infinitism)

## Conclusion

In general, `CoinbaseSmartWallet` protocol presents a well-designed architecture. It exhibits, such as modularity, entity separation, also strictly following ERCs and we believe the team has done a good job regarding the code. However, there are areas for improvement, including tests, and some minor inconsistencies, mentioned in the [Architecture Recommendations](#architecture-recommendations) section. Systemic and Centralization risks do not pose any significant impact to the system. 

Main code recommendations include simplifying functions, fixing typos, adding functions to ease the usage of the contracts.

It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time spent:
25 hours