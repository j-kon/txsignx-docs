# TxSignX Capstone

## Project topic

**TxSignX — A Bitcoin Transaction Explorer and Pre-Signing Security Analyzer**

## Category

**Transaction Explorer**

## Tagline

*Inspect. Verify. Sign with Confidence.*

---

## 1. Problem statement

In Bitcoin, cryptographic signing is an irreversible, binding commitment of value. Once a signature is produced and broadcast, any error—such as an excessive miner fee, a compromised change address, or an uncommitted sighash flag—results in permanent, unrecoverable loss of funds.

Modern Bitcoin wallets frequently force users to sign "blindly." User interfaces often present a high-level abstraction (e.g., "Send 0.05 BTC to Alice") while obscuring:
- Exact script bytecode and output types
- Previous-output values and calculated fee rates
- Signature hash commitments (e.g. `SIGHASH_NONE` or `SIGHASH_SINGLE`)
- Unconfirmed parent transaction dependencies and replaceability risks

Traditional block explorers allow deep inspection of Bitcoin transactions, but only **after** they have been confirmed or broadcast to the network. By that time, it is too late to prevent loss.

---

## 2. Solution

TxSignX bridges this critical gap by bringing explorer-grade inspection and deterministic security policy analysis into the **pre-signing workflow**, delivered across CLI, local HTTP API, and modern Web interfaces.

TxSignX provides:
1. **A full Transaction Explorer**: Inspects consensus-serialized raw transactions offline or queries Bitcoin Core by transaction ID (`txid`) across CLI, HTTP API, and Web (`txsignx-web`), extracting all consensus fields, disassembling scripts into opcodes, deriving addresses, and resolving parent previous outputs for authoritative fee calculation.
2. **Deterministic Pre-Signing Policy Preflight**: Evaluates structured transaction facts against configurable, rule-based policies before signing. It flags structural anomalies (REVIEW) and catches catastrophic errors (BLOCK) before any private key is engaged.

TxSignX is **not a wallet** and **never touches private keys**. It operates as an independent, auditable security layer that empowers developers and signers to inspect facts and verify policies.

---

## 3. What makes TxSignX different

| Traditional Block Explorers | Traditional Bitcoin Wallets | TxSignX |
| --- | --- | --- |
| Post-broadcast only | Pre-signing, but often blind | **Pre-signing explorer** |
| Requires external public servers | Minimal script or fee transparency | **Local, offline, or loopback-only** |
| Passive observation only | Focuses on key custody and signing | **Active deterministic policy evaluation** |
| Cloud-dependent | Monolithic trust | **Modular, read-only security layer** |

---

## 4. Transaction Explorer MVP mapping

TxSignX satisfies every official requirement of the Rust for Bitcoin capstone **Transaction Explorer** category:

