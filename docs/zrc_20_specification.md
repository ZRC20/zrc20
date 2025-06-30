# ZRC-20: Specification

## 1. Introduction

This document defines the full technical specification for the ZRC-20 token standard. ZRC-20 enables the creation of fungible tokens with native privacy properties, using zk-SNARKs to ensure that token balances and transactions are completely encrypted on-chain.

The standard includes core privacy functionality, an optional fair launch minting module, and a mechanism for selective transaction viewing via viewing keys.

## 2. Interface Definition

```solidity
// SPDX-License-Identifier: CC0-1.0
pragma solidity ^0.8.18;

interface IERC_P_Mintable {
    // --- Events ---
    event Transaction(bytes32[] noteCommitments);
    event ViewAccessGranted(address indexed owner, address indexed viewer);
    event ViewAccessRevoked(address indexed owner, address indexed viewer);

    // --- Metadata ---
    function name() external view returns (string memory);
    function symbol() external view returns (string memory);
    function decimals() external view returns (uint8);
    function totalSupply() external view returns (uint256);
    function maxSupply() external view returns (uint256);

    // --- Core ---
    function claim(bytes calldata receivingPublicKey) external payable;
    function transfer(bytes calldata transactionData) external;
    function grantViewAccess(address viewer) external;
    function revokeViewAccess(address viewer) external;
    function isAuthorizedViewer(address owner, address viewer) external view returns (bool);
}
```

## 3. Core Mechanisms

### 3.1 Notes and Commitments

- A **Note** represents an encrypted token unit.
- Each Note is converted to a **Commitment** (hash) that is stored on-chain.
- Commitments are appended to a Merkle Tree in the contract.

### 3.2 Nullifiers

- A **Nullifier** is a unique hash derived from a Note.
- Once a Note is spent, its Nullifier is published to prevent reuse (double-spending).

### 3.3 Zero-Knowledge Proofs

- Clients generate zk-SNARK proofs to validate transactions.
- Proofs demonstrate correct value conservation, nullifier uniqueness, and Note ownership—without revealing private data.

### 3.4 Transaction Format (transfer)

The `transfer(bytes calldata transactionData)` function accepts a serialized byte blob containing:

- zk-SNARK proof
- List of input nullifiers
- List of new note commitments
- Any auxiliary public parameters (e.g., Merkle root)

The format is left open to specific implementations, but must be verifiable by the contract.

## 4. Minting Module

### 4.1 claim(bytes receivingPublicKey)

- Anyone can mint ZRC-20 tokens by paying a fixed `MINT_PRICE`.
- The minted tokens are issued as encrypted Notes using the caller's provided public key.
- Minting is capped by an immutable `maxSupply` value.

**Note:** The receivingPublicKey is used to encrypt the Note on-chain.

## 5. Viewing Key System

### 5.1 Authorization

```solidity
function grantViewAccess(address viewer) external;
function revokeViewAccess(address viewer) external;
```

- These functions store a public authorization mapping on-chain.
- The actual viewing key must be delivered off-chain via secure means.

### 5.2 Verification

```solidity
function isAuthorizedViewer(address owner, address viewer) external view returns (bool);
```

- Returns true if the `viewer` is authorized by the `owner` to access their transaction data.

## 6. Supply and Metadata

### 6.1 totalSupply()

- Returns the total number of tokens minted so far (not burned or spent).

### 6.2 maxSupply()

- Returns the hard cap of total supply set during contract deployment.

### 6.3 Token Information

- `name()` returns the token’s name.
- `symbol()` returns the token’s symbol.
- `decimals()` returns the number of decimals (typically 18).

## 7. Design Assumptions

- The smart contract does not maintain address-based balances.
- All state (notes, nullifiers) is publicly verifiable but semantically hidden.
- Off-chain clients are required to:
  - Track Merkle tree updates
  - Generate and verify zk-proofs
  - Scan for and decrypt notes

## 8. Extensibility

- The ZRC-20 standard can be extended to support multi-asset notes.
- A permissioned minting model (e.g., KYC-gated mint) can be introduced.
- Viewing access can optionally include time-limited or scope-restricted keys.

## 9. Compliance Notes

- Viewing keys enable regulated entities to audit user transactions.
- The grant/revoke model ensures privacy by default with optional transparency.

## 10. Implementation Notes

- zk circuits (e.g., Groth16 or PLONK) must be audited and parameterized securely.
- Off-chain code must handle key management, scanning, and UI/UX for privacy.

## 11. Security Considerations

- zk-SNARK circuit correctness is essential to prevent forgery or inflation.
- Nullifiers must be unique and enforced on-chain.
- Clients must validate Merkle roots and proof inputs.
- Trusted setups must be handled securely or avoided entirely using STARKs or MPC.

## 12. License

This specification is released under CC0 and may be used freely.

