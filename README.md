# TxSignX

**Bitcoin transaction security before signing.**

Inspect. Verify. Sign with Confidence.

TxSignX is a deterministic open-source pre-sign security engine for Bitcoin transactions and PSBTs. Not another wallet. A security layer for wallets.

Milestone 6 adds a local HTTP API and React interface around the Rust engine. This documentation describes that development scope; it is not an audit or a production-readiness claim. Actual test and integration evidence belongs in the Rust repository's `docs/milestone-6-verification.md`, when recorded.

## Read the documentation

- [Architecture](architecture.md): components, data flow and authority.
- [Security model](security-model.md): trust boundaries and what PASS means.
- [Policy registry](policy-registry.md): 15 active rules and two deferred rules.
- [API overview](api.md): local setup, contracts and limits.
- [User workflow](user-workflow.md): inspect, review coverage and export.
- [Capstone demo](capstone.md): reproducible walkthrough, presentation and video script.
- [Limitations and future work](limitations.md): explicit scope boundaries.

## Repositories

[Engine and API](https://github.com/j-kon/txsignx) · [React web app](https://github.com/j-kon/txsignx-web) · [Documentation](https://github.com/j-kon/txsignx-docs)

The local workspace contains these three independent repositories as siblings. `txsignx-brand/` is local only: do not initialize Git there, create a remote, or commit its assets. This repository contains Markdown and needs no build tooling.

Never commit credentials, cookie files, seed phrases, private keys, private descriptors or personal wallet data. Public descriptors can also reveal wallet activity; use only the supplied synthetic fixtures for demonstrations.
