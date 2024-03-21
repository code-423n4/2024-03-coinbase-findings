

# Analysis - Smart Wallet from Coinbase Wallet Contest

## Description overview of The Smart Wallet from Coinbase Wallet Contest

the Smart Wallet system provides a comprehensive framework for managing smart contract wallets, supporting multiple owners, secure signature verification, and efficient account management, all within a single, integrated protocol.It supports multiple owners for smart contracts, utilizing both Ethereum addresses and secp256r1 public keys, ensuring robust access control and permissions management. The MultiOwnable contract is a cornerstone of this system, enabling the management of multiple owners with a focus on security and efficiency.For signature verification, the system implements the ERC-1271 standard, specifically tailored for CoinbaseSmartWallet, to act as a proxy for signers and protect against cross-account replay attacks. This standard is crucial for verifying signatures on behalf of contracts, enhancing the system's security posture.The CoinbaseSmartWalletFactory contract, compliant with ERC-4337, serves as a factory for creating CoinbaseSmartWallet accounts, introducing account abstraction in Ethereum for more flexible and user-friendly account management. This contract leverages the Solidity library LibClone for creating deterministic ERC-1967 proxy contracts, ensuring each new account is uniquely identified.The CoinbaseSmartWallet contract itself is an ERC-4337 compliant smart account, incorporating features from Solady's ERC4337 account implementation and inspired by Alchemy's LightAccount and Daimo's DaimoAccount. It supports both Ethereum addresses and secp256r1 public keys as identifiers for owners, offering a comprehensive set of functionalities including signature validation, batch execution of calls, and cross-chain replayable transactions.The MagicSpend contract supports signature-based withdrawals, compliant with the ERC-4337 EntryPoint v0.6 specification, facilitating secure and efficient withdrawals by leveraging signature validation and nonce management. This contract is part of the broader ecosystem of ERC-4337, enabling more flexible and user-friendly account management.The system also integrates the WebAuthn library for verifying WebAuthn Authentication Assertions, utilizing the RIP-7212 precompile for signature verification as a fallback to the FreshCryptoLib, ensuring secure authentication within the Ethereum ecosystem. Additionally, the FCL library is designed for verifying secp256r1 signatures, focusing on optimized elliptic curve cryptography (ECC) operations for Ethereum smart contracts, enhancing the system's security and efficiency.

## System Overview

### Scope 

- **MultiOwnable.sol**: The MultiOwnable contract is structured around a storage layout named MultiOwnableStorage, which includes mappings to track owners by their index and by their raw bytes representation. This design allows for efficient management of owners, supporting both Ethereum addresses and public keys, facilitating a flexible and scalable approach to access control. The contract also includes a mechanism for initializing owners upon deployment, ensuring that the contract's initial state is securely configured.

_ **ERC1271.sol**: The contract is designed to work with EIP-712, a standard for hashing and signing of typed structured data. It aims to improve the usability of off-chain message signing for on-chain use, providing a secure and standardized method for verifying signatures. The contract includes functions for generating EIP-712 compliant hashes (_eip712Hash and replaySafeHash), validating signatures (isValidSignature), and defining the EIP-712 domain (eip712Domain and domainSeparator). These functionalities are crucial for ensuring the integrity and security of transactions and interactions within the smart contract ecosystem.

_ **CoinbaseSmartWalletFactory.sol**: The CoinbaseSmartWalletFactory operates by deploying new ERC-4337 compliant accounts using a deterministic addressing scheme. This is achieved through the use of CREATE2 for account creation, ensuring that the order of account creation does not interfere with the generated addresses. The factory contract is designed to be staked if it accesses global storage, as per the ERC-4337 specifications, to mitigate potential risks associated with global state access. The contract also includes functionality for predicting the address of an account before it is deployed, enhancing the user experience by allowing for address verification without the need for deployment.

_ **CoinbaseSmartWallet.sol**: The CoinbaseSmartWallet operates as a smart contract wallet that supports multiple owners, allowing for flexible access control and transaction management. It utilizes the UUPSUpgradeable contract for upgradeability, ensuring that the contract can be updated to fix bugs, add features, or improve security without affecting the existing state. The Receiver contract enables the wallet to receive funds, while the ERC1271 contract provides a standard interface for validating signatures, facilitating secure interactions with the wallet. The contract also includes mechanisms for executing batch transactions and validating user operations, enhancing its utility and efficiency.

