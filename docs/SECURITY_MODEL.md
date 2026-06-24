# Security Model - TrustLink

This document outlines the security architecture, threat model, and trust assumptions for the TrustLink platform.

## 1. Threat Model

### 1.1 Attack Surface
- **Public Contract Functions**: Any address can call public methods on the Soroban contracts.
- **Frontend State**: Local storage and session data can be targeted via XSS.
- **RPC Communication**: Traffic between the frontend and the Soroban RPC node.
- **Wallet Interactions**: The interface between the platform and the user's wallet extension.

### 1.2 Potential Attackers
- **Malicious Users**: Attempting to manipulate the escrow lifecycle (e.g., claiming delivery without shipping).
- **Network Adversaries**: Attempting MITM attacks on RPC calls.
- **Frontend Hijackers**: Injecting scripts to siphon funds or steal session data.

### 1.3 Threat Categories & Mitigations

#### 1.3.1 Re-entrancy
- **Risk**: A call-back mechanism that allows an attacker to re-enter a function before the previous execution completes, typically used for double-withdrawals in EVM.
- **Mitigation**: 
  - **Protocol Level**: Soroban contracts are designed to minimize re-entrancy risks through a different execution model than EVM.
  - **Frontend Protection**: The application implements optimistic locking and strict transaction state tracking. Once a transaction is submitted, the UI prevents duplicate submissions via the same wallet session until the transaction hash is resolved or the timeout is reached.

#### 1.3.2 Integer Overflow / Underflow
- **Risk**: Extreme values causing balance calculations to "wrap around" or lose precision.
- **Mitigation**:
  - **BigInt Implementation**: All asset amount calculations in the frontend use `BigInt` to handle the full precision of 128-bit and 256-bit Soroban integers without JavaScript's floating-point issues.
  - **Validation**: Strict boundary checks on all numerical inputs to ensure they correspond to valid asset denominations and balance limits.

#### 1.3.3 Auth Bypass
- **Risk**: Unauthorized users attempting to trigger actions reserved for specific roles (e.g., a Vendor attempting to `confirm_delivery`).
- **Mitigation**:
  - **Cryptographic Signatures**: The frontend only facilitates transaction building. All sensitive operations REQUIRE a valid cryptographic signature from the authorized Stellar address, which is verified on-chain by the Soroban contract.
  - **Session Integrity**: Component-level checks verify that the connected wallet address matches the expected role for the current escrow before displaying sensitive action buttons.

## 2. Trust Assumptions
- **Wallet Security**: We trust that the user's chosen wallet extension (e.g., Freighter) securely manages private keys.
- **RPC Integrity**: We assume the configured Stellar RPC provides accurate state information (verified where possible via transaction receipts).
- **Browser Sandbox**: We rely on the browser's Same-Origin Policy (SOP) to protect the frontend environment.

## 3. Invariants
- **Conservation of Funds**: Funds held in an escrow contract can only be moved to the Buyer (refund) or Vendor (release), or remain in the contract. They cannot be "lost" to an arbitrary third party.
- **Role Consistency**: The identity of the Buyer and Vendor for a specific escrow instance is fixed at creation and cannot be swapped.
- **Sequence Integrity**: An escrow cannot move to a "Completed" state without first being "Funded" and "Delivered".

## 4. Known Limitations
- **Finality Window**: Users must wait for Stellar ledger close (typically 5 seconds) for absolute finality.
- **Wallet Requirement**: The platform cannot function without a compatible browser wallet extension.
- **Dispute Resolution**: The current security model assumes a trusted mediator for dispute resolution (if configured).
