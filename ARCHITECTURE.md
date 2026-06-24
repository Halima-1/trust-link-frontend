# Architecture Overview

This document describes the current frontend architecture for TrustLink and how the main pieces connect during a typical escrow flow.

## Component Tree

```mermaid
flowchart TD
    A[app/] --> B[layout.tsx]
    A --> C[page.tsx]
    A --> D[dashboard/page.tsx]
    A --> E[payment/page.tsx]
    A --> F[track/[escrowId]/page.tsx]
    A --> G[admin/disputes/[id]/page.tsx]

    H[components/]
    H --> H1[providers/WalletProvider.tsx]
    H --> H2[escrow/EscrowCreateForm.tsx]
    H --> H3[escrow/TrackingTimeline.tsx]
    H --> H4[dashboard/VendorDashboardList.tsx]
    H --> H5[payment/PaymentSection.tsx]

    I[hooks/]
    I --> I1[useWallet.ts]
    I --> I2[useEscrow.ts]

    J[lib/]
    J --> J1[api.ts]
    J --> J2[escrowStore.ts]
    J --> J3[stellar/freighter.ts]
    J --> J4[explorer.ts]

    K[types/]
    K --> K1[escrow.ts]

    B --> H1
    C --> H2
    D --> H4
    E --> H5
    F --> H3
    H1 --> I1
    H2 --> I2
    H5 --> J1
    H3 --> J2
    I1 --> J3
```

## Data Flow

1. The user enters the app through the Next.js App Router under `app/`.
2. Route-level pages render domain components from `components/`.
3. Components use hooks in `hooks/` to read wallet state, escrow state, and tracking data.
4. Hooks and components call utility modules in `lib/` for API requests, Stellar interactions, and local state storage.
5. Shared contracts and response shapes are defined in `types/`.

## Main Responsibilities

- `app/` — route entry points and page composition.
- `components/` — UI sections, forms, provider wrappers, and page-specific views.
- `hooks/` — reusable state and side-effect orchestration for wallet and escrow logic.
- `lib/` — network access, storage helpers, and Stellar integration logic.
- `types/` — shared TypeScript definitions used across the frontend.

## Typical Escrow Flow

1. A vendor opens an escrow creation page and submits form data.
2. The page uses the escrow hook and API utilities to create the link or escrow record.
3. The buyer visits the payment or tracking page and connects a wallet.
4. Wallet actions are handled through `WalletProvider` and the Stellar helper layer.
5. Tracking and status updates are rendered from the escrow and timeline components.

This structure keeps page routes thin, UI components reusable, and wallet/escrow logic isolated for easier maintenance.

## Security Considerations

The frontend implements critical security patterns to prevent re-entrancy at the UI level, handle large integer precision for asset amounts, and ensure all sensitive operations are backed by cryptographic signatures. For a detailed breakdown, see the [Security Model documentation](docs/SECURITY_MODEL.md).

For a reference of numeric contract error codes and their resolutions, see the [Contract Error Reference](./docs/CONTRACT_ERRORS.md).
