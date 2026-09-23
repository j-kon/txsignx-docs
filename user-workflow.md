# User Workflows

TxSignX provides comprehensive transaction inspection and pre-signing analysis across both its web application interface and command-line interface. Workflows fall into three primary categories: **offline raw transaction inspection**, **node-assisted transaction exploration (txid lookup)**, and **pre-signing policy preflight (PSBT v0)**.

---

## Web application workflows

The TxSignX web application provides a browser-based presentation interface for the TxSignX HTTP API (`txsignx-api`).

### A. Raw transaction explorer (offline / consensus decoding)

Used to decode, disassemble, and inspect consensus-serialized transaction hex.

1. Navigate to the TxSignX Web Inspector and select the **Raw Transaction** tab.
2. Paste the serialized transaction hex into the input field.
3. Select an optional network from the network dropdown:
   - **No network** (default): Address derivation is omitted. Consensus fields, virtual sizes, input sequence numbers, and script disassemblies are fully displayed.
   - **Explicit network**: Choose from `bitcoin` (mainnet), `testnet`, `testnet4`, `signet`, or `regtest` to derive standard P2PKH, P2SH, P2WPKH, P2WSH, or P2TR addresses.
4. Click **Inspect Transaction**.
5. Review the factual report:
   - **Consensus parameters**: Version, locktime, weight, size, and virtual size (`vsize`).
   - **Transaction identifiers**: TXID and SegWit wTXID.
   - **Inputs & RBF**: Outpoints, sequence numbers, and explicit BIP 125 RBF signaling (`nSequence < 0xFFFFFFFE`).
   - **Script disassembly**: `scriptSig` and `scriptPubKey` disassembled into human-readable opcodes and push bytes.
   - **Fee status**: Correctly marked as **unavailable** because raw Bitcoin transactions omit previous output values; TxSignX never guesses fees.

### B. Transaction ID explorer (node-assisted lookup)

Used to inspect a confirmed or mempool transaction by its 64-character hex TXID via a server-configured Bitcoin Core node.

1. Select the **Transaction ID** tab in the Web Inspector.
2. Paste the 64-character transaction hash (`txid`).
3. Click **Inspect Transaction**.
4. **Security boundary & credential isolation**:
   - The browser sends only `{ "txid": "<TXID>" }` to the TxSignX HTTP API.
   - Bitcoin Core RPC credentials (URL, port, RPC username/password, or `.cookie` path) are configured exclusively on the server.
   - The browser never prompts for, stores, or transmits node credentials. No credentials or transaction data are persisted in `localStorage`, `sessionStorage`, or `IndexedDB`.
5. Review chain and prevout context:
   - **Confirmation status**: Displays `confirmed` with confirmation count and block hash, or `unconfirmed / mempool`.
   - **Resolved previous outputs**: Single-hop parent transaction lookup resolves input amounts and locking scripts.
   - **Authoritative fees**: Calculated exact fee in satoshis and virtual size fee rate in `sat/vB`.
6. **No-node fallback**: If the TxSignX API server was started without `--node-url` / `--rpc-cookie-file`, the API returns a typed `node_not_configured` error. The web interface displays a clear informational message explaining that node configuration is required for TXID lookup.

### C. PSBT v0 pre-signing preflight (security policy analyzer)

Used to analyze unsigned or partially signed BIP 174 PSBT v0 transactions before signing.

1. Select the **PSBT** tab in the Web Inspector.
2. Paste the base64-encoded PSBT string.
3. Click **Analyze PSBT**.
4. Review the deterministic policy report:
   - **Verdict banner**: **PASS**, **REVIEW**, or **BLOCK**.
   - **Coverage breakdown**: Explicit listing of active rules categorized into **Evaluated**, **Partially Evaluated**, and **Not Evaluated** (with transparent reasons for omitted context).
   - **Structured findings**: Any triggered rules (`TG001`–`TG017`) display with severity badges (`CRITICAL`, `HIGH`, `MEDIUM`, `INFO`), actionable descriptions, and parameter evidence.
5. **Explorer vs. Preflight separation**:
   - Transaction Explorer modes display factual consensus and chain data without policy scoring.
   - PSBT Preflight evaluates deterministic security policies against transaction facts.

---

## CLI workflows

### 1. Offline raw transaction inspection

```text
Raw Transaction Hex
        ↓
txsignx tx inspect <RAW_TX_HEX> [--network <NET>] [--json]
        ↓
txsignx-core consensus decoder
        ↓
Factual Report: TXID, wTXID, size, vsize, weight, locktime, inputs, outputs, scripts, disassembly
(Addresses rendered only when --network is specified; fees reported as unavailable)
```