_ **MagicSpend.sol**: The MagicSpend contract operates within the ERC-4337 framework, serving as a paymaster that enables accounts to withdraw funds securely. It supports ETH withdrawals, with a structured approach to managing withdrawal requests, including signature validation, nonce tracking, and expiry management. The contract is designed to be integrated with an ERC-4337 EntryPoint, which acts as the central point for validating and executing user operations. The MagicSpend contract's functionality is extended to support paymasters, allowing it to sponsor transactions for other users, facilitating use cases such as subsidizing fees or enabling fee payment with ERC-20 tokens.

_ **WebAuthn.sol**: The WebAuthn library operates by verifying the authenticity of WebAuthn assertions. It checks various aspects of the assertion, including the authenticator data, client data JSON, and the signature over the authenticator data and client data JSON. The library supports verification of the "User Present" and "User Verified" flags within the authenticator data, ensuring that the user was present and, if required, verified during the authentication process. It also verifies the type of the client data JSON and the challenge contained within it, aligning with the WebAuthn specification's requirements for assertion verification.

_ **FCL.sol**:  The FCL library operates by providing a set of functions for ECDSA signature verification on the secp256r1 curve. It includes core functions for verifying signatures (ecdsa_verify) and supporting functions for elliptic curve operations, such as checking if a point is on the curve (ecAff_isOnCurve) and performing modular inversion (FCL_nModInv). The library is designed to be efficient and secure, utilizing assembly language for critical operations and optimizing memory access to minimize gas costs. It also supports the use of precomputations to further reduce the computational overhead of ECC operations on the EVM.

## The Smart Wallet protocol defines several roles used for access control within the Smart Wallet.

- **Multiple Ownership and Owner Management**: This system allows for multiple entities to own the contract, identified by their raw bytes, and provides functions to add or remove owners. This setup supports a decentralized control mechanism, enhancing security and flexibility.
Access Control: Implemented through a modifier onlyOwner, this feature restricts access to certain functions to only the contract's owners, ensuring that only authorized entities can execute critical operations.
- **ERC-1271 Implementation**: By adhering to the ERC-1271 standard, the system enables signature verification on arbitrary data, accommodating various use cases such as multi-signature wallets and social recovery wallets. This standardization facilitates secure and efficient signature validation processes tailored to smart contract wallets.
- **Cross-Account Replay Protection and EIP-712 Compliance**: The system employs EIP-712 compliant hashes to prevent cross-account replay attacks, ensuring that the same signature cannot be used across different accounts owned by the same signer. This measure significantly enhances the security posture of the system.
- **ERC-4337 Implementation**: This standard is utilized for creating upgradeable contracts, allowing for the deployment of new instances of the CoinbaseSmartWallet contract at deterministic addresses. This approach supports the system's upgradeability and flexibility.
- **MultiOwnable and UUPSUpgradeable**: The system inherits from MultiOwnable for managing multiple owners and UUPSUpgradeable for upgradeability, aligning with the ERC-4337 standard for upgradeable contracts. This combination ensures robust ownership management and upgradeability.
- **ERC1271 Signature Verification**: Implementing the ERC-1271 standard for verifying signatures on arbitrary data is crucial for securely handling transactions and operations, supporting a wide range of use cases and enhancing the system's security.
- **User Operation Validation**: The system provides a method validateUserOp for validating user operations, ensuring that only authorized operations are executed. This includes handling replayable transactions across chains, further securing the system.
- **Paymaster Implementation**: By implementing the IPaymaster interface, the system includes methods for validating paymaster user operations and handling post-operation logic, crucial for sponsoring transactions on behalf of users.
- **Withdrawal Requests and Signature Validation**: The system supports withdrawal requests, which must be signed by the current owner of the contract, and includes functionality for validating withdrawal request signatures. This ensures security and prevents unauthorized withdrawals.
- **Nonce Management and Fund Management**: The system tracks used nonces to prevent replay attacks and manages the ETH available to be withdrawn per user, ensuring that users can only withdraw funds that are available in the contract. This comprehensive approach to fund management supports the system's security and operational efficiency

