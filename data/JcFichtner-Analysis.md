## Smart Wallet: Overview

Overview of the Coinbase Smart Wallet and its features. Here are some of the key points:

- The Coinbase Smart Wallet is an ERC-4337 compliant smart contract wallet. This means that it can be used with any ERC-4337 compatible entry point and paymaster.

- The wallet supports multi-ownership, which means that multiple users can have control over the wallet. This can be useful for shared wallets or for security purposes.

- The wallet uses a "magic link" type of system to allow users to sponsor transaction fees. Users can pre-approve withdrawals from the MagicSpend contract by signing a WithdrawRequest off-chain. This allows for a more user-friendly experience, as users do not need to manually approve each transaction on-chain.

- The wallet is still in early development, but it has the potential to be a powerful and user-friendly tool for managing digital assets.

Overall, the Coinbase Smart Wallet is a promising new development in the world of smart contract wallets. It offers a number of features that make it more user-friendly and secure than traditional wallets. However, it is important to note that the wallet is still in early development and there may be some bugs or vulnerabilities that have not yet been discovered.

## Contracts Explanation (Scope)

###### MultiOwnable.sol

`MultiOwnable` is a smart contract that allows multiple entities to own and control it. Key points:

**1 - MultiOwnableStorage struct:** Stores owners' data, mapping indices to owner bytes and owner bytes to ownership status.

**2 - MultiOwnable contract:** Uses the storage layout for multiple ownership management.

**3 - Storage location:** Follows ERC-7201 for predictable storage location.

**4 - Errors:** Custom errors for unauthorized access, duplicate owners, invalid owner data.

**5 - Events:** AddOwner and RemoveOwner for ownership changes.

**6 - Modifiers:** onlyOwner to restrict function access to owners.

**7 - Owner management:** Functions to add/remove owners using addresses or public keys.

**8 - Owner checks:** Functions to check if an address or public key is an owner.

**9 - Initialization:** _initializeOwners to set initial owners during deployment.

**10 - Internal functions:** _addOwner, _addOwnerAtIndex, _checkOwner for ownership management logic.

**11 - Storage access:** _getMultiOwnableStorage to interact with the contract's storage.

In summary, the contract provides a flexible ownership model with support for both Ethereum addresses and public keys, ensuring secure and decentralized control.

## ERC1271.sol

`ERC1271.sol` is an abstract contract that implements the `ERC-1271` standard for off-chain signature validation, with additional protection against cross-account replay attacks. Here's a quick summary:

**1 - Anti Cross-Account-Replay:** It introduces a mechanism to prevent the same signature from being used across multiple accounts by incorporating the contract's address and chain ID into the hash that is signed.

**2 - EIP-712 Compliance:** It uses EIP-712 structured data hashing and signing for creating verifiable and readable messages on Ethereum.

**3 - Domain Separator:** It defines a domainSeparator function that creates a unique hash for the contract, including its name, version, chain ID, and address, to ensure signatures are contract-specific.

**4 - Signature Validation:** The isValidSignature function validates a given signature against a hash, returning a specific bytes4 value if successful or a failure code otherwise.

**5 - Replay-Safe Hash:** The replaySafeHash function wraps the original hash with the contract's domain information to create a unique hash that is resistant to replay attacks.

**6 - EIP-712 Domain Information:** The eip712Domain function provides information about the EIP-712 domain, which is used in creating the domain separator and replay-safe hashes.

**7 - Abstract Functions:** _domainNameAndVersion and _validateSignature are abstract functions that must be implemented by derived contracts to provide the domain's name and version, and to define the signature validation logic, respectively.

Overall, this contract serves as a template for creating secure, non-replayable signatures for authentication in smart contracts, ensuring that signatures are valid and specific to the contract instance and chain they were intended for.

###### CoinbaseSmartWalletFactory.sol

`CoinbaseSmartWalletFactory` is used to create and manage instances of CoinbaseSmartWallet accounts. Key points:

