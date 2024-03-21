## Pre-request : Understanding ERC-4337 for this Audit

best explanation with diagram

## https://twitter.com/AckeeBlockchain/status/1728853559662506012?s=20

overview of ERC-4337 with Diagram

https://ibb.co/yqcjhDt

## System Overview

### Scope

- src/SmartWallet/MultiOwnable.sol
    
    - contains code for a new, [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337) compliant smart contract wallet from Coinbase
- src/SmartWallet/ERC1271.sol
    
    - The purpose of `ERC1271.sol` is to implement an abstract version of the ERC-1271 standard with additional features to prevent cross-account replay attacks. Specifically, it introduces an anti cross-account-replay layer by inputting the original hash into a new EIP-712 compliant hash. This new hash includes a domain separator containing the chain ID and the address of the contract, ensuring that a signature validated on one account cannot be reused on another account. This is particularly important for scenarios where the same signer might be used across multiple accounts, enhancing security by preventing unauthorized reuse of signatures.
- src/SmartWallet/CoinbaseSmartWalletFactory.sol
    
    - `CoinbaseSmartWalletFactory.sol` is serve as a factory for creating instances of `CoinbaseSmartWallet` accounts. It is designed based on Solady's `ERC4337Factory` and allows for the deployment of ERC-4337 account proxies with a specific set of initial owners. The factory initializes with an address of the ERC-4337 implementation to use for future account deployments. It provides a function `createAccount` that deploys an ERC-4337 account and returns its deterministic address. This function requires a set of initial owners and a nonce to allow for the creation of multiple accounts with the same set of initial owners. The factory ensures that each new `CoinbaseSmartWallet` account has at least one owner by reverting if the `owners` parameter is empty, enforcing the requirement through the `OwnerRequired` error.
- src/SmartWallet/CoinbaseSmartWallet.sol
    
    - The purpose of `CoinbaseSmartWallet.sol` is to provide an ERC4337-compatible smart contract wallet. It is based on the Solady ERC4337 account implementation. This smart contract wallet incorporates features such as multi-ownership, upgradeability via UUPS (Universal Upgradeable Proxy Standard), and signature validation mechanisms. It supports both ECDSA signatures and WebAuthn for authentication, making it versatile for different security requirements. Additionally, it includes functionalities for executing batch calls and handling cross-chain replayable transactions with a reserved nonce key for sequential sequencing.