## Codebase Quality

| Codebase Quality Categories              | Comments |
|------------------------------------------|----------| 
| Code Maintainability and Reliability  | The system is well-structured and follows good practices for maintainability and reliability. It uses a modular approach with internal functions for adding and removing owners, which helps in keeping the codebase organized and reduces the risk of bugs.    |
| Code Comments  | The contract is well-commented, providing clear explanations for the purpose of each function, modifier, and error. This enhances readability and maintainability  |
| Documentation  | The contract includes detailed comments and documentation, which is crucial for understanding the contract's functionality and ensuring that it can be easily maintained and extended.  |
| Code Structure and Formatting  | The contract is structured logically, with clear separation of concerns and consistent formatting. This makes the code easier to read and understand.    |
| Error Handling | The contract uses Solidity's error handling mechanism to provide clear and informative error messages, which aids in debugging and understanding the contract's behavior   |
| Testing | The audit scope of the contracts to be audited is in the range of 75% to 95% and it should be aimed to be 100%.  | 



## Systemic & Centralization Risks

1. Centralization of Control: The system allows for multiple entities to own the system, but this design inherently supports centralization if a few key entities hold a majority of the ownership stakes. This could lead to a situation where a small group of owners can make significant decisions without input from the broader community, potentially leading to centralization of decision-making power.
2. Governance Risks: The system does not include a mechanism for decentralized governance, such as voting on changes or decisions. This means that any changes to the system, including adding or removing owners, must be done by the current owners themselves. This could lead to a situation where a small group of owners can make significant decisions without input from the broader community, potentially leading to centralization of decision-making power.
3. Security Risks: The system supports both Ethereum addresses and public keys for ownership, introducing additional complexity and potential security risks. If an owner's private key is compromised, their ownership could be lost. Additionally, the system does not include functionality for recovering ownership in case of lost private keys, which could lead to permanent loss of control over the system.
4. Upgradeability Risks: The system does not include any functionality for upgrading its code. This means that any vulnerabilities discovered in the system or any need for new features could only be addressed by deploying a new system, which would require consensus among all current owners. This could be a significant barrier to upgrading the system, especially if the owners are not actively engaged or if there are disagreements among them.
5. Replay Attacks: While the  MultiOwnable contract does not explicitly mention prevention of replay attacks, the use of nonces and the deterministic addressing scheme for account creation could potentially introduce vulnerabilities if not properly managed. An attacker could potentially reuse a valid transaction to execute unauthorized actions. While the ERC1271 contract includes measures to prevent replay attacks, the effectiveness of these measures depends on the implementation of the _validateSignature function and the overall security of the contract. Replay attacks could still be a risk if an attacker can intercept or predict the signature and submit it to the contract before the original intended use takes place.
6. Centralization of Signature Verification: The system centralizes the process of signature verification, which inherently supports centralization if the system itself is controlled by a small group of entities. This centralization risk is amplified if the system's code or its deployment is not transparent or auditable, allowing for potential manipulation or exploitation by those controlling the system.
7. Privileged Access Patterns: The ``CoinbaseSmartWalletFactory.sol`` contract allows for the creation of CoinbaseSmartWallet accounts with a set of initial owners. While this design inherently requires some level of privileged access to manage the accounts, the risk lies in the potential for a small group of individuals to control a significant portion of the wallets if they are the initial owners. This could lead to centralization if these individuals have the power to manipulate the contract's logic or the wallets' operations.
8. Multi-Ownership with Limited Access Control: The ``CoinbaseSmartWallet.sol`` contract utilizes a MultiOwnable pattern, allowing for multiple owners. However, the access control mechanisms (onlyEntryPoint, onlyEntryPointOrOwner) are primarily focused on restricting actions to the EntryPoint or the owners. This setup does not inherently prevent a small group of owners from having disproportionate control over the contract's operations, especially if they are the initial owners.


## Conclusion

In general, The Smart Wallet protocol presents a well-designed architecture. the Smart Wallet Ethereum ecosystem, including owner management, signature verification, authentication, and account creation. While they effectively leverage respective standards and methodologies to ensure security and standardization, careful consideration is imperative regarding potential security risks, particularly in preventing frontrunning attacks.

### Time spent:
18 hours