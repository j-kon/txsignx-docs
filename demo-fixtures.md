# Demonstration Fixtures

This document catalogues the reproducible public test fixtures used for TxSignX capstone demonstrations and automated testing. All fixtures are synthetic, deterministic, and safe for public presentation. Zero private keys, seed phrases, or spendable mainnet funds are used.

---

## 1. Raw transaction fixture (`legacy.hex`)

- **File location**: `crates/txsignx-core/tests/fixtures/legacy.hex`
- **Format**: Consensus-serialized Bitcoin transaction hex (ASCII string).
- **Node context required**: **No** (operates 100% offline).
- **Purpose**: Demonstrates consensus decoding, size calculation, script disassembly, and address derivation.
- **Transaction facts**:
  - Version: 2
  - Locktime: 42
  - Inputs: 1 (outpoint `1111111111111111111111111111111111111111111111111111111111111111:1`)
  - Outputs: 2 (Output 0: 100,000 sats P2PKH; Output 1: 50,000 sats P2WPKH)
  - Size / weight / vsize: 118 bytes / 472 WU / 118 vB
  - Explicit RBF: Yes (Sequence: `0xfffffffd` < `0xfffffffe`)
  - SegWit: No (legacy encoding)
- **Expected demo behavior**:
  - `txsignx tx inspect <HEX>`: Disassembles scriptSig (`PUSHBYTES_1 51`) and scriptPubKey (`OP_DUP OP_HASH160 PUSHBYTES_20 ... OP_EQUALVERIFY OP_CHECKSIG`). Reports `Fee: unavailable without prevout context`.
  - `txsignx tx inspect <HEX> --network bitcoin`: Renders standard addresses (`147Us9aEq2PvBC5wobBJw1yEpQEbPKzssA` and `bc1qxven...`).
- **What it demonstrates**: Accurate Bitcoin consensus decoding, opcode disassembly, explicit network requirement for address derivation, and refusal to invent fee data without prevouts.

---

## 2. Policy PASS PSBT fixture (`pass.b64`)

- **File location**: `fixtures/policy/pass.b64`
- **Format**: Standard BIP 174 PSBT v0 in base64.
- **Node context required**: **No** (uses supplied UTXO context in the PSBT).
- **Purpose**: Demonstrates a clean transaction preflight passing all evaluated active development policies.
- **Transaction facts**:
  - Total input: 101,000 sats (supplied prevout metadata)
  - Total output: 100,000 sats
  - Fee: 1,000 sats (1.0% fee share)
  - Sighash: Default / SIGHASH_ALL
- **Expected demo behavior**:
  - `txsignx psbt preflight --file fixtures/policy/pass.b64`:
    - Decision: **PASS** (Exit code `0`)
    - Findings: 0 critical/high/medium findings
    - Coverage: Fact-based rules evaluated; wallet and node rules categorized as **Not Evaluated** with explicit reason recorded.
- **What it demonstrates**: Successful baseline preflight, checked fee arithmetic, and transparent coverage reporting.

---

## 3. Policy REVIEW PSBT fixture (`unusual-sighash.b64`)

- **File location**: `fixtures/policy/unusual-sighash.b64`
- **Format**: Standard BIP 174 PSBT v0 in base64.
- **Node context required**: **No**.
- **Purpose**: Demonstrates warning on dangerous or non-standard signature commitment flags.
- **Transaction facts**:
  - Input specifies explicit numeric sighash `2` (`SIGHASH_NONE`).
- **Expected demo behavior**:
  - `txsignx psbt preflight --file fixtures/policy/unusual-sighash.b64`:
    - Decision: **REVIEW** (Exit code `2`)
    - Finding: `TG011` (Unusual Sighash Type)
    - Severity: `HIGH`
    - Recommendation: Verify whether `SIGHASH_NONE` is intentional, as outputs are uncommitted and can be altered by third parties.
- **What it demonstrates**: Detection of non-standard signature hashes without falsely rejecting valid custom workflows.

---

## 4. Policy BLOCK PSBT fixture (`absolute-fee.b64`)

- **File location**: `fixtures/policy/absolute-fee.b64`
- **Format**: Standard BIP 174 PSBT v0 in base64.
- **Node context required**: **No**.
- **Purpose**: Demonstrates hard blocking of transactions with excessive fees that breach configured thresholds.
- **Transaction facts**:
  - Absolute fee: 200,000 sats.
  - Configured maximum absolute fee limit: 100,000 sats (default).
- **Expected demo behavior**:
  - `txsignx psbt preflight --file fixtures/policy/absolute-fee.b64`:
    - Decision: **BLOCK** (Exit code `3`)
    - Finding: `TG002` (Excessive Absolute Fee)
    - Severity: `CRITICAL`
    - Finding evidence: Fee of 200,000 sats exceeds configured threshold of 100,000 sats.
- **What it demonstrates**: Hard guardrails against catastrophic fee overpayments before cryptographic signing.

---

## 5. Dynamic isolated Regtest suite (`verify-explorer-regtest.py`)

- **Script location**: `scripts/verify-explorer-regtest.py`
- **Node context required**: **Yes** (launches temporary local `bitcoind` in regtest mode).
- **Purpose**: Demonstrates live `--txid` inspection, confirmation status resolution, and single-hop previous output lookup without reliance on external public chain data.
- **Environment**:
  - Isolated temporary directory in system temp (`/tmp/txsignx-explorer-regtest-...`).
  - Flags: `regtest=1`, `networkactive=0`, `txindex=1`, `fallbackfee=0.0001`.
- **Generated transactions**:
  - **Transaction A (Confirmed)**: Funded from mined block rewards, confirmed with 1 block.
    - Demonstrates: `status: confirmed`, `confirmations: 1`, `block_hash`, resolved prevouts, exact fee (141 sats), and fee rate (`1.00 sat/vB`).
  - **Transaction B (Unconfirmed)**: Broadcast to local mempool without mining a block.
    - Demonstrates: `status: unconfirmed / mempool`, resolved prevouts, fee (141 sats), and fee rate (`1.00 sat/vB`).
- **Teardown**: Automatically stops `bitcoind` and wipes the temporary datadir.
