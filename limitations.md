# Limitations and Future Work

TxSignX is a developer-focused security tool and transaction explorer. Its local checks and deterministic policy reports do not constitute a commercial security audit, production-readiness certification, or guarantee that a transaction is spendable or consensus-valid.

## Current boundaries and limitations

### 1. Transaction inspection and explorer boundaries
- **Raw transaction network agnostic**: A raw consensus transaction contains no network identifier (mainnet, testnet, signet, regtest). Address derivation and chain comparison require an explicit `--network` parameter. TxSignX never infers the network or fabricates addresses.
- **Omission of previous-output values in raw hex**: Raw serialized transactions identify outpoints (`txid:vout`) but omit their satoshi values and previous scriptPubKeys. Input totals, absolute fees, and virtual size fee rates (`sat/vB`) cannot be calculated without prevout context. When prevouts are unavailable, fees are explicitly reported as unavailable; TxSignX never invents fee values.
- **Historical transaction index requirement (`txindex=1`)**: In txid mode, arbitrary historical transaction lookups via Bitcoin Core require the node to be run with `txindex=1`. Otherwise, Bitcoin Core can only resolve unconfirmed mempool transactions or wallet-relevant transactions.
- **Node dependency**: Node observations reflect the configured Bitcoin Core node's active chain, mempool, and block tip. A stale, desynchronized, or reorged node will report facts reflecting that local state.
- **Loopback transport limits**: Bitcoin Core RPC access is strictly restricted to literal loopback endpoints (`http://127.0.0.1:PORT` or `http://[::1]:PORT`). The transport enforces a 5-second timeout, 8,192-byte header ceiling, 1,024-byte status line ceiling, and a strict 1 MiB (1,048,576-byte) response body limit. Transactions whose verbose RPC representation exceeds 1 MiB are rejected with an RPC error.
- **Single-hop prevout resolution**: Previous output resolution for txid inspection fetches only the immediate parent transaction outputs spending into the target transaction. It does not recursively crawl historical transaction graphs. Input resolution is capped at 256 inputs (`MAX_NODE_INPUTS`).

### 2. Pre-signing policy and format boundaries
- **PSBT format**: Supports PSBT v0 ([BIP 174](https://bips.dev/174/)). PSBT v2 ([BIP 370](https://bips.dev/370/)) is unsupported and rejected with a typed error.
- **Scope of PASS**: A policy decision of PASS strictly means: *"No evaluated active rule requires review or blocking."* It is not a guarantee of universal security or absence of risk. If wallet or node context is absent, the rules requiring those contexts are recorded as **Not Evaluated** rather than passed.
- **Deferred rules**: Reserved codes `TG007` (Dust Output) and `TG008` (Address Reuse) have no evaluator and are never evaluated; they must never be interpreted as passing.
- **Consensus vs. observation**: Successful consensus decoding by `rust-bitcoin` does not prove script spendability, cryptographic signature validity, or miner relay acceptance.

### 3. Explicit operational exclusions
- **Not a wallet**: TxSignX holds no private keys, generates no mnemonic seed phrases, imports no WIF keys, and manages no keychains.
- **Not a signer**: TxSignX does not sign transactions, produce signatures, or finalize PSBTs.
- **No mainnet broadcast**: TxSignX has no arbitrary mainnet broadcast API. The CLI provides an advanced broadcast command strictly gated to Regtest with preflight PASS requirements.
- **Deterministic engine, not AI**: TxSignX does not use machine learning, neural networks, or LLMs to score risks or determine policy outcomes. Decisions are 100% deterministic Rust rules.

## Future scope

Any expansion of scope requires dedicated design, formal threat modeling, and regression testing:
1. **PSBT v2 support**: Parsing and contextual validation for BIP 370 inputs and outputs.
2. **Relay policy context (TG007)**: Incorporating explicit dust thresholds and relay assumptions.
3. **Address history context (TG008)**: Controlled integration of trustworthy wallet address history to detect address reuse.
4. **Hardware wallet integrations**: Clean companion export pipelines for external signers (e.g. Coldcard, Jade, BitBox02, Ledger).