**1 - Immutable Implementation:** The contract has an immutable implementation address, set at construction, pointing to the ERC-4337 implementation used for deploying new accounts.

**2 - Constructor:** Initializes the implementation address.

**3 - Account Creation:** The createAccount function allows deploying a new CoinbaseSmartWallet with a set of owners and a nonce to allow for address uniqueness. It uses LibClone to create a minimal ERC1967 proxy pointing to the implementation.

**4 - Error Handling:** An OwnerRequired error is thrown if an attempt is made to create a wallet without owners.

**5 - Deterministic Address:** The getAddress function predicts the address of a wallet that would be created with given owners and nonce, without actually deploying it.

**6 - Initialization Code Hash:** The initCodeHash function provides the hash of the initialization code for the account proxy.

**7 - Salt Generation:** The _getSalt internal function generates a deterministic salt from the owners and nonce.

The contract leverages the ERC-4337 standard for account abstraction and uses deterministic deployment to ensure predictable addresses for the created wallets.

## CoinbaseSmartWallet.sol

`CoinbaseSmartWallet` is an ERC4337-compatible smart wallet. Key features and components include:

- Inherits from MultiOwnable, UUPSUpgradeable, Receiver, and ERC1271.
- Uses libraries and contracts from Solady and account abstraction interfaces.
- Implements signature verification and multi-ownership management.
- Provides functions for executing transactions (execute, executeBatch, executeWithoutChainIdValidation).
- Includes nonce management with a special key for cross-chain replayable transactions.
- Custom validateUserOp function for ERC-4337 user operation validation.
- Modifier payPrefund to handle transaction funding.
- Functions for wallet initialization and upgrade authorization.
- Implements ERC1271 for signature validation with support for ECDSA and WebAuthn.
- Contains checks for permissions and conditions using modifiers and custom errors.

## WebAuthn.sol

`WebAuthn` is a library for verifying WebAuthn Authentication Assertions, which is used for secure user authentication. Key points:

- It imports external libraries: FCL, Base64, and LibString.
- Defines a struct WebAuthnAuth to encapsulate authentication data.
- Contains constants for user presence (AUTH_DATA_FLAGS_UP), user verification (AUTH_DATA_FLAGS_UV), and a secp256r1 curve parameter.
- Uses a precompiled contract address (VERIFIER) for signature verification.
- The verify function checks the validity of an authentication assertion, including user presence, user verification, challenge integrity, and signature validity.
- It uses both a precompile (if available) and a fallback library (FCL) for signature verification.
- The code omits certain WebAuthn verification steps based on assumptions about the use case.

## FCL.sol

`FCL` is a library for verifying ECDSA signatures on the Ethereum blockchain. It's optimized for the secp256r1 curve (also known as prime256v1 or NIST P-256). Key points:

**1 - Constants:** Defines curve parameters, including the prime modulus (p), curve coefficients (a, b), base point (gx, gy), curve order (n), and precomputed values for optimization.

**2 - ecdsa_verify:** Main function to verify ECDSA signatures given the message hash, signature components (r, s), and public key coordinates (Qx, Qy).

**3 - ecAff_isOnCurve:** Checks if a point is on the curve.

**4 - FCL_nModInv and FCL_pModInv:** Calculate modular inverses using the ModExp precompile.

**5 - ecZZ_mulmuladd_S_asm:** Core function for scalar multiplication and addition in assembly, optimized for gas efficiency.

**6 - ecAff_add, ecZZ_Dbl, ecZZ_AddN:** Helper functions for point addition and doubling.

The code uses assembly for low-level optimizations and interacts with a precompiled contract for modular exponentiation. It's designed for efficiency and adherence to the ECDSA verification algorithm.

## MagicSpend.sol

`MagicSpend` is an implementation of an ERC-4337 Paymaster compatible with EntryPoint v0.6. The contract allows accounts to withdraw ETH funds and handles user operation validation and post-operation processing for account abstraction.

