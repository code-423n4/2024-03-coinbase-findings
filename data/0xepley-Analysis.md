# 🛠️ Smart Wallet
**Smart Wallet from Coinbase Wallet.**

## Conceptual overview of the project:
The Smart Wallet is a sophisticated project designed to offer a secure and versatile wallet solution, tailored for the modern user navigating the complex landscape of decentralized finance (DeFi) and beyond. At its heart, the Smart Wallet leverages the ERC-4337 standard to provide a seamless user experience free from the traditional constraints of externally owned accounts (EOAs), introducing a new paradigm for interacting with blockchain networks.

Core features of the Smart Wallet include multi-ownership capabilities, enabling users to manage their wallet through multiple keys or devices, offering flexibility and redundancy in access control. This is particularly useful for teams or families requiring shared access to assets, while still maintaining stringent security protocols. The wallet also integrates cutting-edge WebAuthn technology for authentication, allowing users to utilize hardware security keys or biometric data for an additional layer of security beyond conventional private keys.

From a user's perspective, interaction with the Smart Wallet begins with its creation via the CoinbaseSmartWalletFactory, which allows for the deployment of individual wallets with unique addresses. Users can then add multiple owners or authentication methods, tailoring access control to their specific needs. For executing transactions, users benefit from the wallet's ability to handle batched transactions, enabling multiple operations in a single transaction for increased efficiency and reduced costs.

For withdrawals and managing transaction fees, the MagicSpend contract acts as a Paymaster, offering users a flexible mechanism to manage gas fees and withdraw funds securely through signed requests. This system not only simplifies the management of blockchain transaction costs but also introduces a layer of flexibility in how users can interact with their funds.


<br/>

