# Coinbase Smart Wallet Security Analysis Report

## 1. An Overview of the Coinbase Smart Wallet

The Coinbase Smart Wallet is an advanced, ERC-4337 compatible smart contract wallet designed to provide users with enhanced security, flexibility, and usability features. Built on the Ethereum blockchain, the Smart Wallet aims to address the limitations of traditional externally owned accounts (EOAs) by offering a multi-owner architecture, support for hardware-based authentication (WebAuthn), and cross-chain functionality.

## 2. Evaluation Approach

The security analysis of the Coinbase Smart Wallet codebase was conducted using a combination of manual code review, static analysis tools, and dynamic testing techniques. The following steps were taken during the evaluation process:

1. Code Review: A thorough manual review of the smart contract code was performed to identify potential vulnerabilities, logic flaws, and deviations from best practices.

2. Architecture Analysis: The overall architecture of the Smart Wallet was examined to assess its design, modularity, and potential risks associated with the chosen architecture.

3. Centralization and Admin Control: The codebase was analyzed to identify any centralization risks and potential for admin control abuse.

4. Mechanism Review: The implemented mechanisms, such as access control, signature verification, and cross-chain functionality, were reviewed to ensure their correctness and robustness.

5. Systemic Risk Assessment: Potential systemic risks arising from the interaction of the Smart Wallet with external systems and dependencies were evaluated.

## 3. Architecture Overview

The Coinbase Smart Wallet follows a modular architecture, separating concerns into different contracts and libraries. The main components of the architecture include:

- `CoinbaseSmartWallet`: The core smart wallet contract that handles user operations, access control, and cross-chain functionality.
- `MultiOwnable`: A contract that manages multiple owners and their permissions.
- `ERC1271`: An abstract contract that provides ERC-1271 signature validation functionality.
- `CoinbaseSmartWalletFactory`: A factory contract for deploying new instances of the Smart Wallet.
- `WebAuthn`: A library for verifying WebAuthn authentication assertions.
- `FreshCryptoLib`: A library for secp256r1 signature verification.

The Smart Wallet leverages the ERC-4337 standard for account abstraction and interacts with an EntryPoint contract for transaction execution.

## 4. Code Quality Analysis

The overall code quality of the Coinbase Smart Wallet is high, adhering to best practices and following a clean and modular structure. However, there are a few areas that could be improved:

1. Error Handling: The codebase could benefit from more consistent and informative error messages using custom error types. This would enhance the clarity and debugging capabilities of the contracts.

2. Code Documentation: While the code is well-structured, additional comments and documentation, particularly for complex functions and interactions, would improve the overall readability and maintainability of the codebase.

3. Gas Optimization: Some gas optimizations could be considered, such as using `calldata` instead of `memory` for function parameters when possible, and avoiding unnecessary storage reads and writes.

4. Unused Code: There are a few instances of commented-out code that should be removed to maintain code cleanliness and reduce confusion.

1. Inconsistent error messages: Some error messages lack detail or context, making it harder to understand the specific issue.

   Example:
   ```solidity
   if (owners.length == 0) {
       revert OwnerRequired();
   }
   ```
   Recommendation: Provide more descriptive error messages to improve debugging and understanding.

2. Lack of input validation: Some functions do not perform sufficient input validation, which could lead to unexpected behavior or vulnerabilities.

   Example:
   ```solidity
   function addOwnerPublicKey(bytes32 x, bytes32 y) public virtual onlyOwner {
       _addOwner(abi.encode(x, y));
   }
   ```
   Recommendation: Add input validation to ensure that the provided public key is valid and within the expected range.

3. Potential unbounded loop: The `executeAndBatch` function iterates over an array of calls, which could lead to excessive gas consumption or even contract freezing if the array is large.

   Example:
   ```solidity
   function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
       for (uint256 i; i < calls.length;) {
           _call(calls[i].target, calls[i].value, calls[i].data);
           unchecked {
               ++i;
           }
       }
   }
   ```
   Recommendation: Consider setting a maximum limit on the number of calls allowed in a single batch to prevent potential DoS attacks or gas exhaustion.

## 5. Centralization Risks and Admin Control

1. Owner Privilege Escalation: The Smart Wallet allows owners to add and remove other owners, which could lead to privilege escalation if not properly controlled.

   Example:
   ```solidity
   function addOwnerAddress(address owner) public virtual onlyOwner {
       _addOwner(abi.encode(owner));
   }
   ```
   Risk: A malicious owner could add their own address multiple times, increasing their voting power and potentially bypassing multi-signature requirements.

   Recommendation: Implement checks to prevent owners from adding the same address multiple times and ensure that adding owners requires approval from a minimum number of existing owners.