| Capstone Requirement | Status | TxSignX Implementation Details |
| --- | --- | --- |
| **Accept raw transaction hex** | **Implemented** | Available across CLI (`txsignx tx inspect`), HTTP API (`POST /api/v1/transactions/inspect`), and Web (`txsignx-web` Raw tab); decoded via `rust-bitcoin` with defensive 4M weight unit limits. |
| **Accept txid from node** | **Implemented** | Available across CLI (`--txid`), HTTP API (`{"txid":"..."}`), and Web (`Transaction ID` tab) via loopback RPC connection to local Bitcoin Core. |
| **Decode core fields** | **Implemented** | Decodes version, inputs, outputs, previous outpoints (`txid:vout`), `scriptSig`, witness items, sequence, and locktime. |
| **Script classification** | **Implemented** | Identifies `P2PKH`, `P2SH`, `P2WPKH`, `P2WSH`, `P2TR`, `OP_RETURN`, and `Unknown` scripts. |
| **Address derivation** | **Implemented** | Derives standard addresses for recognized output scripts when explicit `--network` or web selector is provided; omits for OP_RETURN or nonstandard. |
| **Script disassembly** | **Implemented** | Disassembles `scriptSig` and `scriptPubKey` into opcodes and safe, bounded push data without script execution. |
| **Size / weight / vsize** | **Implemented** | Calculates serialized size in bytes, BIP 141 weight in Weight Units (WU), and virtual size in virtual bytes (vB). |
| **Fee calculation** | **Implemented** | Bounded single-hop lookup of parent transactions via Bitcoin Core resolves input prevout values and calculates exact fee. |
| **Fee rate calculation** | **Implemented** | Calculates exact `sat/vB` fee rate when fee and transaction vsize are known. |
| **SegWit detection** | **Implemented** | Detects witness presence and SegWit encoding. |
| **Explicit RBF detection** | **Implemented** | Checks BIP 125 explicit signaling (`nSequence < 0xfffffffe`) per-input and per-transaction. |
| **Confirmation status** | **Implemented** | Reports `confirmed` (with confirmation count and block hash) or `unconfirmed / mempool` when queried via Bitcoin Core. |
| **Multi-interface output** | **Implemented** | Human-readable terminal output, pure machine-readable `--json` pipeline support, local REST API, and polished Web interface. |
| **Pre-signing security extension** | **Implemented** | 15 deterministic policy rules evaluating fee limits, wallet change, script types, sighashes, and node discrepancies. |

---

## 5. System architecture

TxSignX uses a dual-path architecture where the Transaction Explorer operates independently, or feeds its facts into the policy engine:

```text
                 INPUT
                   │
        ┌──────────┴──────────┐
        │                     │
     RAW TX / PSBT           TXID
        │                     │
        │                Bitcoin Core
        │                     │
        └──────────┬──────────┘
                   ↓
             txsignx-core
       transaction / PSBT facts
                   │
       ┌───────────┼───────────┐
       │           │           │
   wallet ctx   node ctx    explorer
       │           │           │
       └──────┬────┘           │
              ↓                │
        txsignx-policy         │
              │                │
       PASS/REVIEW/BLOCK       │
              └───────┬────────┘
                      ↓
                  CLI / API
                      ↓
                    Web
                      ↓
                External signer
```

For full architectural diagrams and component boundaries, see [architecture.md](architecture.md).

---

## 6. Demonstration prerequisites

1. **Rust toolchain**: Stable Rust 1.85+ with `cargo`.
2. **Build CLI**:
   ```sh
   export DEVELOPER_DIR=/Library/Developer/CommandLineTools
   cargo build -p txsignx-cli
   ```
3. **Optional Bitcoin Core**: For live txid inspection, a local `bitcoind` binary (version 24+) installed on `PATH`. The automated regtest script manages daemon startup and teardown automatically.

---

## 7. Interactive demonstrations

### Demo 1: Offline raw transaction inspection
Inspect a consensus-serialized transaction hex string completely offline without network access:
```sh
./target/debug/txsignx tx inspect "$(cat crates/txsignx-core/tests/fixtures/legacy.hex)"
```
- **Observed facts**: TXID, wTXID, Version (2), Locktime (42), Size (118 bytes), Virtual size (118 vB), SegWit (no), Explicit RBF (yes).
- **Script disassembly**:
  - `scriptSig asm`: `PUSHBYTES_1 51`
  - `scriptPubKey asm`: `OP_DUP OP_HASH160 PUSHBYTES_20 2222222222222222222222222222222222222222 OP_EQUALVERIFY OP_CHECKSIG`
- **Fee explanation**: The report displays `Fee: unavailable without prevout context`. Raw Bitcoin transactions specify which outpoints are being spent, but omit their satoshi values. TxSignX refuses to guess or invent fee values.

### Demo 2: Explicit network address derivation
Raw transactions do not encode whether they belong to mainnet, testnet, or regtest. Adding `--network` enables address derivation:
```sh
./target/debug/txsignx tx inspect \
  "$(cat crates/txsignx-core/tests/fixtures/legacy.hex)" \
  --network bitcoin
```
- **Output 0 (P2PKH)**: `147Us9aEq2PvBC5wobBJw1yEpQEbPKzssA`
- **Output 1 (P2WPKH)**: `bc1qxvenxvenxvenxvenxvenxvenxvenxven2ymjt8`

