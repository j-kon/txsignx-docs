# Capstone demonstration

A reproducible demonstration shows real Rust output from public synthetic inputs. It does not claim production readiness, independently verified signatures or live node context when none was supplied.

## Prepare

Use the Milestone 6 Rust and web branches. Follow [local startup](api.md), then confirm `/api/v1/health` and `/api/v1/capabilities` respond. Use only the Rust repository's public fixtures:

| Case | File in Rust repository | Expected default offline preflight |
| --- | --- | --- |
| PASS | `fixtures/policy/pass.b64` | No findings; 1,000-sat supplied-context fee |
| REVIEW | `fixtures/policy/unusual-sighash.b64` | TG011: explicit sighash NONE (numeric 2) |
| BLOCK | `fixtures/policy/absolute-fee.b64` | TG002: 200,000-sat fee exceeds 100,000-sat limit |

These expected outcomes come from the deterministic fixture definitions. Run them against the local API before presenting; this page is a walkthrough, not a record that a particular integration run passed. The fixtures use synthetic outpoints, not spendable funds. Do not enable node analysis for these offline cases.

A terminal smoke check from the Rust repository, with the API already running:

```sh
python3 - <<'PYTHON'
import json
import urllib.request
from pathlib import Path

for name, expected in [('pass', 'pass'), ('unusual-sighash', 'review'), ('absolute-fee', 'block')]:
    payload = json.dumps({'psbt': Path(f'fixtures/policy/{name}.b64').read_text().strip()}).encode()
    request = urllib.request.Request(
        'http://127.0.0.1:8080/api/v1/psbt/preflight',
        data=payload,
        headers={'Content-Type': 'application/json'},
    )
    with urllib.request.urlopen(request, timeout=35) as response:
        report = json.load(response)
    actual = report['policy']['decision']
    print(name, actual)
    assert actual == expected, (name, actual)
PYTHON
```

This command sends public fixtures to loopback only and performs no signing, broadcasting or node startup.

## Live walkthrough

1. **Home:** read “Bitcoin transaction security before signing.” Explain that TxSignX is a security layer for wallets.
2. **Inspector:** load the PASS sample, or paste `pass.b64`. Inspect real input/output facts, signing state and fee status. Explain that the fee is based on supplied UTXO context.
3. **Preflight:** run without wallet/node context. Show PASS and its scope wording. Expand Not Evaluated and point out the absent wallet/node checks.
4. **REVIEW:** load `unusual-sighash.b64`, run preflight and inspect TG011, its High severity and the explicit sighash. Explain that unusual commitments can be intentional.
5. **BLOCK:** load `absolute-fee.b64`, run preflight and show TG002 with the 200,000-sat fee and 100,000-sat configured threshold.
6. **Policies:** show the registry fetched from Rust: 15 active rules and two deferred rules. Explain that the React app does not evaluate those rules.
7. **Report:** explicitly copy or download JSON and show its decision, findings and separate coverage entries. Do not expose personal data or credentials on screen.
8. **Failure handling:** submit a clearly invalid PSBT and show the sanitized error. If demonstrating an offline API, stop only your task-owned API process, retry, then restart it.
9. **Architecture and limits:** show [the diagram](architecture.md), point to Rust policy authority and the external signer, then explain no signing, finalization or API broadcast.

## Optional node-aware segment

Keep this separate from the offline demonstration. Use only the Rust verification harness or an explicitly isolated task-owned Regtest node. Every `bitcoind` invocation must specify `-regtest`, an exact temporary `-datadir`, nondefault ports and `-connect=0 -dnsseed=0 -listen=0 -networkactive=0 -discover=0`. Never run plain `bitcoind`, start a system Bitcoin service, use `~/.bitcoin`, or sync a public chain for this demo.

Configure the API's literal-loopback RPC URL and cookie-file path server-side. Supply matching public wallet context in the request. Demonstrate node rules only with real Regtest-created fixture context; never relabel offline synthetic results as node verification. Stop only the task-owned node, record the temporary datadir size, remove only that exact task-created directory and verify its deletion. The final Rust verification record should capture commands and actual results.

## Presentation outline

| Slide | Message / visual |
| --- | --- |
| 1. Problem | Signing commits value; users need understandable pre-sign evidence |
| 2. Signing gap | Transaction bytes alone do not prove intent, ownership, fee context or chain state |
| 3. Architecture | Web → API → Rust core/wallet/node/policy → report → external signer |
| 4. Milestones | M1 raw facts; M2 PSBT inspection; M3 policy; M4 wallet context; M5 node context; M6 API and web product |
| 5. Policy rules | 15 active, two deferred; findings, severity and coverage are Rust-owned |
| 6. Live demo | Real PASS, REVIEW and BLOCK from public synthetic fixtures |
| 7. Security model | Untrusted input, bounded descriptor matching, trusted node observations, limited PASS |
| 8. Explicit exclusions | No keys, signing, finalization or API broadcast |
| 9. Evidence and next scope | Show only recorded checks; deferred policies and separately reviewed integrations |

## Video script — approximately four minutes

**0:00–0:30 — Problem and positioning**

“Bitcoin transaction security before signing. TxSignX is a deterministic open-source pre-sign security engine for transactions and PSBTs. It is not another wallet: it is a security layer for wallets. The goal is to make transaction facts and policy concerns visible before a user moves to an external signer.”

**0:30–1:05 — Architecture**

“The React interface sends HTTP JSON to a local Axum API. Rust inspects the transaction, adds optional wallet and node context, and evaluates the policy rules. Rust supplies the decision, findings and coverage. The frontend presents those results; it does not implement its own security scoring. Signing remains outside this flow.”

**1:05–1:45 — PASS and coverage**

“Here is a public synthetic PSBT. We can inspect its inputs, outputs, signing state and fee status. Running preflight produces PASS. This means no evaluated active rule requires review or blocking. It does not mean universally safe. We have not supplied a wallet or node here, so those checks are listed as not evaluated. The fee comes from supplied previous-output information, not independently authenticated chain data.”

**1:45–2:20 — REVIEW**

“This next sample explicitly requests an unusual sighash. Rust returns REVIEW and TG011, with the finding's severity and explanation. This can be intentional, but it changes the commitment assumptions that the user needs to understand. The finding is evidence for review, not a claim that every unusual transaction is malicious.”

**2:20–2:55 — BLOCK**

“The third sample has a 200,000-sat fee against a configured 100,000-sat maximum. TG002 produces a critical finding and BLOCK. The threshold and evidence are visible. The browser displays the decision returned by Rust, without recomputing it.”

**2:55–3:25 — Registry and export**

“The policy page gets its registry from the API. There are 15 active rules and two deferred codes. Deferred does not mean passed. We can export the actual JSON report, including findings and coverage. Export is deliberate, because transaction and wallet context can be sensitive.”

**3:25–4:05 — Boundaries and conclusion**

“TxSignX does not handle seeds or private keys, sign, finalize, or broadcast through the API. Optional node checks use server-configured local Core context and remain dependent on that node's state. This is development software, not an independently audited production wallet. Milestone 6 connects the existing Rust engine to a usable local inspection workflow: inspect facts, review evidence and coverage, then make any signing decision outside TxSignX.”

## Record evidence honestly

Record the actual commit IDs, commands, outcomes and environment used for the presentation. Link to the Rust milestone verification document when complete. Do not substitute fixture expectations, screenshots, this script or prior milestone counts for new integration evidence. A failed scenario should be fixed or disclosed before recording.
