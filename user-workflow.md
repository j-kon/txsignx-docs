# User Workflows

TxSignX supports three primary operational workflows: offline raw transaction inspection, node-assisted transaction exploration, and pre-signing policy preflight.

---

## 1. Offline raw transaction explorer

Used to decode and inspect consensus-serialized Bitcoin transaction hex without network access.

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

### Steps
1. Run `txsignx tx inspect <RAW_TX_HEX>`.
2. Inspect structural transaction parameters: version, locktime, input count, output count.
3. Review inputs for sequence values and explicit BIP 125 RBF signaling.
4. Review script disassembly (`scriptSig asm` and `scriptPubKey asm`) for opcodes and push bytes.
5. Provide `--network <network>` (e.g. `bitcoin` or `regtest`) if you wish to derive standard output addresses.
6. Note that fees and fee rates are reported as **unavailable** because raw transactions omit previous output amounts.

---

## 2. Node-assisted transaction explorer (txid lookup)

Used to inspect a confirmed or mempool transaction by its transaction ID (`txid`) using a local Bitcoin Core node.

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

### Steps
1. Ensure your local Bitcoin Core node is running (for historical lookups, `txindex=1` is recommended).
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

---

## 3. Pre-signing security preflight (PSBT)

Used to verify transaction intent, evaluate fee thresholds, match wallet keychains, and uncover discrepancies before signing.

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

### Steps
1. Export unsigned or partially signed PSBT from your wallet coordinator (e.g. Sparrow, Electrum, Core).
2. Run preflight in CLI or paste into the web interface:
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

| Scenario | Cause | Resolution |
| --- | --- | --- |
| `invalid txid format` | Supplied `--txid` is not 64 hex characters | Verify transaction hash string. |
| `TransactionNotFound` | Txid is absent from mempool and node has no `txindex=1` | Ensure txid exists or run node with `txindex=1`. |
| `UnsupportedNetwork` | Node chain does not match explicit `--network` | Check whether node is on mainnet, testnet, or regtest. |
| `NodeNotReady` | Node is in Initial Block Download (IBD) or headers desynced | Wait for Bitcoin Core to finish synchronizing. |
| `OutputExceedsInput` | Resolved input values are less than output values | Invalid transaction context; fee cannot be negative. |
| `InvalidEndpoint` | Node URL is not literal `127.0.0.1` or `[::1]` | Use literal loopback IP; hostnames like `localhost` are rejected. |