### Demo 3: Node-assisted txid lookup (isolated Regtest)
Execute the automated regtest explorer verification suite:
```sh
python3 scripts/verify-explorer-regtest.py
```
- Launches an isolated temporary Bitcoin Core daemon with `networkactive=0`, `regtest=1`, `txindex=1`.
- **Confirmed Transaction**: Reports `Status: confirmed`, `Confirmations: 1`, block hash, resolved parent prevouts, exact input total (5,000,000,000 sats), fee (141 sats), and fee rate (`1.00 sat/vB`).
- **Unconfirmed Transaction**: Reports `Status: unconfirmed / mempool`, resolved prevouts, and exact fee rate.
- Automatically cleans up the temporary datadir.

### Demo 4: Pre-signing policy preflight (PSBT)
Demonstrates deterministic security policy evaluation over PSBTs:
1. **PASS (`pass.b64`)**:
   ```sh
   ./target/debug/txsignx psbt preflight --file fixtures/policy/pass.b64
   ```
   *Result*: Decision **PASS** (exit code `0`). Coverage report explicitly marks unrun wallet/node rules as **Not Evaluated**.
2. **REVIEW (`unusual-sighash.b64`)**:
   ```sh
   ./target/debug/txsignx psbt preflight --file fixtures/policy/unusual-sighash.b64
   ```
   *Result*: Decision **REVIEW** (exit code `2`). Rule `TG011` flags non-standard `SIGHASH_NONE` commitment.
3. **BLOCK (`absolute-fee.b64`)**:
   ```sh
   ./target/debug/txsignx psbt preflight --file fixtures/policy/absolute-fee.b64
   ```
   *Result*: Decision **BLOCK** (exit code `3`). Rule `TG002` catches 200,000 sat fee exceeding configured 100,000 sat limit.

---

## 8. Policy registry

Running `txsignx policy list` displays the active registry of 15 deterministic Rust rules:
- `TG001`: Wrong Network (Critical)
- `TG002`: Excessive Absolute Fee (Critical)
- `TG003`: Excessive Fee Percentage (Critical)
- `TG004`: Unknown Wallet Input (High)
- `TG005`: Unknown Change Output (High / Critical)
- `TG006`: Immature Coinbase Input (Critical)
- `TG009`: Invalid UTXO Context (Critical)
- `TG010`: Missing UTXO Context (High)
- `TG011`: Unusual Sighash Type (High)
- `TG012`: Unknown or Proprietary Metadata (Info)
- `TG013`: Unrecognized Script Type (Medium)
- `TG014`: Non-Zero OP_RETURN Value (Critical)
- `TG015`: Node UTXO Unavailable (Critical)
- `TG016`: Node Prevout Mismatch (Critical)
- `TG017`: Mempool Spend Conflict (High)
- *Reserved/Deferred*: `TG007` (Dust Output) and `TG008` (Address Reuse).

For complete rule descriptions and triggering semantics, see [policy-registry.md](policy-registry.md).

---

## 9. Presentation and demonstration resources

- [Demonstration Fixtures](demo-fixtures.md): Reference guide for all test fixtures.
- [Live Demonstration Script](demo-script.md): 5–7 minute script for live Demo Day presentation.
- [Video Recording Script](video-script.md): 3–4 minute script with precise commands and visual cues.
- [Presentation Outline](presentation-outline.md): 8-slide structured outline.
- [Empirical Verification Record](verification.md): Full record of unit tests, Clippy, audits, and CI runs.
- [Security Model](security-model.md): Detailed analysis of trust boundaries and decision semantics.
- [Limitations & Future Work](limitations.md): Explicit documentation of current operational boundaries.
