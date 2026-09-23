# TxSignX Documentation

**TxSignX — A Bitcoin Transaction Explorer and Pre-Signing Security Analyzer**

*Inspect. Verify. Sign with Confidence.*

TxSignX is an open-source Bitcoin transaction explorer and deterministic pre-signing security analyzer written in Rust.

A traditional transaction explorer explains what a transaction contains on-chain. TxSignX brings that exploration capability into the pre-signing workflow, combining consensus transaction decoding with deterministic, rule-based security policy evaluation before cryptographic signatures are committed.

> [!NOTE]
> **What TxSignX is not:**
> TxSignX is **not a wallet**, **not a signer**, **not an AI decision engine**, and **not a custody product**. It holds no private keys, generates no seeds, and leaves signing decisions strictly to external signers.

---

## Capstone Documentation Directory

### Capstone & Demo Day Resources
- **[Capstone Overview](capstone.md)**: Full capstone presentation covering problem, solution, MVP requirement mapping, 4 interactive demos, and architecture.
- **[Demonstration Fixtures](demo-fixtures.md)**: Catalog of reproducible, synthetic test fixtures for offline and node-assisted demos.
- **[Live Demo Script](demo-script.md)**: Timed 5–7 minute script for live Demo Day presentation.
- **[Video Recording Script](video-script.md)**: 3–4 minute terminal recording script with exact visual cues and narration.
- **[Presentation Slide Outline](presentation-outline.md)**: 8-slide structured outline for presentation decks.
- **[Empirical Verification Record](verification.md)**: Complete test evidence from 345 Rust workspace tests, 48 web tests, Clippy, audits, and isolated Regtest integration runs.

### Technical & Architectural Specifications
- **[Architecture](architecture.md)**: Dual-path system architecture (Transaction Explorer vs. Preflight), Mermaid data flow, and component responsibilities.
- **[Security Model](security-model.md)**: Trust boundaries, deterministic Rust authority, and explicit PASS / REVIEW / BLOCK semantics.
- **[Policy Registry](policy-registry.md)**: Complete catalog of the 15 active deterministic security rules (`TG001`–`TG017`) and 2 deferred codes.
- **[User Workflows](user-workflow.md)**: Step-by-step guides for offline raw inspection, node-assisted txid lookup, and PSBT preflight.
- **[Local API Overview](api.md)**: Local HTTP service routes, loopback restrictions, and transport payload limits.
- **[Limitations & Scope](limitations.md)**: Explicit boundaries regarding consensus, network parameters, and transport limits.

---

## Repositories

| Repository | Description | Link |
| --- | --- | --- |
| **Rust Engine & CLI** | Core consensus decoder, explorer, node RPC, wallet matching, and policy engine | [github.com/j-kon/txsignx](https://github.com/j-kon/txsignx) |
| **Web Application** | React + TypeScript + Vite presentation interface | [github.com/j-kon/txsignx-web](https://github.com/j-kon/txsignx-web) |
| **Documentation** | Technical documentation, capstone specifications, and demo scripts | [github.com/j-kon/txsignx-docs](https://github.com/j-kon/txsignx-docs) |

---

## Responsible Demonstration Practices

All demonstrations and tests in this repository use public, synthetic test fixtures or temporary, isolated Regtest daemons. Never enter private keys, seed phrases, WIF strings, or production credentials into any command or interface.