2. Upgradability Abuse: The Smart Wallet utilizes the UUPS proxy pattern, allowing the contract implementation to be upgraded by the owners.

   Example:
   ```solidity
   function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
   ```
   Risk: Malicious owners could upgrade the contract to a version that introduces backdoors or alters the expected behavior.

   Recommendation: Implement a time-lock mechanism for upgrades, allowing users to review and opt-out if necessary. Require multiple owner approvals for upgrades.

The Coinbase Smart Wallet has a multi-owner architecture, allowing multiple owners to control the wallet. While this provides flexibility and redundancy, it also introduces potential centralization risks:

1. Owner Collusion: If a subset of owners collude, they can perform actions such as adding or removing other owners, potentially leading to a centralized control of the wallet.

2. Owner Key Compromise: If an attacker gains control of an owner's private key, they can perform unauthorized actions on behalf of that owner. The risk is mitigated by requiring multiple owners to agree on critical actions.

3. Upgradeability: The Smart Wallet utilizes the UUPS (Universal Upgradeable Proxy Standard) pattern for upgradeability. While this allows for flexibility and future improvements, it also introduces the risk of centralized control if the upgrade process is not properly governed.

1. Tracing the flow of assets:
The main functions that can move assets are `execute`, `executeBatch`, and `_call` (which is used internally by the other two).

`execute` and `executeBatch` both have the `onlyEntryPointOrOwner` modifier: https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L196-L198

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L205-L212

```solidity
function execute(address target, uint256 value, bytes calldata data) public payable virtual onlyEntryPointOrOwner {
    _call(target, value, data);
}

function executeBatch(Call[] calldata calls) public payable virtual onlyEntryPointOrOwner {
    for (uint256 i; i < calls.length;) {
        _call(calls[i].target, calls[i].value, calls[i].data);
        // ...
    }
}
```

The [`onlyEntryPointOrOwner`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L74-L80) modifier checks that the caller is either the EntryPoint contract or an owner:

```solidity
modifier onlyEntryPointOrOwner() virtual {
    if (msg.sender != entryPoint()) {
        _checkOwner();
    }
    _;
}
```

Where [`_checkOwner()`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L201-L207) verifies the caller is either an owner address or the smart wallet itself:

```solidity
function _checkOwner() internal view virtual {
    if (isOwnerAddress(msg.sender) || (msg.sender == address(this))) {
        return;
    }
    revert Unauthorized();
}
```

I couldn't find any way to bypass this check. The only way to move funds is to be the EntryPoint or an owner.

> However, one potential issue is that once inside `execute` or `executeBatch`, the `target` and `value` are not restricted. An authorized caller (owner or EntryPoint) could send funds to any address. This is by design to allow flexibility, but it does mean owners need to be trusted.

2. Manipulating the owner list:
The relevant functions are `removeOwnerAtIndex`, `addOwnerAddress`, and `addOwnerPublicKey`. They all have the `onlyOwner` modifier, so only existing owners can call them.

In [`removeOwnerAtIndex`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L102-L110), there's a check to prevent removing a non-existent owner:

```solidity
function removeOwnerAtIndex(uint256 index) public virtual onlyOwner {
    bytes memory owner = ownerAtIndex(index);
    if (owner.length == 0) revert NoOwnerAtIndex(index);
    // ...
}
```

In[ `addOwnerAddress` and `addOwnerPublicKey`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L179-L196), the new owner is added via `_addOwner`:

```solidity
function _addOwner(bytes memory owner) internal virtual {
    _addOwnerAtIndex(owner, _getMultiOwnableStorage().nextOwnerIndex++);
}

function _addOwnerAtIndex(bytes memory owner, uint256 index) internal virtual {
    if (isOwnerBytes(owner)) revert AlreadyOwner(owner);
    // ...
}
```

Here, `isOwnerBytes(owner)` prevents adding an owner that already exists.

