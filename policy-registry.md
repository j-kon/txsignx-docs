# Policy registry

This reference follows `txsignx-policy/src/registry.rs` and the rule implementations in the Rust repository. The live API exposes that Rust registry directly; frontend code must not maintain a competing rule implementation. There are **15 active** rules and **two deferred** codes.

| Code | Name | Trigger severity | Meaning and context |
| --- | --- | --- | --- |
| TG001 | Wrong Network | Critical | Explicit configured network differs from the node-reported chain; requires node context |
| TG002 | Excessive Absolute Fee | Critical | Available fee exceeds configured absolute limit |
| TG003 | Excessive Fee Percentage | Critical | Available fee divided by total input value exceeds configured basis-point limit |
| TG004 | Unknown Wallet Input | High | Valid resolved prevout does not match public wallet descriptors within the derivation window |
| TG005 | Unknown Change Output | High / Critical | Explicit expected change matches the external keychain (High), or neither keychain within the window (Critical) |
| TG006 | Immature Coinbase Input | Critical | Node reports coinbase with fewer than 100 confirmations |
| TG009 | Invalid UTXO Context | Critical | Supplied previous-output metadata is internally inconsistent |
| TG010 | Missing UTXO Context | High | Missing previous-output metadata prevents reliable fee evaluation; absence alone does not establish invalidity |
| TG011 | Unusual Sighash Type | High | Explicit numeric sighash differs from DEFAULT (0) or ALL (1); unusual commitments can be intentional |
| TG012 | Unknown or Proprietary Metadata | Info | Extension fields exist which TxSignX does not interpret; presence alone is not malicious |
| TG013 | Unrecognized Script Type | Medium | An output or valid resolved prevout has an unrecognized script template; custom scripts may be legitimate |
| TG014 | Non-Zero OP_RETURN Value | Critical | Positive value is assigned to a provably unspendable OP_RETURN output |
| TG015 | Node UTXO Unavailable | Critical | Outpoint is absent from the queried node's chain and mempool UTXO views |
| TG016 | Node Prevout Mismatch | Critical | Resolved PSBT prevout value or script differs from the node's report |
| TG017 | Mempool Spend Conflict | High | UTXO exists in the node's chain view but is absent from its mempool-aware view |

Defaults are 100,000 sats maximum absolute fee and 1,000 basis points (10%) maximum fee share of **total input value**. Equality does not trigger either fee rule. These are development policy defaults, not universal Bitcoin safety thresholds.

| Deferred code | Name | Missing scope |
| --- | --- | --- |
| TG007 | Dust Output | Explicit relay/dust assumptions or node policy context |
| TG008 | Address Reuse | Wallet address/history context |

Deferred codes have no evaluator or assigned severity. Do not display them as checks that passed. Catalog descriptions may retain milestone-specific wording from Rust; `active: false` is authoritative.

Severity and decision are related but distinct: a report with TG012 alone can be PASS with low risk. Coverage remains essential even when the finding list is empty; see [security model](security-model.md).
