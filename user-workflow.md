# Inspect and review

1. Start the [local API and web app](api.md). Check which API is configured before entering data. Use public synthetic samples for a demonstration.
2. Open **Inspector** from Home. Choose PSBT or raw transaction. Paste input, upload a supported text file, or select a sample. No private keys or seeds are accepted.
3. Inspect the facts. For PSBTs, review transaction ID, signing state, fee status, input/output counts, explicit RBF signal, scripts, sighash metadata, metadata counts and UTXO consistency. Raw transaction facts have less context. **Unavailable** means missing information, never zero.
4. Run PSBT preflight. Use default policy limits or explicitly configure them. Optionally add public wallet descriptors, a network, a bounded derivation window and expected-change output indexes. Request node analysis only when the local API has a configured node.
5. Read the Rust decision and risk level. Inspect each finding's TG code, severity, location, evidence and recommendation. REVIEW and BLOCK are valid reports, not HTTP failures.
6. Read **Evaluated**, **Partially Evaluated** and **Not Evaluated** separately. Without a node, node rules cannot verify chain or mempool facts. PASS means: “No evaluated active rule requires review or blocking.”
7. Open **Policies** to inspect the live Rust registry, including active and deferred entries. A deferred rule is not a passed rule.
8. Copy or download JSON only when needed. Reports may contain transaction and wallet context; manage exported files and clipboard contents accordingly. Clear the form or inspect another transaction when finished.

Signing remains in an external signer. TxSignX has no active signing or broadcast action.

## Recover from errors

- **API unavailable:** start the local API, check its configured URL and allowed browser origin, then retry.
- **Invalid input:** verify that PSBT input is standard base64 PSBT v0 or that raw input is transaction hex; do not paste secrets to diagnose errors.
- **Too large:** choose an input within the published API limits. Upload checks do not override server checks.
- **Node unavailable:** inspect offline with clearly reduced coverage, or correct server-side node configuration. Do not paste RPC credentials into the browser.
- **Malformed response or timeout:** retain no invented result; retry after diagnosing the local service. A stale result must not be mistaken for the new submission.

The browser holds analysis inputs in memory and sends them only to the configured API. Persistent browser storage and URL parameters are not input storage mechanisms. See [security model](security-model.md).
