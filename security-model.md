# Security Model

TxSignX provides deterministic pre-signing transaction inspection and policy analysis. It is designed to verify transaction intent and uncover structural discrepancies before committing cryptographic signatures.

TxSignX does not authorize signing, guarantee universal transaction safety, or replace independent hardware wallet and full-node verification.

## Core security principles

1. **Deterministic Rust authority**: All transaction decoding, address derivation, fee calculations, and policy evaluations are executed exclusively in Rust (`txsignx-core`, `txsignx-node`, `txsignx-wallet`, `txsignx-policy`).
2. **Zero AI decision-making**: TxSignX does **NOT** use machine learning, neural networks, or LLMs to score risk or determine decisions. Policy outcomes (PASS, REVIEW, BLOCK) are 100% deterministic functions over exact transaction facts and configured limits.
3. **Passive presentation**: The web application and HTTP API are strictly presentation and transport layers. The browser never computes policy findings, recalculates risk levels, or overrides the Rust engine's decisions.
4. **External signing boundary**: Signing decisions and private key operations remain strictly external to TxSignX.

## Interpreting policy decisions and coverage

### Meaning of PASS, REVIEW, and BLOCK

| Decision | Exit code (CLI) | Definition |
| --- | --- | --- |
| **PASS** | `0` | No evaluated active rule requires review or blocking. (May include `INFO` findings). |
| **REVIEW** | `2` | One or more evaluated rules triggered a `HIGH` or `MEDIUM` severity finding requiring user confirmation. |
| **BLOCK** | `3` | One or more evaluated rules triggered a `CRITICAL` finding indicating serious discrepancy or dangerous parameters. |

> [!IMPORTANT]
> **PASS does not mean universally safe.**
> PASS strictly asserts that within the scope of evaluated rules and supplied context, no threshold was breached. A transaction evaluated without wallet or node context will list those respective rules as **Not Evaluated**, not passed.

### Explicit evaluation coverage groups

Every policy evaluation categorizes all 15 active rules into three explicit groups:
- **Evaluated**: The rule executed against complete, required context and produced its verdict (finding or no finding).
- **Partially Evaluated**: Partial context was available (e.g. some inputs were resolvable while others were absent).
- **Not Evaluated**: Essential context (wallet descriptors or Bitcoin Core connection) was omitted. The engine records an explicit reason for the omission.

Deferred rules (`TG007` and `TG008`) are metadata-only definitions; they have no evaluator and are never reported as evaluated or passed.

## Trust boundaries

### 1. Untrusted transaction input
All user-supplied hex, base64 PSBTs, and transaction IDs are treated as untrusted input. Consensus decoding is handled by `rust-bitcoin` with defensive resource bounds (max 8,000,000 hex characters, 4,000,000 weight units, 100,000 witness items). Memory allocation is bounded before parsing.

### 2. Node context: trusted observation, not absolute truth
Bitcoin Core is treated as a trusted local observer, not infallible consensus ground truth. Node responses can reflect local network partitions, unconfirmed mempool states, or configuration mismatches.
- **Fail-closed readiness**: If Bitcoin Core is in Initial Block Download (IBD) or headers/blocks are desynchronized, node-dependent checks fail closed with `NodeError::NodeNotReady`.
- **Exact chain binding**: The explicit requested `--network` is compared directly with `getblockchaininfo`. Any discrepancy triggers a `CRITICAL` finding (`TG001`) and fails closed.
- **Loopback isolation**: RPC communication is restricted to literal IP addresses (`127.0.0.1` or `[::1]`). Hostnames (including `localhost`) and DNS resolution are prohibited to eliminate DNS rebinding vulnerabilities.

### 3. Bounded wallet context
Wallet descriptors are strictly public (`xpub`, `tpub`, output descriptors). Private keys, seed phrases, and WIF strings are rejected on ingestion without echoing. Derivation window expansion is strictly bounded (default 1,000 addresses) to prevent denial-of-service via unbounded public key derivation.

### 4. Machine-readable purity
Machine-readable outputs (`--json`) omit all ANSI escapes, color formatting, and ASCII art banners. JSON payloads are structured, typed, and deterministic, ensuring reliable automated ingestion by scripts and CI/CD pipelines.

### 5. Secret sanitization
Credentials, authentication cookies, and private filesystem paths are never serialized into reports, echoed in errors, or logged to terminal buffers.
