# Presentation Slide Outline

Target length: **8 slides**  
Presentation context: Capstone project presentation and Demo Day.

---

### Slide 1: TxSignX — A Bitcoin Transaction Explorer & Pre-Signing Security Analyzer

- **Main message**: Bringing deep transaction exploration and deterministic policy analysis into the pre-signing workflow.
- **Bullets**:
  - Open-source Rust tool for consensus transaction inspection and security preflight
  - Capstone Category: **Transaction Explorer**
  - Core Tagline: *Inspect. Verify. Sign with Confidence.*
  - Available across Web application, HTTP API, and Command-Line Interface
- **Suggested visual**: TxSignX Web application interface alongside terminal ASCII banner and code snippet.
- **Speaker note**: *"Today I am presenting TxSignX. Our goal is to give Bitcoin users and developers complete visibility and deterministic security checks before they ever commit a cryptographic signature."*

---

### Slide 2: The Signing Problem: Blind Signing in Bitcoin

- **Main message**: Signing is an irreversible cryptographic commitment, yet users are frequently forced to sign blindly.
- **Bullets**:
  - Wallets often hide critical script details, sighash flags, and fee mechanics
  - Malicious malware or coordinator bugs can inflate fees or hijack change outputs
  - Traditional block explorers only inspect transactions *after* broadcast
  - Pre-signing verification is critical to prevent catastrophic fund loss
- **Suggested visual**: Side-by-side comparison: a simplified "Send" prompt hiding script details vs. a transaction showing hidden SIGHASH_NONE and excessive fees.
- **Speaker note**: *"Once you sign a Bitcoin transaction, your funds are committed. Today, most tools only show you what happened after broadcast. TxSignX brings explorer-grade visibility to the moment right before signing."*

---

### Slide 3: Transaction Explorer Capstone MVP

- **Main message**: Full alignment with official Rust for Bitcoin capstone requirements plus pre-signing security extensions.
- **Bullets**:
  - Consensus decoding: inputs, outputs, scripts, locktime, sequence, witness items
  - Script disassembly: safe opcode and push-byte extraction
  - Address derivation: standard P2PKH, P2SH, P2WPKH, P2WSH, P2TR (explicit network)
  - Virtual size, weight, and fee rate (`sat/vB`) calculations
  - Confirmation status and single-hop parent transaction prevout resolution
- **Suggested visual**: Feature matrix showing all Capstone MVP requirements checked green across Web, HTTP API, and CLI.
- **Speaker note**: *"TxSignX satisfies every core requirement of the Transaction Explorer capstone: from consensus decoding and opcode disassembly to address derivation, confirmation tracking, and prevout fee calculation."*

---

### Slide 4: Dual-Path Architecture

- **Main message**: Clean separation of derived consensus facts, optional contexts, deterministic policy rules, and external signing.
- **Bullets**:
  - `txsignx-core`: Decodes transactions and PSBTs via `rust-bitcoin`
  - Dual paths: Independent Transaction Explorer vs. Policy Preflight
  - Multi-interface delivery: Web application (React/Vite), HTTP API (Axum), and CLI
  - Server-side isolation: Bitcoin Core RPC credentials stay on the server; never enter the browser
  - `txsignx-policy`: 15 active deterministic rules; zero AI heuristics
  - External signer: Keys and signing remain strictly outside TxSignX
- **Suggested visual**: Architectural flow diagram from `architecture.md` highlighting the separation between core facts, policy, and signing.
- **Speaker note**: *"Our architecture is strictly modular. The Transaction Explorer can operate completely standalone offline or with Bitcoin Core. When policy preflight is requested, the same facts flow into our deterministic policy engine."*

---

### Slide 5: Live Explorer: Offline Hex & Node-Assisted Txid

- **Main message**: Honest, fail-closed transaction exploration with zero fabricated data.
- **Bullets**:
  - Offline mode: Disassembles opcodes; refuses to invent fees without prevouts
  - Explicit network: Addresses derived only when network is declared; no guessing
  - Txid mode: Queries local Bitcoin Core over loopback RPC (`127.0.0.1` / `[::1]`)
  - Prevout resolution: Single-hop parent lookup resolves input values and fee rates
  - Confirmation status: Explicit `confirmed` with count and block hash, or `unconfirmed / mempool`
- **Suggested visual**: Screenshot of Web Inspector in Raw mode side-by-side with Node-assisted TXID mode showing resolved fees.
- **Speaker note**: *"In our live demo, we show that offline raw inspection never fabricates fees. In txid mode, TxSignX safely queries Bitcoin Core, resolves parent inputs, and calculates exact sat/vB fee rates."*

---

### Slide 6: Pre-Signing Policy Engine: PASS / REVIEW / BLOCK

- **Main message**: Actionable, deterministic security guardrails before signing.
- **Bullets**:
  - **PASS (0)**: Clean evaluation; explicitly records unrun rules as 'Not Evaluated'
  - **REVIEW (2)**: Flags structural risks (e.g. `TG011` unusual sighash like SIGHASH_NONE)
  - **BLOCK (3)**: Catastrophic errors blocked (e.g. `TG002` excessive fee over threshold, `TG014` non-zero OP_RETURN)
  - 15 active rules; 2 deferred rules (`TG007` dust, `TG008` address reuse)
- **Suggested visual**: Tri-color badge diagram (Green PASS / Amber REVIEW / Red BLOCK) showing real fixture examples and web UI cards.
- **Speaker note**: *"Our policy engine uses a clear three-tier decision model. A minor risk triggers REVIEW; a fat-finger fee triggers BLOCK. And PASS never lies about checks that weren't run."*

---

### Slide 7: Hardened Security Boundaries & Verification

- **Main message**: Defense-in-depth implementation verified by rigorous automated testing.
- **Bullets**:
  - Strict loopback RPC only (`127.0.0.1` and `[::1]`); DNS hostnames prohibited
  - 5-second timeout, 8 KB header cap, 1 MiB response body limit
  - Zero private keys, seed handling, or mainnet broadcast
  - **345 Rust workspace tests** passing across 6 crates; 0 Clippy warnings; 0 RustSec vulnerabilities
  - **48 Web tests** passing; 0 npm vulnerabilities; clean linter and build
  - Automated isolated Regtest integration suite with clean teardown
- **Suggested visual**: Test summary dashboard showing `cargo test` 345 passed, `vitest` 48 passed, and audit/linter clean results.
- **Speaker note**: *"Security isn't an afterthought. We enforce literal loopback addressing, bounded buffers, zero secret leakage, and validate everything through 345 automated Rust tests and 48 web tests."*

---

### Slide 8: Summary and Future Work

- **Main message**: TxSignX delivers an auditable pre-signing explorer and policy engine for the Bitcoin ecosystem.
- **Bullets**:
  - Delivered: Complete Capstone Transaction Explorer MVP across Web, API, and CLI
  - Pre-signing policy layer: 15 active deterministic security rules
  - Next steps: PSBT v2 ([BIP 370](https://bips.dev/370/)) support and hardware wallet companion workflows
  - Open source: Rust engine, React web interface, and full documentation available on GitHub
  - Tagline: *Inspect. Verify. Sign with Confidence.*
- **Suggested visual**: Final slide with GitHub repository links and key project metadata.
- **Speaker note**: *"TxSignX bridges the gap between raw consensus data and user security. By making transactions transparent before signing, we can eliminate costly mistakes. Thank you!"*