- src/WebAuthnSol/WebAuthn.sol
    
    - Webauthn-sol is a Solidity library for verifying WebAuthn authentication assertions. It builds on [Daimo's WebAuthn.sol](https://github.com/daimo-eth/p256-verifier/blob/master/src/WebAuthn.sol).
        
        This library is optimized for Ethereum layer 2 rollup chains but will work on all EVM chains. Signature verification always attempts to use the [RIP-7212 precompile](https://github.com/ethereum/RIPs/blob/master/RIPS/rip-7212.md) and, if this fails, falls back to using [FreshCryptoLib](https://github.com/rdubois-crypto/FreshCryptoLib/blob/master/solidity/src/FCL_ecdsa.sol#L40).
        
- src/FreshCryptoLib/FCL.sol
    
    - `FCL.sol` (FreshCryptoLib) is serve as a fallback mechanism for signature verification in the context of verifying WebAuthn Authentication Assertions. When the primary method of signature verification using the RIP-7212 precompile fails or is unavailable, `FCL.sol` is used to perform the signature verification process. This is particularly relevant in the implementation of WebAuthn within smart contracts, where ensuring the authenticity and integrity of signatures is crucial for security purposes.
- src/MagicSpend/MagicSpend.sol
    
    - \`MagicSpend.sol\` serve as an ERC4337 Paymaster implementation compatible with Entrypoint v0.6. It facilitates the process of signed withdraw requests, allowing accounts to withdraw funds from the contract. This implementation supports only ETH withdrawals. It includes mechanisms to prevent replay attacks through the use of unique nonces and tracks the ETH available to be withdrawn per user. Additionally, it enforces signature validation and expiry checks for withdraw requests, ensuring that only valid and timely

# Overview

This audit covers four separate but related groups of code

- SmartWallet is a smart contract wallet. In addition to Ethereum address owners, it supports passkey owners and validates their signatures via WebAuthnSol. It supports multiple owners and allows for signing account-changing user operations such that they can be replayed across any EVM chain where the account has the same address. It is ERC-4337 compliant and can be used with paymasters such as MagicSpend.
- WebAuthnSol is a library for verifying WebAuthn Authentication Assertions onchain.
- FreshCryptoLib is an excerpt from <ins>FreshCryptoLib</ins>, including the function `ecdsa_verify` and all code this function depends on. `ecdsa_verify` is used by WebAuthnSol onchains without the RIP-7212 verifier, and `FCL.n` is used to check for signature malleability.
- MagicSpend is a contract that allows for signature-based withdraws. MagicSpend is a EntryPoint v0.6 compliant paymaster and also allows using withdraws to pay transaction gas, in this way.

### additional information

1.  **User/Owner**:  initiates operations such as transactions or upgrades.
    
2.  **Bundler (EOA - Externally Owned Account)**: Responsible for submitting `UserOperation` to the EntryPoint. The bundler plays a crucial role in the transaction process, especially in the context of ERC-4337, by ensuring that user operations are correctly packaged and submitted to the network.
    
3.  **EntryPoint**: A contract that acts as the entry point for executing `UserOperations`. It is responsible for validating user operations, interacting with paymasters for fee sponsorship, and executing the operations. The EntryPoint also manages deposits and stakes for paymasters and potentially for accounts.
    
4.  **Paymaster**: Entities that sponsor transactions for users. Paymasters deposit ETH into the EntryPoint to cover transaction fees for operations they agree to sponsor. They must validate operations they're sponsoring and can have their stake affected based on their behavior and the system's reputation mechanisms.
    
5.  **Smart Contract Wallet (SCW)**: Mentioned as `UserOperation.sender`, it validates the user operation. This role is part of the SmartWallet ecosystem, acting as the sender of operations and interacting with the EntryPoint to ensure operations are valid and properly executed.
    

## Approach Taken-in Evaluating The SmartWallet

- **Initial Scope and Documentation Review**: Thoroughly went through the Contest Readme File  to understand the protocol's objectives and functionalities.
    
- **High-level Architecture Understanding**: Performed an initial architecture review of the codebase by going through all files without going into function details.
    
- **Test Environment Setup and Analysis**: Set up a test environment and execute all tests. Additionally, use Static Analyzer tools like Slither to identify potential vulnerabilities.
    
- **Comprehensive Code Review**: Conducted a line-by-line code review focusing on understanding code functionalities.
    
- **Report Writing**: Write Report by compiling all the insights I gained throughout the line by line code review.
    

&nbsp;

## Centralization Risks

SmartWallet have Centralization Risk because

1.  Owners have significant control.
2.  Owners can add/remove other owners.
3.  No limit on the number of owners, potentially centralizing power.
4.  and so on...

## 7\. Test analysis

The audit scope of the contracts to be reviewed is 95%, with the aim of reaching 100% to increase the safety.

## Codebase Quality

Overall, I consider the quality of the SmartWallet codebase to be Good. The code appears to be mature and well-developed. I have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories | Comments |
| --- | --- |
| **Code Maintainability and Reliability** | The SmartWallet contracts demonstrates good maintainability through modular structure, consistent naming. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space. |
| **Code Comments** | The code is well commented with NatSpec comments, which are clear and informative. |
| **Documentation** | The way they put everything in the ReadMe associated with the code was very helpful. it was very easy to find the details about each contract. |
| **Code Structure and Formatting** | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last. |
| **Error** | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain. |
| **Imports** | In SmartWallet  protocol the contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files. |
| **Gas Optimization** | SmartWallet  demonstrates a strong commitment to gas optimization, incorporating many widely accepted techniques. |

## Conclusion

In general, the SmartWallet project exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

## Time Spend

23 hours

### Time spent:
23 hours