**Key components:**

**1 - WithdrawRequest struct:** Contains details for withdrawal requests, including a signature, asset (only ETH supported), amount, nonce, and expiry.

**2 - _withdrawableETH mapping:** Tracks ETH amounts that users can withdraw.

**3 - _nonceUsed mapping:** Prevents replay attacks by keeping track of used nonces.

**4 - Events:** MagicSpendWithdrawal for logging withdrawals.

**5 - Errors:** Various custom errors for handling invalid operations like InvalidSignature, Expired, InvalidNonce, etc.

**6 - Modifiers:** onlyEntryPoint ensures certain functions can only be called by the EntryPoint contract.

**7 - Functions:**
- Constructor to set initial owner.
- validatePaymasterUserOp: Validates user operations for ERC-4337.
- postOp: Handles post-operation state updates and fund transfers.
- withdrawGasExcess: Allows users to withdraw excess gas funds.
- withdraw: Allows withdrawing funds with a valid request.
- Owner functions: ownerWithdraw, entryPointDeposit, entryPointWithdraw, entryPointAddStake, entryPointUnlockStake, entryPointWithdrawStake.
- Signature validation: isValidWithdrawSignature and getHash.
- Nonce checking: nonceUsed.
- entryPoint: Returns the address of the EntryPoint contract.

**8 - Internal functions:**
- _validateRequest: Validates the withdrawal request against replay attacks and logs the withdrawal.

- _withdraw: Handles the actual transfer of funds.

In summary, MagicSpend is designed to facilitate ETH withdrawals and manage gas payments for user operations in an account abstraction context, ensuring security through signature verification and nonce management to prevent replay attacks. It also provides administrative functions for the contract owner to manage funds and interact with the EntryPoint contract.

## Scurity (Scope)

###### MultiOwnable.sol

Potential vulnerabilities in the MultiOwnable.sol contract:

- **Centralization risk:** The contract allows for multiple owners, but all owners have the same level of privileges. This means that any single owner can perform critical actions such as adding or removing other owners, updating the protocol fee recipient, and withdrawing funds. This centralization of power could be a security risk if one of the owners is compromised or acts maliciously.

- **Lack of granular access control:** The contract does not provide a way to assign different roles or permissions to different owners. This means that all owners have full access to all contract functions, which may not be desirable in all cases.

- **Potential for griefing:** The removeOwnerAtIndex function allows any owner to remove another owner from the contract. This could be used by a malicious owner to grief other owners by repeatedly removing them from the contract.

**Recommendations:**

- **Consider implementing a multi-signature scheme:** This would require multiple owners to approve critical actions, reducing the risk of a single owner being compromised.

- **Implement granular access control:** This would allow you to assign different roles and permissions to different owners, limiting the potential damage that a single compromised owner could cause.

- **Consider adding a timelock or other mechanism to delay critical actions:** This would give other owners time to react and prevent malicious actions from being executed.

###### CoinbaseSmartWallet.sol

Security Analysis of CoinbaseSmartWallet.sol

This contract implements an ERC-4337 compliant smart contract wallet with additional features such as multi-ownership and cross-account replay protection.

Here are some key observations about the contract's security:

- **Centralization risk:** Although the contract allows for multiple owners, all owners have the same level of privileges. This means that any single owner can perform critical actions such as adding or removing other owners, updating the protocol fee recipient, and withdrawing funds. This centralization of power could be a security risk if one of the owners is compromised or acts maliciously.

- **Potential for griefing:** The removeOwnerAtIndex function allows any owner to remove another owner from the contract. This could be used by a malicious owner to grief other owners by repeatedly removing them from the contract.

**Recommendations:**

- Consider implementing a multi-signature scheme to reduce the risk of a single owner being compromised.

- Implement granular access control to limit the potential damage that a single compromised owner could cause.

- Consider adding a timelock or other mechanism to delay critical actions and give other owners time to react.
























### Time spent:
30 hours