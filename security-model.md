# Security model

TxSignX helps a user review transaction intent before signing. It does not authorize signing, prove universal safety, or replace independent wallet and node verification.

## Interpret the result with its coverage

**PASS: “No evaluated active rule requires review or blocking.”**

The Rust policy engine maps critical findings to BLOCK, high or medium findings to REVIEW, and no findings or informational-only findings to PASS. PASS can therefore contain an informational finding. HTTP success is independent of the policy decision: PASS, REVIEW and BLOCK are all successful evaluations.

Read `rule_evaluations` alongside every decision:

- **Evaluated:** the engine ran the rule under its current semantics.
- **Partially evaluated:** some input context was usable and some was unavailable.
- **Not evaluated:** necessary wallet, expected-change or node context was absent or unusable; read the engine's reason.

The legacy `evaluated_rules` list also includes partially evaluated rules. Use `rule_evaluations` for the three separate coverage groups. Some fact-based rules run and emit no finding when the needed fact is unavailable; “evaluated” must not be read as a guarantee that every imaginable check occurred. Deferred rules have no evaluator.

## Trust boundaries

**PSBT data is untrusted.** Internally consistent UTXO metadata is still supplied context, not independently authenticated chain data. An available fee is calculated from that context. Raw transactions alone do not provide input values or an inferable network. Signature presence and signing state do not verify cryptographic signature correctness.

**Wallet context is bounded.** Public descriptors identify scripts in the configured derivation window. No match within that window does not establish that an input belongs to someone else. Collaborative transactions can legitimately trigger TG004. Change intent must be declared with output indexes; TxSignX does not infer it from amount, position or appearance. Public descriptors remain privacy-sensitive.

**Node context is trusted context, not consensus proof.** The local Core node can be stale, compromised or configured for another chain. Results describe its current chain and mempool views and can change. TG001 compares the explicitly configured network with the node-reported chain; it does not derive a network from the transaction. TG017 reflects that node's mempool view and can be intentional in replacement workflows. Cookie authentication and literal-loopback RPC restrict access; they do not make node answers infallible. No node means node rules are not evaluated.

**The API is a local service.** Default binding is loopback. Request body, output, concurrency and time limits bound processing. Explicit CORS origins and Host checks reduce browser-origin attacks; CORS is not authentication against local processes. External binding requires deliberate startup configuration and a warning, and does not turn this into a production hosting service. Node URL and cookie-file configuration belong only to server startup. Errors are sanitized and input bodies, descriptors and credentials must not be logged.

**The browser presents Rust results.** Inputs go only to the configured API. PSBTs, transaction hex and descriptors must not enter persistent browser storage, URLs, analytics or console logs. Copying or downloading a report is an explicit user action and creates a sensitive artifact outside the app's memory. Browser extensions, the operating system and clipboard are outside TxSignX's protection.

See [API controls](api.md) and [limitations](limitations.md). Development verification is not an independent security audit.