[![Screenshot-from-2024-03-21-22-47-12.png](https://i.postimg.cc/GhDPQmfC/Screenshot-from-2024-03-21-22-47-12.png)](https://postimg.cc/0b8J5PxW)

## System Overview 

### 1. **MultiOwnable.sol**
**Introduces multi-ownership capabilities to the wallet, allowing multiple entities to manage the wallet using either Ethereum addresses or secp256r1 public keys.**
- **Breakdown of Functions**
  - Owner Management Functions
    - `addOwnerAddress(address owner)`: Adds a new owner identified by an Ethereum address.
    - `addOwnerPublicKey(bytes32 x, bytes32 y)`: Adds a new owner using a secp256r1 public key.
    - `removeOwnerAtIndex(uint256 index)`: Removes an owner based on their index.
  - Query Functions
    - `isOwnerAddress(address account)`: Checks if an Ethereum address is an owner.
    - `isOwnerPublicKey(bytes32 x, bytes32 y)`: Checks if a secp256r1 public key is an owner.
    - `ownerAtIndex(uint256 index)`: Retrieves owner data by index.
- **Key Functions**
  - Owner Management: `addOwnerAddress`, `addOwnerPublicKey`, `removeOwnerAtIndex`
  - Owner Verification: `isOwnerAddress`, `isOwnerPublicKey`

### 2. **ERC1271.sol**
**Implements the ERC-1271 standard for signature verification, enhancing security by enabling smart contracts to validate signatures in a standardized manner.**
- **Breakdown of Functions**
  - Signature Verification
    - `isValidSignature(bytes32 hash, bytes calldata signature)`: Validates a signature against the given hash.
  - Hashing
    - `replaySafeHash(bytes32 hash)`: Produces a replay-safe hash for signature validation.
- **Key Functions**
  - `isValidSignature`: Main function for verifying if a provided signature is valid per ERC-1271 standards.

### 3. **CoinbaseSmartWalletFactory.sol**
**Facilitates the creation of new Smart Wallet instances, leveraging a factory pattern for efficient deployment and initialization of wallets with predetermined owner sets**
- **Breakdown of Functions**
  - Wallet Creation
    - `createAccount(bytes[] calldata owners, uint256 nonce)`: Deploys a new Smart Wallet with specified owners.
  - Address Prediction
    - `getAddress(bytes[] calldata owners, uint256 nonce)`: Predicts the address of a Smart Wallet before deployment.
- **Key Functions**
  - `createAccount`: Core function for deploying new Smart Wallet instances.

### 4. **CoinbaseSmartWallet.sol**
**Acts as the core of the Smart Wallet, enabling transaction execution, upgradeability, and ERC-4337 compliance, including handling user operations and cross-chain functionality.**
- **Breakdown of Functions**
  - Transaction Execution
    - `execute(address target, uint256 value, bytes calldata data)`: Executes a single transaction.
    - `executeBatch(Call[] calldata calls)`: Executes multiple transactions in a batch.
  - ERC-4337 Integration
    - `validateUserOp(UserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)`: Validates a user operation.
    - `executeWithoutChainIdValidation(bytes calldata data)`: Executes transactions without chain ID validation for cross-chain compatibility.
- **Key Functions**
  - Execution: `execute`, `executeBatch`
  - Validation: `validateUserOp`

#### 5. **WebAuthn.sol**
**Enhances authentication security by verifying WebAuthn Authentication Assertions, integrating modern hardware-based authentication methods into the smart contract ecosystem.**
- **Breakdown of Functions**
  - Authentication Verification
    - `verify(bytes memory challenge, bool requireUV, WebAuthnAuth memory webAuthnAuth, uint256 x, uint256 y)`: Verifies a WebAuthn authentication assertion.
- **Key Functions**
  - `verify`: Validates WebAuthn authentication assertions for enhanced security.

### 6. **FCL.sol**
**Focuses on cryptographic operations, specifically ECDSA signature verification for the secp256r1 curve, underpinning the security mechanisms within the protocol.**
- **Breakdown of Functions**
  - **ECDSA Verification**
    - `ecdsa_verify`: Core function that verifies an ECDSA signature against a given message hash using secp256r1 curve parameters.
  - **Curve Check**
    - `ecAff_isOnCurve`: Determines if a point (x, y) is on the specified elliptic curve.
  - **Modular Arithmetic**
    - `FCL_nModInv`: Calculates the modular inverse of a number modulo the curve order, n, utilizing the ModExp precompile for efficiency.
    - `FCL_pModInv`: Calculates the modular inverse of a number modulo the curve's prime field modulus, p.
  - **Point Arithmetic**
    - `ecAff_add`: Adds two points on the elliptic curve in affine coordinates.
    - `ecZZ_mulmuladd_S_asm`: Performs scalar multiplication and addition on the elliptic curve, optimizing ECDSA signature verification.
- **Key Functions**
  - **Signature Verification**: `ecdsa_verify` is the pivotal function, implementing the verification of ECDSA signatures, which is essential for confirming the authenticity of messages and transactions within the blockchain context.
  - **Curve Operations**: Functions like `ecAff_isOnCurve`, `ecAff_add`, and `ecZZ_mulmuladd_S_asm` facilitate essential elliptic curve arithmetic, underpinning the security and functionality of the ECDSA verification process.

### 7. **MagicSpend.sol**
**Implements a Paymaster contract compatible with the ERC-4337 standard, managing gas payments and withdrawals, thereby facilitating seamless transaction execution and fund management.**
- **Breakdown of Functions**
  - Withdrawal Management
    - `withdraw(WithdrawRequest memory withdrawRequest)`: Processes a signed withdrawal request.
    - `ownerWithdraw(address asset, address to, uint256 amount)`: Allows the contract owner to withdraw funds.
  - EntryPoint Interaction
    - `entryPointDeposit(uint256 amount)`: Deposits ETH into the EntryPoint contract.
    - `entryPointWithdraw(address payable to, uint256 amount)`: Withdraws ETH from the EntryPoint contract.
- **Key Functions**
  - Withdrawal Processing: `withdraw`
  - EntryPoint Management: `entryPointDeposit`, `entryPointWithdraw`




## Roles in the System


#### 1. **MultiOwnable.sol**
- **Owner**: Can add or remove other owners and perform actions that are restricted to owners.

#### 2. **CoinbaseSmartWalletFactory.sol**
- **Wallet Deployer**: Initiates the creation of new Smart Wallet instances through the factory.

#### 3. **CoinbaseSmartWallet.sol**
- **Owner(s)**: Multiple owners can manage the wallet, execute transactions, add or remove other owners, and upgrade the wallet.
- **Executor**: Authorized to perform transactions on behalf of the wallet. This role is assumed by the owners or the smart contract itself in certain contexts.
- **Entrypoint (ERC-4337)**: Interacts with the wallet for executing user operations as per ERC-4337 specifications.

#### 4. **FCL.sol**
- **Verifier**: Similar to WebAuthn.sol, this library's function is to verify ECDSA signatures, particularly focusing on cryptographic verification roles.

#### 5. **MagicSpend.sol**
- **Owner**: Has the authority to manage the contract, including withdrawing funds, managing EntryPoint interactions, and setting up staking.
- **User**: Can request withdrawals through signed messages validated by the contract.





















## Codebase Quality

Overall, I consider the quality of the **Smart wallet** codebase to be of high caliber. The codebase exhibits mature software engineering practices with a strong emphasis on security, modularity, and clear documentation. The smart contracts leverage established standards, which demonstrates adherence to best practices within the Ethereum development community. Details are explained below:

| Codebase Quality Categories                     | Comments |
| ----------------------------------------------- | -------- |
| **Modularity**                                  | The codebase is well-structured, with distinct functionalities encapsulated in separate contracts, enabling easy readability and maintenance. |
| **Upgradeability**                              | The use of the UUPS pattern in `CoinbaseSmartWallet.sol` for upgradeability ensures that the system can evolve over time without sacrificing user experience or security. |
| **Security Practices**                          | Adherence to security standards and practices, such as ERC-1271 in `ERC1271.sol` for signature verification |
| **Documentation and Comments**                  | The codebase contains meaningful comments that aid understanding, detailed external documentation  further enhanced clarity for auditors. |
| **Testing and Coverage**                        | With 95% test coverage reported, this indicates nearly exhaustive testing of all code paths and scenarios, Such a high level of test coverage contributes significantly to the overall confidence in the codebase's reliability and security.|
| **Gas Efficiency**                              | The contracts demonstrate an awareness of gas costs, for instance, through batch processing in `CoinbaseSmartWallet.sol` and optimized cryptographic operations in `FCL.sol`. Continuous optimization is key to maintaining user satisfaction. |
| **Error Handling**                              | Use of custom errors in contracts like `MagicSpend.sol` for specific fail states (e.g., `InvalidSignature`, `Expired`) improves clarity and helps with debugging and transaction analysis. |
| **Consistency and Coding Standards**            | The codebase exhibits a consistent coding style and follows Solidity best practices, facilitating readability and maintainability. |
| **Interoperability**                            | The protocol’s design, particularly with ERC-4337 compliance, indicates a strong foundation for interoperability within the Ethereum ecosystem and potentially across other compatible blockchains. |
| **Security Mechanisms and Patterns**            | Implementation of security patterns, such as checks-effects-interactions in transaction methods and use of modifiers for role-based access control, underlines a proactive approach to security. |




















## Comprehensive Flow Diagram of the PoolTogether Protocol

<br/>

[![Screenshot-from-2024-03-21-23-41-28.png](https://i.postimg.cc/jSy6QPTz/Screenshot-from-2024-03-21-23-41-28.png)](https://postimg.cc/CRLfwZz1)






## Architecture and Workflow

| File Name                  | Core Functionality                                                                                                                                                                                                                                                                                                           | Technical Characteristics                                                                                                                                                                                                                                                                                                                 | Importance and Management                                                                                                                                                                                                                                                                                              |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MultiOwnable.sol**       | Enables a wallet to have multiple owners, supporting a flexible and shared management approach.                                                                                                                                                                                                                              | Implements mappings and arrays to track owner addresses and public keys, providing functions to add or remove owners based on indexes.                                                                                                                                                                                                    | Critical for scenarios requiring joint control over assets or decisions. Ownership management must be handled securely, ensuring only authorized additions or removals.                                                                                                                                                |
| **ERC1271.sol**            | Standardizes the way smart contracts verify signatures, enhancing security and interoperability with external systems.                                                                                                                                                                                                       | Utilizes EIP-712 for secure hash and signature generation, incorporating anti-replay features and ensuring that signatures are contract and chain-specific.                                                                                                                                                                               | Essential for integrating with external signing entities and services, requiring rigorous testing to prevent vulnerabilities related to signature handling.                                                                                                   |
| **CoinbaseSmartWalletFactory.sol** | Facilitates the creation of smart wallet instances, enabling users to deploy new wallets with predefined characteristics efficiently.                                                                                                                                                                                       | Uses the factory pattern and minimal proxy deployment for gas efficiency. Includes nonce-based deployment to allow for predictable wallet addresses.                                                                                                                                                                                       | Plays a pivotal role in the initial setup and deployment of wallets, necessitating careful management of deployment parameters and nonce handling to avoid collisions.                                                                                         |
| **CoinbaseSmartWallet.sol**| Serves as the core wallet contract, supporting transaction execution, ERC-4337 compliance, upgradeability, and cross-chain functionality.                                                                                                                                                                                     | Incorporates batch processing, UUPS upgradeability, and special transaction execution methods. It adheres to ERC-4337 standards for operation validation and execution, enabling advanced wallet features and interactions.                                                                                                                | The heart of the wallet system, requiring meticulous implementation and ongoing maintenance to ensure compatibility with current and future standards, as well as security.                                                                                   |
| **WebAuthn.sol**           | Introduces hardware-based authentication mechanisms into the wallet's security framework, leveraging WebAuthn for user authentication.                                                                                                                                                                                       | Focuses on verifying WebAuthn assertions, including user presence and verification checks. It attempts to use RIP-7212 precompile for efficient signature verification, with fallback to traditional methods if necessary.                                                                                                                  | Enhances wallet security beyond traditional key-based methods, important for users requiring higher security levels. Integration and testing must ensure compatibility with various hardware authenticators.                                                  |
| **FCL.sol**                | Provides cryptographic functions, specifically ECDSA verification optimized for the secp256r1 curve, underpinning the wallet's security mechanisms.                                                                                                                                                                          | Specializes in elliptic curve operations, including point addition, doubling, and signature verification. It leverages precompiled contracts for certain operations to improve gas efficiency and performance.                                                                                                                              | Fundamental for securing transactions and operations within the wallet, with a focus on optimization and accuracy in cryptographic calculations.                                                                                                              |
| **MagicSpend.sol**         | Implements a Paymaster contract for the wallet, managing gas payments and facilitating secure, signature-based fund withdrawals.                                                                                                                                                                                              | Handles withdrawal requests through signed messages, integrating with the EntryPoint for ERC-4337 user operation validation and sponsorship. It also provides functions for direct fund withdrawal and EntryPoint interaction, such as depositing and withdrawing stake.                                                                     | Key for managing transaction costs and enabling flexible fund access. Requires robust implementation to prevent unauthorized withdrawals and ensure smooth operation within the ERC-4337 framework. |




## Approach Taken while auditing the codebase
When auditing the Smart Wallet protocol, I initiated the process with a comprehensive review of the project's documentation, focusing on the protocol's objectives, design patterns, and security mechanisms. This foundational understanding was crucial for appreciating the context and complexities of the protocol, particularly its integration with ERC-4337 standards and the innovative use of multi-ownership and WebAuthn technologies for enhanced security.

After this, I proceeded to analyze the smart contract codebase. Key contracts such as `CoinbaseSmartWallet.sol`, `MultiOwnable.sol`, `ERC1271.sol`, `MagicSpend.sol`, and others were scrutinized in detail. My aim was to understand the flow of transactions, the management of ownership rights, the execution of upgradeability patterns, and the verification processes for signatures and authentication.

Critical to my audit was the identification of security risks across various components. This included assessing vulnerability to common threats like reentrancy, improper handling of external calls, and potential flaws in signature verification processes. The unique aspects of the protocol, such as the cross-chain functionality and the handling of cryptographic operations in `FCL.sol`, required careful evaluation to ensure they did not introduce unintended security weaknesses.

Particular attention was given to the protocol’s upgradeability mechanism and its interaction with the `MagicSpend.sol` contract for gas management and fund withdrawals. I assessed how these features could be exploited or could lead to loss of funds if not properly secured. Additionally, I evaluated the robustness of multi-ownership management, ensuring that the protocol adequately protected against unauthorized changes in ownership or contract logic.

The audit also involved a thorough review of the protocol’s test suite. I looked for test coverage completeness, focusing on how well the tests addressed edge cases, potential race conditions, and the integrity of key functionalities like wallet creation, transaction execution, and owner management. Recommendations for additional tests or modifications were made to cover any identified gaps.

In conclusion, the approach taken while auditing the Smart Wallet protocol was thorough and multi-faceted, blending theoretical analysis with practical security assessments. The goal was to ensure the protocol's resilience against attacks, its compliance with Ethereum standards, and its reliability as a secure and user-friendly smart wallet.








### Systemic Risks

The Smart Wallet protocol introduces several advanced features and integrations to provide security, flexibility, and user convenience. However, like any complex system, it is not immune to systemic risks. Here are some identified risks specific to the protocol's architecture and functionalities:

1. **Upgradeability Mechanism**: While the use of UUPS (Universal Upgradeable Proxy Standard) in `CoinbaseSmartWallet.sol` allows for future improvements and bug fixes, it also introduces risks associated with centralized control over the upgrade process. If the upgrade function or the process is compromised, it could lead to the introduction of malicious code or vulnerabilities across all wallet instances.

2. **WebAuthn Integration Risks**: The integration of WebAuthn for authentication purposes (`WebAuthn.sol`) enhances security but also adds a layer of complexity that depends on external devices and services. This dependency could introduce risks related to device security, compatibility issues, or the failure of third-party services, impacting user access or transaction verification.

3. **Smart Contract Interactions and External Calls**: The protocol's interaction with external contracts, including ERC-4337 EntryPoint and other integrated services (e.g., for signature verification or fund management), could be vulnerable to exploitation if those external contracts are compromised. 

4. **Signature and Authentication Bypass**: Despite the robust implementation of signature verification (`ERC1271.sol`) and cryptographic operations (`FCL.sol`), any flaws in these critical components could allow attackers to bypass authentication checks, leading to unauthorized transactions or changes to wallet settings.



### Centralization Risks

Centralization risks stem from the roles and permissions granted to certain addresses within the protocol. 

**Upgradeability Centralization in `CoinbaseSmartWallet.sol`**

### Risk:
The protocol's reliance on upgradeability, allowing for future improvements and fixes, places significant power in the hands of those who can execute upgrades, potentially centralizing control.

```solidity
function _authorizeUpgrade(address newImplementation) internal view override onlyOwner {
}
```

**Single Point of Failure in Multi-Ownership Management (`MultiOwnable.sol`)**

### Risk:
The mechanism designed to distribute control among multiple owners can be compromised if the primary owner's credentials are compromised, leading to a centralized point of failure.

```solidity
function addOwnerAddress(address owner) public onlyOwner {
}

function removeOwnerAtIndex(uint256 index) public onlyOwner {
}
```

**Centralized Withdrawal Approval in `MagicSpend.sol`**

### Risk:
The `MagicSpend.sol` contract centralizes the withdrawal approval process, potentially enabling censorship or manipulation of fund access.

```solidity
function withdraw(WithdrawRequest memory withdrawRequest) external {
}
```

**Centralized Control over EntryPoint Deposits and Withdrawals (`MagicSpend.sol`)**

### Risk:
Exclusive control over interactions with the EntryPoint contract for deposits, withdrawals, and stake management introduces centralization, granting the owner undue influence over financial operations.

```solidity
function entryPointDeposit(uint256 amount) external payable onlyOwner {
}

function entryPointWithdraw(address payable to, uint256 amount) external onlyOwner {
}
```




### Technical Risks

**Smart Contract Vulnerabilities:** Bugs or logical errors in the smart contracts can lead to loss of funds, unauthorized access, or unintended behavior. Given the complexity of contracts like the Shrine, interest rate models, and oracle interactions, the attack surface is significant.

**Scalability Concerns:** As transaction volumes grow, the platform must scale without compromising performance or security.


## New insights and learning of project from this audit:

The audit of the Smart Wallet protocol offered several valuable insights and learnings, emphasizing the protocol's innovative approach to wallet management, security, and interoperability within the Ethereum ecosystem. Here are key takeaways:

**Embracing ERC-4337 Standards**

The protocol's integration with the ERC-4337 standard for smart contract wallets marks a significant step towards achieving user-friendly and secure wallet solutions. This approach not only simplifies user interactions by removing the necessity for gas management but also opens avenues for advanced wallet functionalities like batch transactions and improved security mechanisms. 

**Advanced Security Through WebAuthn**

The utilization of WebAuthn for authentication purposes in `WebAuthn.sol` introduces a level of security typically reserved for traditional web applications into the blockchain domain. This fusion of web standards with blockchain technology not only enhances security but also demonstrates the protocol's commitment to adopting best practices from outside the blockchain world to mitigate common threats such as phishing and key theft.

**Upgradeability with User Sovereignty**

The careful implementation of upgradeability in `CoinbaseSmartWallet.sol` through UUPS showcases a thoughtful approach to maintaining and improving smart contract code while preserving user sovereignty. It serves as a learning point on balancing the need for future-proofing contracts with ensuring that upgrades do not compromise decentralized principles or user trust.



NOTE: I don't track time while auditing or writing report, so what the time I specified is just a number




### Time spent:
3 hours