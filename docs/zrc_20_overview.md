# ZRC-20: Native Privacy Fungible Token Standard

**Version:** 0.1  
**Status:** Draft  
**Type:** Standard Track  
**Category:** Token Standard  
**Created:** 2025-06-30  
**License:** CC0

## Abstract
ZRC-20 defines a new fungible token standard for Ethereum that supports native privacy by default. Unlike ERC-20, where balances and transactions are publicly visible on-chain, ZRC-20 ensures all token transfers and ownership details (amount, sender, receiver) are cryptographically hidden using zero-knowledge proofs (zk-SNARKs).

The standard includes optional features such as auditable viewing keys and a fair launch minting module. Token holders may authorize third parties to audit their transaction history by granting and revoking viewing access.

## Motivation
The transparent nature of Ethereum has created significant privacy challenges for users. Wallet balances, transaction histories, and financial relationships are publicly visible and subject to front-running, surveillance, and targeted attacks.

ZRC-20 aims to:

- Provide **private token transactions** similar to those in Zcash and Monero.
- Create a **standard interface** for wallets, DApps, and exchanges to support privacy tokens.
- Enable **opt-in auditable compliance** through viewing key authorization.
- Introduce a **fair launch mechanism** with fixed pricing and hard-capped supply.

## Design Principles
- **Privacy by default**: All balances and transfers are encrypted and only accessible to the owner.
- **Client-side processing**: The wallet performs proof generation, balance decryption, and scanning.
- **Selective transparency**: Users can share viewing keys for compliance or audits.
- **Minimal on-chain data**: Smart contracts only store cryptographic commitments and nullifiers.

## Key Concepts
- **Note**: An encrypted representation of a user's token balance.
- **Commitment**: A public cryptographic hash of a Note.
- **Nullifier**: A unique value used to invalidate a spent Note.
- **zk-SNARK Proof**: Zero-knowledge proof generated off-chain and verified on-chain.
- **Viewing Key**: A secret key used to decrypt a user’s transaction history.

## Core Features
### Privacy-preserving transfers
All transfers are executed using zk-SNARK proofs. The transaction input/output values and addresses remain hidden. The smart contract verifies the proof without learning any sensitive information.

### View authorization
Users can grant or revoke access to their viewing keys on-chain. The viewing key itself must be transmitted securely off-chain.

### Fair launch minting
Anyone can mint tokens by paying a fixed price using the `claim()` function. The maximum supply is immutable, preventing future inflation.

### Stateless balances
There is no `balanceOf` function. Wallets scan on-chain commitments and decrypt Notes locally to determine their own balance.

## Interface Overview
The `IERC_P_Mintable` interface includes:

- `claim(bytes receivingPublicKey)`
- `transfer(bytes transactionData)`
- `grantViewAccess(address viewer)`
- `revokeViewAccess(address viewer)`
- `isAuthorizedViewer(address owner, address viewer)`
- Metadata functions: `name()`, `symbol()`, `decimals()`, `totalSupply()`, `maxSupply()`

## Compatibility
ZRC-20 is not directly compatible with ERC-20 or ERC-721. Integration requires custom logic by wallets, exchanges, and DApps.

## Security Considerations
- **zk-SNARK circuit correctness** is critical. Any bug can lead to token inflation or theft.
- **Trusted setup** (e.g., Groth16) must be securely executed, or replaced with transparent setups like STARKs.
- **Viewing key leakage** may reveal complete transaction history.
- **Client security** is paramount—proofs and decryption occur entirely off-chain.
- **Nullifier tracking** prevents replay and double-spend attacks.

## Use Cases
- Confidential payments
- Privacy-preserving DAO payroll
- Compliance-friendly private accounting
- MEV-resistant trading

## License
This standard is released under CC0. It is free to use, fork, or modify without restriction.

