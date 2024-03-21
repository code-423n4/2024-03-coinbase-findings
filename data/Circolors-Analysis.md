# Table of Contents
1. [Security Review Approach](#security-review-approach)
2. [Protocol Overview](#protocol-overview)
3. [Roles](#roles)
4. [Contract Architecture](#contract-architecture)
5. [Risks and Recommendations](#risks-and-recommendations)
6. [General Comments](#general-comments)

## Security Review Approach
When undertaking the review of the Coinbase Smart Wallet protocol the following steps were taken:
1. Go through the provided documentation as well as documentation for related projects (with a particular focus on ERC-4337)
2. Conduct a thorough manual review of the protocols codebase. Taking note of both potential issues and any parts of the codebase that are unclear.
3. Repeat steps 1 & 2 until a thorough understanding of the codebase has been acheived.
4. Create control flow diagrams of the key actions to reference back to where necessary. See [Contract Architecture](#contract-architecture)
5. Go through the notes of potential issues one by one until each are either validated or invalidated (either by manually breaking down the logic involved or writing edge cases tests in foundry).
6. Write up the findings as well as general comments and potential recommendations to the protocol team.

A total of approximately 30 hours was spent conducting the security review of this codebase over a 7 day period.

## Protocol Overview
The Coinbase Smart Wallet protocol is a combination of codebases that function together to create an ERC-4337 compliant smart account and paymaster contract. Two main distinctions set this protocol apart from other ERC-4337 implementations:
- [Cross Chain Signatures](#cross-chain-signatures)
- [Multiple Owner Types](#multiple-owner-types)

### Cross Chain Signatures
The codebase contains logic to allow cross chain replayable signatures, allowing users to perform admin functions (such as adding/removing owners or upgrading their contracts logic implementation) across all supported chains with a single signature. This is a move to improve a users experience when managing their account on multiple chains.

### Multiple Owner Types
The codebase supports users being able to give their account multiple different owners. Furthermore the owners of an account can be either EVM addresses or passkey (Secp256r1) public keys.

## Roles
Entities in the protocol fall primarily into one of three roles:

### Account Owner(s)
As a smart contract account protocol the main intended user is the owner(s) of the account. Owners can either be Ethereum address or a Secp256r1 passkey.

Owners can interact with their wallets in the following ways:
- Execute Transactions via EOA (single and batch)
- Execute Transactions via ERC-4337 EntryPoint
- Change Owners
- Upgrade account implementation

### MagicSpend Funder
This role will be fulfilled by Coinbase and will require they always have enough Ether in the contract to cover the `withdrawable` amounts users as signed off to use. If/when ERC20 tokens become supported assets, the same will apply them too.

### EntryPoint Transaction Batcher
The transaction batcher is responsible for taking the `UserOperations` and putting them on chain. This role may initially be fulfilled by Coinbase themselves but could also be done by any user with access to "mempool" of Coinbase users `UserOperations`.

## Contract Architecture
The protocol's code is separated into four distinct parts:
- [Smart Wallet](#smart-wallet)
- [Magic Spend](#magic-spend)
- [WebAuthnSol](#webauthnsol)
- [FreshCryptoLib](#freshcryptolib)

### Smart Wallet
Smart Wallet represents the key logic associated with the ERC-4337 compliant Smart Wallet, the multiple owner functionality and the ability to validate cross chain replayable signatures.

### Magic Spend
Magic Spend provides the key functionality of the codebases ERC-4337 paymaster implementation, which will allow users to cover their gas fees using their balances on the Coinbase exchange.

### WebAuthnSol
WebAuthnSol is a library for verifying WebAuthn assertations. In the context of this codebase it is used to verify the signature of an ERC-4337 EntryPoint transactions where the signer of that transactions is a Secp256r1 passkey.

### FreshCryptoLib
The FreshCryptoLib library's `ecdsa_verify` function is used by the WebAuthnSol library when verifying passkey signatures.

### Control flow from EntryPoint
The following gives a general overview of the logic flow when completing an ERC-4337 EntryPoint transaction on a users `CoinbaseSmartWallet` where gas is covered by the `MagicSpend` paymaster:
![tx overview](https://i.imgur.com/8mtrnER.png)

## Risks and Recommendations

### Issues with cross chain replayable signatures
The protocol has made the design choice to allow specific function calls to be permittable to be replayed across different EVM chains. This is a choice to improve user experience and allow them to update an account's ownership status or upgrade the account's implementation logic.

However the team should be aware of a number of pitfalls users may run into because of this. Below are a few examples:
- If a user makes any ownership change on a single chain it becomes very likely problems will occur when later attempting a cross chain replayable function such as `removeOwnerAtIndex` as there's no guarantee that the same owner will be at that index on all chains.
- If in the future Coinbase decides to integrate with a new EVM chain (Fantom, Blast, Zora, etc) it will be necessary to first get the newly deployed account into the same state and reach the same replayable nonce as currently exists on other chains. This would have to be done while avoiding the possibility malicious actors are able to replay previously signed transaction on this new chain (such as adding an owner that has since become compromised and removed from the accounts owner's on the other chains.

### Dangers of `address(0)` owners
The protocol team should be aware the Solady implementation of `isValidSignatureNow` does not return false in the event that the address passed to it is the zero address. This means that all signatures will be deemed valid if the zero address is an owner. This is an issue for this codebase for two reasons:
- The `CoinbaseSmartWallet` constructor currently sets the proxy implementation's owner to `address(0)`
- There are no checks when adding an owner that it is not the zero address being added.

## General Comments
### Centralisation Risks
The codebase appears to be sufficiently decentralised. The only ownership the protocol has over the contract is that it will be in charge of the `Magic Spend` paymaster contract. This seems like a necessary exception to make as they will be required to add/remove funds from the contract as well as move them to the EntryPoint contract, all of which will require some amount of access control.

### Code Complexity
The primary code complexity from this codebase is mostly unavoidable as it stems from it's integration with ERC-4337. However the codebase handles this integration well. Overall the codebase is concise and avoids ununecessary complication. 

### Documentation
The codebase includes complete NATSPEC comments as well as additional inline comments providing necessary context for certain aspects. On top of this the provided documentation explains the system well, however there is some incorrect information such as non-existant functions being listed as being able to skip chain id validation [here](https://github.com/code-423n4/2024-03-coinbase/tree/main/src/SmartWallet). Finally the Coinbase/Base team have provided user friendly guides explaining the protocol, in the original [announcement article](https://www.coinbase.com/blog/evolving-wallets-to-bring-a-billion-users-onchain) as well as the following [video](https://twitter.com/WilsonCusack/status/1764355750149710190).

### Test Coverage
The codebase has good test coverage, but has room for improvement. One example of this would be to add a more complete integration testing suite to ensure the contracts are working as intended with both the deployed `EntryPoint` instance and interacts as intended when other smart contract accounts are owners and vice versa (Gnosis Safe, ERC6551 token bound accounts). However as the contract are currently being actively used on Base Sepolia testnet, this may make up for some of the missing edge case coverage.

### Time spent:
30 hours