# ZRC-20 Auditor Guide: Viewing Key Authorization and Usage

## Overview

This document provides guidance for auditors, regulators, and integrators on how to use the ZRC-20 viewing key system to perform compliance checks, transaction reviews, and user-level audits. It outlines the mechanism by which users grant access to their encrypted transaction data and how auditors can securely use this access.

## 1. Key Concepts

### 1.1 Viewing Key

A **Viewing Key** is a cryptographic key that allows a third party to:

- Decrypt Notes (token commitments) associated with a user.
- View inbound and outbound transactions (nullifiers and new commitments).
- Reconstruct the balance and transaction history of the authorized user.

The viewing key **does not** enable spending or access to the user's private spend key.

### 1.2 Grant/Revoke Access

ZRC-20 token holders can use on-chain functions to **grant** or **revoke** visibility rights to specific Ethereum addresses. The actual key exchange is handled **off-chain**, while the access permissions are registered **on-chain**.

## 2. Auditor Workflow

### Step 1: Access Request

- The auditor submits a request to the user (e.g., a DAO multisig or private token holder) to view their transaction history.

### Step 2: Granting Authorization

- The user calls the ZRC-20 contract function:

```solidity
function grantViewAccess(address viewer) external;
```

- This function logs an `ViewAccessGranted` event on-chain. It signals to DApps that the user has permitted `viewer` to access their transaction data.

### Step 3: Off-Chain Key Transmission

- The user securely shares their **Viewing Key** with the auditor via an encrypted channel (e.g., Signal, PGP, or enterprise key exchange).

### Step 4: Auditing Transactions

- The auditor imports the Viewing Key into a compatible wallet or analysis tool.
- The tool scans historical commitment events and decrypts the user’s Notes.
- The auditor can verify:
  - Amounts received and sent
  - Transaction timing
  - Counterparties (if disclosed via tagging or metadata)

### Step 5: (Optional) Revocation

- The user can call:

```solidity
function revokeViewAccess(address viewer) external;
```

- This revokes visibility rights and logs an `ViewAccessRevoked` event.

## 3. Security Model

- Viewing Key does not allow spending or note creation.
- On-chain permission mapping is public; viewing key data is not.
- Auditors must handle viewing keys securely to prevent privacy leakage.
- Users can rotate their viewing key (e.g., per audit period) to enforce forward secrecy.

## 4. Integration Recommendations

- **Wallet Developers:**

  - Support importing viewing keys for read-only audit mode.
  - Enable scan tools with decrypt+export functions for external auditors.

- **Auditor Tools:**

  - Implement standard interface for viewing key format.
  - Display decrypted balance and note flow over time.

- **Compliance Frameworks:**

  - ZRC-20 allows verifiable audit trails without exposing full public state.
  - Viewing key logs can be used to demonstrate good-faith compliance.

## 5. Example Use Cases

| Use Case                 | Description                                                                                       |
| ------------------------ | ------------------------------------------------------------------------------------------------- |
| DAO Treasury Audit       | DAO shares a viewing key to allow token flow review by external governance reviewers.             |
| Private Enterprise Audit | A company issues payroll via ZRC-20 and grants periodic viewing rights to regulators or partners. |
| Exchange Risk Assessment | An exchange receives viewing access from large holders to verify origin of funds.                 |

## 6. Limitations

- Auditors cannot independently discover users—only those who explicitly authorize them.
- Viewing key leakage compromises user privacy.
- Historical Notes must be retained by indexers to be audit-accessible.

## 7. License

This document is released under CC0. Use freely in compliance toolchains, wallets, and regulatory submissions.