Importantly, there's no function to remove all owners at once. Owners can only be removed one at a time via `removeOwnerAtIndex`. And since an owner can't remove themselves (they don't know their own index), it's impossible for an attacker to remove all other owners and lock themselves out.

The only scenario would be if an attacker was the only owner to begin with. But that would be an issue with the initial setup, not the contract itself.

3. `executeWithoutChainIdValidation`:
This function can only be called by the EntryPoint, and it only allows a specific set of functions to be called without chain ID validation: https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L252-L262

```solidity
function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
    if (
        functionSelector == MultiOwnable.addOwnerPublicKey.selector
            || functionSelector == MultiOwnable.addOwnerAddress.selector
            || functionSelector == MultiOwnable.removeOwnerAtIndex.selector
            || functionSelector == UUPSUpgradeable.upgradeToAndCall.selector
    ) {
        return true;
    }
    return false;
}
```

The main concern here is `upgradeToAndCall`. If this was called via `executeWithoutChainIdValidation` and replayed on another chain, it could upgrade the wallet to a malicious implementation, allowing the attacker to steal funds.

The other functions (`addOwnerPublicKey`, `addOwnerAddress`, `removeOwnerAtIndex`) are less severe, as they would "only" allow the attacker to manipulate the owner list on the other chain, not directly steal funds. But this could still lead to loss of control of the wallet on that chain.

4. Adversarial thinking:
Some attack scenarios I considered:

- Can I somehow bypass the `onlyEntryPointOrOwner` check to move funds unauthorized? 
  - I couldn't find a way. The check seems solid.

- Can I manipulate the owner list to gain control?
  - An attacker could try to get themselves added as an owner (e.g., by tricking an existing owner into calling `addOwnerAddress` with the attacker's address).
  - Then, the attacker could remove all other owners.
  - But, the attacker can't remove themselves, so they can't lock everyone out.
  - The only risk is if the attacker was the only owner to begin with.

- Can I exploit `executeWithoutChainIdValidation` to replay sensitive actions cross-chain?
  - The main risk is replaying `upgradeToAndCall` to take control of the wallet on other chains.
  - The other allowed functions (`addOwnerPublicKey`, `addOwnerAddress`, `removeOwnerAtIndex`) could allow the attacker to manipulate the owner list on other chains, but not directly steal funds.

- Is there any way to brick the wallet permanently?
  - Not that I could find. There's always at least one owner, and owners can add new owners. So the wallet can always recover.

In summary, the main risks I see are:
1. Owners can move funds to any address, so owners need to be highly trusted.
2. If an attacker gets added as an owner (e.g., by tricking an existing owner), they could remove other owners and take sole control of the wallet. But they can't completely lock everyone out.
3. `executeWithoutChainIdValidation` allowing `upgradeToAndCall` is a major risk for cross-chain replay attacks leading to wallet takeover.

To mitigate these risks, it is recommended to establish robust off-chain governance mechanisms and multi-signature requirements for critical actions such as owner management and contract upgrades.

## 6. Mechanism Review

1. Signature Validation: The Smart Wallet uses ERC-1271 and WebAuthn signatures for owner authentication. The signature validation process is critical to prevent unauthorized access.

   Example:
   ```solidity
   function _validateSignature(bytes32 message, bytes calldata signature)
       internal
       view
       virtual
       override
       returns (bool)
   {
       // ...
       return SignatureCheckerLib.isValidSignatureNow(owner, message, sigWrapper.signatureData);
   }
   ```
   Risk: If the signature validation logic is flawed or if there are vulnerabilities in the underlying libraries, it could allow attackers to bypass authentication and perform unauthorized actions.

   Recommendation: Thoroughly test the signature validation process, including edge cases and potential attacks. Regularly audit and update the dependencies to ensure their security.

2. Replay Protection: The Smart Wallet includes cross-chain replay protection using the `canSkipChainIdValidation` function.

   Example:
   ```solidity
   function canSkipChainIdValidation(bytes4 functionSelector) public pure returns (bool) {
       // ...
   }
   ```
   Risk: If the replay protection mechanism is not properly implemented or maintained, it could allow attackers to replay transactions across different chains.

   Recommendation: Regularly review and update the `canSkipChainIdValidation` function to ensure it covers all relevant function selectors. Implement additional safeguards, such as nonces or timestamps, to prevent replay attacks.

The Coinbase Smart Wallet implements several key mechanisms to ensure security and functionality:

1. Access Control: The Smart Wallet enforces access control through the use of modifiers such as `onlyOwner` and `onlyEntryPointOrOwner`. These modifiers ensure that only authorized entities (owners or the EntryPoint contract) can perform certain actions.

   ```solidity
   modifier onlyOwner() virtual {
       _checkOwner();
       _;
   }

   modifier onlyEntryPointOrOwner() virtual {
       if (msg.sender != entryPoint()) {
           _checkOwner();
       }
       _;
   }
   ```

   The access control mechanisms are well-implemented and consistently used throughout the codebase. However, it is important to note that the security of the wallet relies on the integrity of the owner private keys and the EntryPoint contract.

2. Signature Verification: The Smart Wallet supports both ECDSA signatures (for EOAs) and WebAuthn authentication assertions (for hardware keys). The signature verification process is handled by the `ERC1271` contract and the `WebAuthn` library.

   ```solidity
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
           // ECDSA signature verification
           // ...
       } else if (ownerBytes.length == 64) {
           // WebAuthn signature verification
           // ...
       } else {
           revert InvalidOwnerBytesLength(ownerBytes);
       }
   }
   ```

   The signature verification process is well-structured and follows the respective standards (ERC-1271 and WebAuthn). However, it is important to ensure that the WebAuthn library is thoroughly tested and audited, as it is a critical component for hardware key-based authentication.

3. Cross-Chain Functionality: The Smart Wallet includes a mechanism for executing transactions without chain ID validation, allowing for cross-chain replay of certain functions.

   ```solidity
   function executeWithoutChainIdValidation(bytes calldata data) public payable virtual onlyEntryPoint {
       bytes4 selector = bytes4(data[0:4]);
       if (!canSkipChainIdValidation(selector)) {
           revert SelectorNotAllowed(selector);
       }
       _call(address(this), 0, data);
   }
   ```

   While this functionality provides flexibility for cross-chain operations, it also introduces potential risks if not properly controlled. The `canSkipChainIdValidation` function should be carefully reviewed to ensure that only safe and intended functions are allowed to be executed without chain ID validation.

## 7. Systemic Risks

The Coinbase Smart Wallet interacts with external systems and dependencies, which may introduce systemic risks:

1. EntryPoint Contract: The Smart Wallet relies on the EntryPoint contract for transaction execution and gas sponsorship. Any vulnerabilities or failures in the EntryPoint contract could impact the security and functionality of the Smart Wallet.

2. External Libraries: The Smart Wallet utilizes external libraries such as `FreshCryptoLib` for cryptographic operations. The security of the wallet depends on the correctness and robustness of these libraries.

3. Cross-Chain Interactions: The cross-chain functionality of the Smart Wallet introduces additional risks, as transactions executed on one chain can have unintended consequences on other chains if not properly controlled.

To mitigate these systemic risks, it is recommended to thoroughly audit and test the external dependencies, establish secure communication channels between the Smart Wallet and the EntryPoint contract, and implement strict validation and access control mechanisms for cross-chain operations.

1. Dependency Risks: The Smart Wallet relies on external libraries and contracts, such as the ERC-4337 EntryPoint and the WebAuthn verification library.

   Risk: If these dependencies contain vulnerabilities or are compromised, it could impact the security of the Smart Wallet.

   Recommendation: Regularly monitor and update the dependencies to ensure they are secure. Conduct thorough audits of the external libraries and contracts used by the Smart Wallet.

2. Upgradability Risks: The Smart Wallet's upgradability feature allows for contract improvements but also introduces risks.

   Risk: If the upgrade process is not properly managed or if a malicious implementation is deployed, it could compromise the wallet's security and lead to loss of funds.

   Recommendation: Implement strict access controls and multi-signature requirements for initiating contract upgrades. Thoroughly test and audit any proposed upgrades before deployment.

## Key Features

1. **Multi-Owner Architecture**: The Smart Wallet allows for multiple owners to control the wallet, providing redundancy and reducing the risk of a single point of failure. Owners can be added or removed through the `MultiOwnable` contract, which manages the ownership structure.

2. **Hardware-based Authentication**: In addition to traditional ECDSA signatures, the Smart Wallet supports WebAuthn authentication assertions, enabling the use of hardware keys for secure transaction signing. This enhances the overall security of the wallet by leveraging the benefits of hardware-based authentication.

3. **ERC-4337 Compatibility**: The Smart Wallet is designed to be compatible with the ERC-4337 standard for account abstraction. It interacts with an EntryPoint contract, which handles the transaction execution and gas sponsorship, enabling gasless transactions and improving the user experience.

4. **Cross-Chain Functionality**: The Smart Wallet includes a mechanism for executing transactions without chain ID validation, allowing for the replay of certain functions across different chains. This feature provides flexibility for users who operate on multiple chains, enabling them to manage their assets seamlessly.

## Risk Assessment

While the Coinbase Smart Wallet offers advanced features and security enhancements, it is essential to assess the potential risks associated with its architecture and implementation.

1. **Centralization Risks**: The multi-owner architecture of the Smart Wallet introduces the risk of centralization if a subset of owners collude or if an attacker gains control of a significant number of owner keys. This risk can be mitigated by implementing robust off-chain governance mechanisms and requiring multi-signature approvals for critical actions.

   [Code snippet from `MultiOwnable.sol`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/MultiOwnable.sol#L85-L87)
   ```solidity
   function addOwnerAddress(address owner) public virtual onlyOwner {
       _addOwner(abi.encode(owner));
   }
   ```

2. **Access Control Risks**: The Smart Wallet relies on access control modifiers, such as `onlyOwner` and `onlyEntryPointOrOwner`, to restrict access to sensitive functions. Any vulnerabilities or mistakes in the implementation of these modifiers could lead to unauthorized access and potential fund loss.

   [Code snippet from `CoinbaseSmartWallet.sol`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L74-L80):
   ```solidity
   modifier onlyEntryPointOrOwner() virtual {
       if (msg.sender != entryPoint()) {
           _checkOwner();
       }
       _;
   }
   ```

3. **Cross-Chain Risks**: The cross-chain functionality of the Smart Wallet, enabled by the `executeWithoutChainIdValidation` function, introduces the risk of unintended consequences if not properly controlled. An attacker could potentially trick an owner into signing a transaction that executes a sensitive function on another chain, leading to fund loss or wallet compromise.

   [Code snippet from `CoinbaseSmartWallet.sol`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L180-L187)
   ```solidity
   function executeWithoutChainIdValidation(bytes calldata data) public payable virtual onlyEntryPoint {
       bytes4 selector = bytes4(data[0:4]);
       if (!canSkipChainIdValidation(selector)) {
           revert SelectorNotAllowed(selector);
       }
       _call(address(this), 0, data);
   }
   ```

4. **Dependency Risks**: The Smart Wallet relies on external dependencies, such as the EntryPoint contract and the `FreshCryptoLib` library, for critical functionalities. Any vulnerabilities or failures in these dependencies could impact the security and reliability of the Smart Wallet.

5. **Upgrade Risks**: The Smart Wallet utilizes the UUPS (Universal Upgradeable Proxy Standard) pattern for upgradeability, allowing the contract implementation to be upgraded by the owners. While this provides flexibility for future enhancements, it also introduces the risk of centralized control and potential code injection if the upgrade process is not properly governed.

   [Code snippet from `CoinbaseSmartWallet.sol`](https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/SmartWallet/CoinbaseSmartWallet.sol#L330)
   ```solidity
   function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
   ```

To mitigate these risks, it is crucial to implement robust security measures, such as:
- Secure key management practices for wallet owners
- Multi-signature requirements for critical actions and upgrades
- Strict access control and validation mechanisms for cross-chain operations
- Continuous monitoring and updating of the Smart Wallet and its dependencies

## 8. Vulnerability Analysis

During the security analysis, the following potential vulnerabilities were identified:

1. Cross-Chain Replay Attacks: The `executeWithoutChainIdValidation` function allows certain functions to be executed without chain ID validation, potentially enabling cross-chain replay attacks.

   ```solidity
   function executeWithoutChainIdValidation(bytes calldata data) public payable virtual onlyEntryPoint {
       bytes4 selector = bytes4(data[0:4]);
       if (!canSkipChainIdValidation(selector)) {
           revert SelectorNotAllowed(selector);
       }
       _call(address(this), 0, data);
   }
   ```

   If an attacker can trick an owner into signing a transaction that calls a sensitive function (e.g., `upgradeToAndCall`) and replay it on another chain, it could lead to unintended consequences, such as upgrading the wallet to a malicious implementation.

   Mitigation: Remove sensitive functions, such as `upgradeToAndCall`, from the `canSkipChainIdValidation` allowlist to prevent their execution without chain ID validation.

2. Unchecked Return Values: In the `execute` and `executeBatch` functions, the return values of the low-level calls are not checked, potentially leading to silent failures.

   ```solidity
   function _call(address target, uint256 value, bytes memory data) internal {
       (bool success, bytes memory result) = target.call{value: value}(data);
       if (!success) {
           assembly ("memory-safe") {
               revert(add(result, 32), mload(result))
           }
       }
   }
   ```

   Mitigation: Add proper error handling and revert with meaningful error messages if the low-level calls return an error.

3. Unprotected Upgrades: The Smart Wallet uses the UUPS pattern for upgradeability, allowing owners to upgrade the contract implementation. However, if not properly controlled, an attacker who gains control of an owner's private key could upgrade the wallet to a malicious implementation.

   ```solidity
   function _authorizeUpgrade(address) internal view virtual override(UUPSUpgradeable) onlyOwner {}
   ```

   Mitigation: Implement a multi-signature or time-lock mechanism for contract upgrades to prevent single-owner abuse.

## 9. Recommendations

Based on the security analysis, the following recommendations are proposed to enhance the security and robustness of the Coinbase Smart Wallet:

1. Implement a more granular access control system, separating critical functions (e.g., owner management, upgrades) from regular wallet operations. Consider using a multi-signature scheme for critical actions.

2. Conduct thorough testing and auditing of the WebAuthn library and the secp256r1 signature verification process to ensure their correctness and security.

3. Establish secure off-chain communication channels and governance mechanisms for coordinating owner actions and managing wallet operations.

4. Regularly review and update the allowlist of functions that can be executed without chain ID validation to minimize the risk of cross-chain replay attacks.

5. Implement comprehensive error handling and logging mechanisms to facilitate debugging and incident response.

6. Continuously monitor the external dependencies and systems (e.g., EntryPoint contract, external libraries) for vulnerabilities and update them as needed.

7. Educate wallet owners about security best practices, such as secure key management, transaction signing, and cross-chain risks.

## 10. Conclusion

The Coinbase Smart Wallet codebase demonstrates a high level of security and adherence to best practices. The modular architecture, access control mechanisms, and signature verification processes are well-implemented. However, there are potential risks associated with centralization, cross-chain functionality, and external dependencies that need to be carefully managed.

By addressing the identified vulnerabilities, implementing the recommended security enhancements, and establishing robust governance mechanisms, the Coinbase Smart Wallet can provide a secure and reliable solution for users.

It is important to note that this security analysis is based on the provided codebase and documentation. Further testing, auditing, and monitoring should be conducted on an ongoing basis to ensure the continued security of the Smart Wallet.

# Risk Assessment and Architecture Overview of the Coinbase Smart Wallet

## Risk Assessment

The Coinbase Smart Wallet, being an advanced smart contract wallet, introduces several potential risks that need to be carefully considered and mitigated. Let's discuss the key risk areas:

1. **Centralization Risks**:
   - The multi-owner architecture of the Smart Wallet introduces the risk of centralization if a subset of owners collude or if an attacker gains control of a significant number of owner keys.
   - The upgrade mechanism, which allows owners to upgrade the contract implementation, could lead to centralized control if not properly governed.

2. **Access Control Risks**:
   - The Smart Wallet relies on access control modifiers to restrict access to sensitive functions. Any vulnerabilities or mistakes in the implementation of these modifiers could lead to unauthorized access and potential fund loss.

3. **Cross-Chain Risks**:
   - The cross-chain functionality of the Smart Wallet introduces the risk of unintended consequences if not properly controlled. An attacker could potentially trick an owner into signing a transaction that executes a sensitive function on another chain.

4. **Dependency Risks**:
   - The Smart Wallet relies on external dependencies, such as the EntryPoint contract and the `FreshCryptoLib` library. Any vulnerabilities or failures in these dependencies could impact the security and reliability of the Smart Wallet.

5. **Signature Verification Risks**:
   - The Smart Wallet supports both ECDSA signatures and WebAuthn authentication assertions. Ensuring the correctness and security of the signature verification process is crucial to prevent unauthorized access.

## Architecture Overview

![Architecture Overview Diagram](https://i.imgur.com/KcGjGmA.png)

The Coinbase Smart Wallet follows a modular architecture, separating concerns into different contracts and libraries. The main components of the architecture include:

- `CoinbaseSmartWallet`: The core smart wallet contract that handles user operations, access control, and cross-chain functionality.
- `MultiOwnable`: A contract that manages multiple owners and their permissions.
- `ERC1271`: An abstract contract that provides ERC-1271 signature validation functionality.
- `CoinbaseSmartWalletFactory`: A factory contract for deploying new instances of the Smart Wallet.
- `WebAuthn`: A library for verifying WebAuthn authentication assertions.
- `FreshCryptoLib`: A library for secp256r1 signature verification.

The Smart Wallet interacts with an EntryPoint contract, which handles transaction execution and gas sponsorship, enabling gasless transactions and improving the user experience.

## Admin Flow

![Admin Flow Diagram](https://i.imgur.com/Dt7QL5G.png)

The admin flow in the Coinbase Smart Wallet involves the management of owners and wallet settings. The key steps in the admin flow include:

1. Adding or removing owners: Owners can add new owners or remove existing owners using the `addOwnerAddress`, `addOwnerPublicKey`, and `removeOwnerAtIndex` functions in the `MultiOwnable` contract.

2. Upgrading the wallet implementation: Owners can upgrade the wallet implementation using the `upgradeToAndCall` function in the `UUPSUpgradeable` contract, which is inherited by the `CoinbaseSmartWallet` contract.

Risks involved in the admin flow:
- Centralization risk: If a subset of owners collude or an attacker gains control of a significant number of owner keys, they could manipulate the wallet settings or upgrade to a malicious implementation.
- Access control risk: Any vulnerabilities in the access control modifiers could allow unauthorized access to admin functions.

## User Flow

![User Flow Diagram](https://i.imgur.com/1VWKxIn.png)

The user flow in the Coinbase Smart Wallet involves executing transactions and managing funds. The key steps in the user flow include:

1. Executing transactions: Users can execute transactions using the `execute` or `executeBatch` functions in the `CoinbaseSmartWallet` contract. Transactions can be signed using either ECDSA signatures or WebAuthn authentication assertions.

2. Cross-chain transactions: Users can execute transactions without chain ID validation using the `executeWithoutChainIdValidation` function, allowing for the replay of certain functions across different chains.

Risks involved in the user flow:
- Cross-chain risk: If not properly controlled, an attacker could trick a user into signing a transaction that executes a sensitive function on another chain, leading to unintended consequences.
- Signature verification risk: Any vulnerabilities in the signature verification process could allow unauthorized access to user funds.

## Core Contract Flow

![Core Contract Flow Diagram](https://i.imgur.com/6yjOrsN.png)

The core contract flow in the Coinbase Smart Wallet involves the interaction between the main contracts and libraries. The key components and their interactions include:

1. `CoinbaseSmartWallet`: The core smart wallet contract that handles user operations, access control, and cross-chain functionality. It interacts with the `MultiOwnable` contract for owner management and the `ERC1271` contract for signature validation.

2. `MultiOwnable`: A contract that manages multiple owners and their permissions. It is used by the `CoinbaseSmartWallet` contract to control access to sensitive functions.

3. `ERC1271`: An abstract contract that provides ERC-1271 signature validation functionality. It is inherited by the `CoinbaseSmartWallet` contract to support signature-based authentication.

4. `WebAuthn` and `FreshCryptoLib`: Libraries used by the `CoinbaseSmartWallet` contract for verifying WebAuthn authentication assertions and secp256r1 signatures, respectively.

Risks involved in the core contract flow:
- Dependency risk: Any vulnerabilities or failures in the external libraries (`WebAuthn` and `FreshCryptoLib`) could impact the security of the Smart Wallet.
- Mechanism risk: Any flaws in the implementation of the core mechanisms, such as access control or signature verification, could lead to unauthorized access or fund loss.

## Contract Analysis

![Contract Analysis Diagram](https://i.imgur.com/iOGtJvP.png)

The contract analysis diagram provides an overview of the main contracts, their relationships, and the potential risks associated with each component:

1. `CoinbaseSmartWallet`:
   - Centralization risk: The multi-owner architecture and upgrade mechanism could lead to centralized control if not properly managed.
   - Cross-chain risk: The `executeWithoutChainIdValidation` function introduces the risk of unintended consequences if not properly controlled.

2. `MultiOwnable`:
   - Access control risk: Any vulnerabilities in the access control modifiers could allow unauthorized access to owner management functions.

3. `ERC1271`:
   - Signature verification risk: Ensuring the correctness and security of the signature verification process is crucial to prevent unauthorized access.

4. `WebAuthn` and `FreshCryptoLib`:
   - Dependency risk: Any vulnerabilities or failures in these libraries could impact the security of the Smart Wallet.

5. `EntryPoint`:
   - Mechanism risk: The Smart Wallet relies on the EntryPoint contract for transaction execution and gas sponsorship. Any issues with the EntryPoint mechanism could affect the functionality and security of the Smart Wallet.

By understanding the architecture, flows, and associated risks, developers and auditors can identify potential vulnerabilities and implement appropriate mitigation measures to ensure the security and reliability of the Coinbase Smart Wallet.

Review of the SmartWallet contract for missing or incomplete access control checks, verify the trustworthiness and correct integration of external contracts and libraries, and ensure the contract cannot be self-destructed or locked into an unusable state.

1. Access Control Checks:
   a. Owner Management Functions:
      - The `addOwnerAddress`, `addOwnerPublicKey`, and `removeOwnerAtIndex` functions in the `MultiOwnable` contract are protected by the `onlyOwner` modifier, ensuring that only existing owners can add or remove owners.
      - However, there is a potential issue with the `removeOwnerAtIndex` function, as it allows removing an owner without checking if it would leave the contract without any owners. This could lead to a situation where the contract becomes inaccessible if all owners are removed.

   b. Contract Upgrade Functions:
      - The `upgradeToAndCall` function in the `UUPSUpgradeable` contract is protected by the `onlyProxy` modifier, which is overridden in the `SmartWallet` contract to use the `onlyOwner` modifier.
      - This ensures that only existing owners can upgrade the contract to a new implementation.
      - However, there are no additional checks on the `newImplementation` address to verify that it is a valid and trusted contract.

   c. Fund Movement Functions:
      - The `execute`, `executeBatch`, and `executeWithoutChainIdValidation` functions are protected by the `onlyEntryPointOrOwner` and `onlyEntryPoint` modifiers, respectively.
      - This ensures that only the EntryPoint contract or an existing owner can execute transactions and move funds.
      - The access control for these functions appears to be properly implemented.

2. External Contracts and Libraries:
   a. Solady's `Ownable`:
      - The `Ownable` contract from the Solady library is used to manage ownership in the `SmartWallet` contract.
      - Solady is a well-known and trusted library, and the `Ownable` contract has been widely used and audited.
      - The integration of the `Ownable` contract appears to be correct, with the `onlyOwner` modifier being used appropriately.

   b. Solady's `UUPSUpgradeable`:
      - The `UUPSUpgradeable` contract from the Solady library is used to enable upgradability in the `SmartWallet` contract.
      - Solady's implementation of the UUPS (Universal Upgradeable Proxy Standard) pattern is widely used and has undergone security audits.
      - The integration of the `UUPSUpgradeable` contract appears to be correct, with the `_authorizeUpgrade` function being overridden to use the `onlyOwner` modifier.

   c. WebAuthnSol:
      - The `WebAuthnSol` contract is used for verifying WebAuthn authentication assertions in the `SmartWallet` contract.
      - It is important to ensure that the `WebAuthnSol` contract has been thoroughly audited and is secure.
      - The integration of the `WebAuthnSol` contract appears to be correct, with the `verify` function being used appropriately in the `_validateSignature` function.

   d. ERC1271:
      - The `ERC1271` contract is used for signature validation in the `SmartWallet` contract.
      - The `ERC1271` standard is widely used and has undergone security audits.
      - The integration of the `ERC1271` contract appears to be correct, with the `isValidSignature` function being implemented properly.

3. Self-Destruct and Unusable State:
   - The `SmartWallet` contract does not contain any `selfdestruct` or `suicide` functions, which means it cannot be directly self-destructed.
   - However, there is a potential issue with the `removeOwnerAtIndex` function, as mentioned earlier. If all owners are removed, the contract could become inaccessible and effectively unusable.
   - To mitigate this risk, it is recommended to implement a check in the `removeOwnerAtIndex` function to ensure that at least one owner remains after the removal.

4. Recommendations:
   a. Implement a check in the `removeOwnerAtIndex` function to prevent removing the last owner and leaving the contract inaccessible.
   b. Consider adding additional checks in the `upgradeToAndCall` function to verify that the `newImplementation` address is a valid and trusted contract.
   c. Thoroughly review and audit the `WebAuthnSol` contract to ensure its security and correctness.
   d. Regularly monitor and update the external contracts and libraries used in the `SmartWallet` contract to ensure they remain secure and up to date.

5. Conclusion

The Smart Wallet codebase demonstrates a well-structured design and follows best practices in many aspects. However, there are several areas for improvement, particularly in input validation, error handling, and centralization risks.

The upgradability mechanism and the concentration of power among owners pose significant risks that need to be carefully managed. Implementing strict access controls, multi-signature requirements, and time-locks can help mitigate these risks.

### Time spent:
37 hours