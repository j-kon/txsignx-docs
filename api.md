# Local API overview

The versioned HTTP interface (`txsignx-api`) exposes consensus transaction inspection and deterministic security policy evaluation over local HTTP. The implementation contract is recorded in the Rust repository; empirical verification evidence is documented in [verification.md](verification.md).

## Start locally

From the Rust repository:

```sh
cargo run -p txsignx-api -- --bind 127.0.0.1:8080
```

From the web repository, in a second terminal:

```sh
npm ci
VITE_TXSIGNX_API_URL=http://127.0.0.1:8080 npm run dev
```

Open the Vite URL. The default allowed origins are `http://localhost:5173` and `http://127.0.0.1:5173`; a different port or origin requires explicit API configuration. Consult the binary's `--help` for startup options. Nonloopback binding requires `--allow-external` and emits a warning; no authentication service is provided.

## Routes

| Method | Path | Body / response |
| --- | --- | --- |
| GET | `/api/v1/health` | Service status, name, and version (`{"service":"txsignx-api","status":"ok","version":"0.1.0"}`) |
| GET | `/api/v1/capabilities` | Supported features, node availability, rule counts, and request limits |
| GET | `/api/v1/policies` | Rust `RuleCatalog` with `active_rules` (15) and `deferred_rules` (2) |
| GET | `/api/v1/policies/{code}` | One rule's metadata (e.g. `/api/v1/policies/TG002`); unknown codes return 404 |
| POST | `/api/v1/transactions/inspect` | Raw hex or node-backed txid → `TransactionReport` |
| POST | `/api/v1/psbt/inspect` | `{"psbt":"<BASE64>"}` → `PsbtReport` |
| POST | `/api/v1/psbt/preflight` | Typed PSBT + context → `PreflightReport` |

Analysis requests require `Content-Type: application/json`. Unknown JSON fields are rejected. Do not put transaction data or descriptors in query strings. There is no signing, finalization, or broadcast route.

## Transaction Explorer endpoint

`POST /api/v1/transactions/inspect` supports three invocation modes:

### 1. Raw transaction inspection (offline / no network)

```json
{
  "raw_transaction": "<RAW_TRANSACTION_HEX>"
}
```
Returns structural facts: txid, wtxid, version, locktime, inputs, outputs, size, weight, virtual size, explicit RBF signaling, and script disassembly. Raw transactions do not encode network metadata, so output addresses are omitted (`null`), and fee/chain context remain `null` without fabrication.

### 2. Raw transaction with address rendering

```json
{
  "raw_transaction": "<RAW_TRANSACTION_HEX>",
  "network": "bitcoin"
}
```
Supported network identifiers: `bitcoin`, `testnet`, `testnet4`, `signet`, `regtest`. Derives standard addresses for recognized output scripts (`P2PKH`, `P2SH`, `P2WPKH`, `P2WSH`, `P2TR`). Mainnet is never silently assumed. Fees and confirmation status remain `null`.

### 3. Node-backed transaction ID inspection

```json
{
  "txid": "<64_HEX_TRANSACTION_ID>"
}
```
Queries the server-configured local Bitcoin Core node over loopback RPC. Resolves single-hop parent prevouts to compute exact input totals, fees, and fee rates (`sat/vB`). Reports confirmation status (`confirmed` with block hash and confirmation count, or `mempool` without fabricated hashes).

**Rules and boundaries:**
- Mutually exclusive parameters (`raw_transaction` + `txid`, or `txid` + `network`, or neither) fail closed with HTTP 400 `invalid_context`.
- TXID mode requires a server-configured Bitcoin Core node; if no node was configured at API startup, it returns HTTP 400 `node_not_configured`.
- The browser never supplies Bitcoin Core credentials. RPC URL, cookie file path, username, and password are strictly server-side configurations.

## PSBT Preflight endpoint

```json
{
  "psbt": "<BASE64>",
  "policy": {
    "max_absolute_fee_sats": 100000,
    "max_fee_ratio_bps": 1000
  }
}
```

- `policy`: Optional; omitted fields use engine defaults (100,000 sats fee cap, 10% fee ratio limit).
- `wallet`: Optional public descriptor context containing `network`, `external_descriptor`, `internal_descriptor`, `derivation_window` (default 1000), and `expected_change_outputs`. Descriptors must be public; private keys and seed phrases are rejected.
- `node`: Optional `{"use_configured_node": true}` to request verification against the server-configured node.

## Reading responses

Successful policy evaluations return HTTP 200, including `review` and `block`. Decisions and Rust enums use snake_case JSON values (`pass`, `review`, `block`). Preflight reports include inspection details, policy findings, risk level, explicit rule coverage (`evaluated`, `partially_evaluated`, `not_evaluated`), and a scope note.

Transaction Explorer reports are strictly factual and do not contain PASS, REVIEW, or BLOCK verdicts. Integer precision is preserved for all satoshi amounts; report consumers must avoid rounding large integers.

## Service limits and errors

| Boundary | Limit |
| --- | --- |
| Input text | 1 MiB (1,048,576 bytes) |
| Total request body | 2 MiB (2,097,152 bytes) |
| Serialized response | 8 MiB (8,388,608 bytes) |
| Concurrent requests | 4 admitted requests |
| Worker concurrency | 2 blocking workers |
| Request timeout | 30.0 seconds |

Errors return a sanitized JSON envelope:

```json
{"error":{"code":"<ERROR_CODE>","message":"<DESCRIPTION>"}}
```

| HTTP status | Error code | Description |
| --- | --- | --- |
| 400 | `invalid_json` | Malformed or unparseable JSON payload |
| 400 | `invalid_network` | Unrecognized or unsupported Bitcoin network identifier |
| 400 | `invalid_txid` | Malformed transaction ID (not 64 lowercase hex characters) |
| 400 | `node_not_configured` | TXID lookup attempted without server-configured Bitcoin Core node |
| 400 | `invalid_context` | Conflicting or mutually exclusive parameters |
| 403 | `forbidden` | Rejected browser Origin or Host header (DNS rebinding protection) |
| 404 | `not_found` | Resource or policy rule code not found |
| 405 | `method_not_allowed` | HTTP method not supported on this route |
| 413 | `size_limit` | Request or response exceeds byte limit |
| 415 | `unsupported_media_type` | Request header `Content-Type` is not `application/json` |
| 422 | `invalid_transaction` | Raw transaction hex failed consensus parsing |
| 422 | `invalid_input` | Malformed PSBT or semantic preflight context error |
| 500 | `internal` | Internal service error |
| 503 | `node_unavailable` | Configured Bitcoin Core node is unresponsive or returned an RPC error |
| 503 | `busy` | Worker pool or request admission limit saturated |
| 504 | `timeout` | Request exceeded 30-second deadline |

**Consumer guidance:**
Frontend consumers should branch on the structured `error.code` rather than parser message text or assuming that one HTTP status code implies a single condition.