1. Run `txsignx tx inspect <RAW_TX_HEX>`.
2. Inspect structural transaction parameters: version, locktime, input count, output count.
3. Review inputs for sequence values and explicit BIP 125 RBF signaling.
4. Review script disassembly (`scriptSig asm` and `scriptPubKey asm`) for opcodes and push bytes.
5. Provide `--network <network>` (e.g. `bitcoin` or `regtest`) to derive standard output addresses.
6. Note that fees and fee rates are reported as **unavailable** because raw transactions omit previous output amounts.

### 2. Node-assisted transaction explorer (txid lookup)

```text
Transaction ID (txid)
        ↓
txsignx tx inspect --txid <TXID> --node-url <URL> --cookie-file <PATH> --network <NET>
        ↓
Bitcoin Core RPC (getblockchaininfo & getrawtransaction)
        ↓
Single-hop prevout resolution (getrawtransaction for parent txids)
        ↓
Complete Explorer Report: confirmation status, block hash, resolved prevouts, input total, fee, fee rate
```

1. Ensure your local Bitcoin Core node is running (for arbitrary historical lookups, `txindex=1` is recommended).
2. Execute:
   ```sh
   txsignx tx inspect \
     --txid <TXID> \
     --node-url http://127.0.0.1:8332 \
     --cookie-file ~/.bitcoin/.cookie \
     --network bitcoin
   ```
3. Inspect chain context: status (`confirmed` with confirmation count and block hash, or `unconfirmed / mempool`).
4. Inspect resolved previous outputs: previous values and scriptPubKeys.
5. Review authoritative fee calculation and virtual size fee rate (`sat/vB`).

### 3. Pre-signing security preflight (PSBT)

```text
Unsigned / Partially Signed PSBT v0
        ↓
txsignx psbt preflight --file <PSBT_FILE> [--wallet-config <CONFIG>] [--node-url ...]
        ↓
txsignx-core (extract facts) + txsignx-wallet (descriptor matching) + txsignx-node (UTXO state)
        ↓
txsignx-policy evaluation (15 active rules)
        ↓
Decision: PASS (0), REVIEW (2), or BLOCK (3) with explicit findings and coverage
        ↓
User reviews evidence → Passes to external signer if approved
```

1. Export unsigned or partially signed PSBT from your wallet coordinator (e.g. Sparrow, Electrum, Core).
2. Run preflight in CLI:
   ```sh
   txsignx psbt preflight --file payment.psbt
   ```
3. Review the policy decision:
   - **PASS**: No evaluated active rule triggered. Confirm that required contexts (wallet/node) were evaluated or deliberately omitted.
   - **REVIEW**: Review specific `HIGH` or `MEDIUM` findings (e.g. `TG011` unusual sighash, `TG010` missing prevouts).
   - **BLOCK**: Discrepancy detected (e.g. `TG002` excessive fee, `TG014` non-zero OP_RETURN, `TG016` node prevout mismatch). Abort signing.
4. If the decision is acceptable, hand off the PSBT to your external signer (hardware wallet, cold storage, or air-gapped signer).

---

## Error handling and recovery

| Scenario / Code | Cause | Resolution |
| --- | --- | --- |
| `invalid_txid` | Supplied `--txid` or payload `txid` is not 64 hex characters | Verify transaction hash string. |
| `invalid_transaction` | Raw hex fails Bitcoin consensus deserialization | Verify transaction hex encoding. |
| `invalid_network` | Network name not recognized (`bitcoin`, `testnet`, `testnet4`, `signet`, `regtest`) | Provide a supported network name. |
| `node_not_configured` | API or CLI requested `--txid` lookup without node connection | Configure Bitcoin Core node parameters. |
| `TransactionNotFound` | Txid is absent from mempool and node has no `txindex=1` | Ensure txid exists or run node with `txindex=1`. |
| `UnsupportedNetwork` | Node chain does not match explicit network | Check whether node is on mainnet, testnet, or regtest. |
| `NodeNotReady` | Node is in Initial Block Download (IBD) or headers desynced | Wait for Bitcoin Core to finish synchronizing. |
| `OutputExceedsInput` | Resolved input values are less than output values | Invalid transaction context; fee cannot be negative. |
| `InvalidEndpoint` | Node URL is not literal `127.0.0.1` or `[::1]` | Use literal loopback IP; hostnames like `localhost` are rejected. |
