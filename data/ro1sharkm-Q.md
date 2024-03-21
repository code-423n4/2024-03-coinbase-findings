## [QA-1] Hardcoded EntryPoint Address

https://github.com/code-423n4/2024-03-coinbase/blob/e0573369b865d47fed778de00a7b6df65ab1744e/src/MagicSpend/MagicSpend.sol#L304

The `entryPoint` address, which is essential for interacting with the ERC4337 standard and enabling account abstraction is hardcoded within the smart contract. While this  simplifies interactions, it restricts the contract's ability to adapt to new developments or entry point upgrades within the Ethereum ecosystem.

Make the EntryPoint address configurable. This can be achieved through a  a setter function.