# Local API overview

The versioned HTTP interface orchestrates the Rust crates. The implementation contract is recorded in the Rust repository at `docs/milestone-6-plan.md`; final verification evidence is separate. These instructions target the Milestone 6 branches until merged.

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
| GET | `/api/v1/health` | Service status, name and version; no credentials or paths |
| GET | `/api/v1/capabilities` | Supported features, configured node availability, rule counts and limits |
| GET | `/api/v1/policies` | Rust `RuleCatalog` with `active_rules` and `deferred_rules` |
| GET | `/api/v1/policies/TG002` | One Rust rule's metadata; unknown codes return 404 |
| POST | `/api/v1/transactions/inspect` | `{"raw_transaction":"<HEX>"}` → `TransactionReport` |
| POST | `/api/v1/psbt/inspect` | `{"psbt":"<BASE64>"}` → `PsbtReport` |
| POST | `/api/v1/psbt/preflight` | Typed context below → `PreflightReport` |

Analysis requests require `Content-Type: application/json`. Unknown fields are rejected. Do not put transaction data or descriptors in query strings. There is no signing, finalization or broadcast route.

Minimal preflight request:

```json
{
  "psbt": "<BASE64>",
  "policy": {
    "max_absolute_fee_sats": 100000,
    "max_fee_ratio_bps": 1000
  }
}
```

`policy` is optional; omitted fields use engine defaults. Optional `wallet` contains required `network`, `external_descriptor` and `internal_descriptor`, plus `derivation_window` (default 1000) and `expected_change_outputs` (default empty). Descriptors must be public. Explicit expected-change indexes describe user intent; they are not automatic change detection.

Optional `node: {"use_configured_node": true}` requests the server-configured node. It requires wallet context and a network matching server configuration. No browser RPC URL, cookie, cookie-file path or arbitrary filesystem path is accepted.

The server alone accepts node startup configuration through `--network`, `--rpc-url` and `--rpc-cookie-file`. RPC must use literal-loopback addressing and cookie authentication. `node_context_available` indicates configuration, not successful live connectivity. A node request without usable configured context returns a typed error, never fabricated offline node facts.

## Reading responses

Successful policy evaluations return HTTP 200, including REVIEW and BLOCK. Decisions and other Rust enums use snake_case JSON values, for example `pass`, `review` and `block`. The frontend may capitalize labels but must not replace the decision. Preflight includes inspection and policy reports, with optional contextual reports. Read the policy's `findings`, `risk_level`, `rule_evaluations` and `scope_note` together.

Absent fee/input values remain unavailable, not zero. Raw inspection cannot infer network or input values. An available PSBT fee uses supplied previous-output data. Report consumers must preserve integer precision rather than silently round large JSON integers.

## Limits and errors

| Boundary | M6 local-service contract |
| --- | --- |
| Input text | 1 MiB |
| Total request body | 2 MiB |
| Serialized response | 8 MiB |
| Admitted requests | Four concurrently |
| CPU/RPC workers | Two, with immediate overload rejection |
| HTTP deadline | 30 seconds |

Engine limits also apply. A timed-out blocking computation retains its worker permit until it ends; timeout does not claim to cancel running Rust/RPC work. These controls do not substitute for production deployment infrastructure.

Errors have a sanitized envelope:

```json
{"error":{"code":"invalid_input","message":"Invalid input or context."}}
```

| HTTP status | Category |
| --- | --- |
| 400 | `invalid_json`: invalid request or malformed JSON |
| 403 | `forbidden`: rejected Origin or Host |
| 404 / 405 | `not_found` / `method_not_allowed` |
| 413 / 415 | `size_limit` (request or response) / `unsupported_media_type` |
| 422 | `invalid_input`: invalid transaction, PSBT or semantic context |
| 500 | `internal`: internal failure |
| 503 | `node_unavailable` or `busy` |
| 504 | `timeout` |

Consumers should handle the structured error code and avoid relying on parser wording. Rejected browser origins or Host headers can also fail before analysis. Raw RPC errors, request data, private descriptors and filesystem secrets must not appear in errors. Responses use no-store caching and browser security headers. Explicit CORS origins never imply authentication.

See [security model](security-model.md) for the meaning of node trust and PASS.
