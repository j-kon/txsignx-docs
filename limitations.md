# Limitations and future work

TxSignX is development software. Its local checks and deterministic policy reports do not constitute an independent audit, production-readiness certification or guarantee that a transaction is safe to sign.

## Current limits

- PSBT v0 only; no PSBT v2.
- No signing, finalization, seed handling, private-key custody or WIF import.
- No broadcast through the API; the existing advanced CLI broadcast path is Regtest-only.
- No network inference from transaction data, invented input amounts or invented fees.
- Supplied PSBT metadata is not independently authenticated without appropriate trusted context; cryptographic signature correctness is not verified.
- Descriptor matching is bounded; no full wallet synchronization, address history or persistent wallet database.
- Node observations depend on the configured Core node and its current state; offline or hosted demos cannot pretend to provide live node results.
- No hardware-wallet integration, Electrum, Esplora, BDK FFI, Flutter SDK or mobile application.
- No AI security decisions, LLM risk scoring, cloud accounts or production hosted wallet service.
- Local HTTP controls do not replace authentication and operational hardening for a public service.

## Possible future scope

Future work requires a separately reviewed design and tests; none is claimed as implemented by M6. Candidates include explicit relay-policy assumptions for TG007 (Dust Output), trustworthy wallet history for TG008 (Address Reuse), richer coverage explanations and independently reviewed integrations with external signers. Any broader deployment would need a threat model, authentication and privacy/operational review.

The current product boundary remains: inspect facts, verify available context, explain Rust policy findings, leave signing decisions external.
