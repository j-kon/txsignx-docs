# Video Demonstration Script

Target length: **3–4 minutes**  
Recording format: Terminal capture (1080p, 60fps), clean prompt, large monospace font, high contrast.  
Audio: Clear voiceover narration.

---

## Scene 1: Introduction and CLI startup [0:00 – 0:30]

**Visual**: Terminal at repo root (`~/Developer/jaykon/txsignx/txsignx`).  
**Command**:
```sh
txsignx
```
**Expected on-screen output**:
```text
████████╗██╗  ██╗███████╗██╗ ██████╗ ███╗   ██╗██╗  ██╗
╚══██╔══╝╚██╗██╔╝██╔════╝██║██╔════╝ ████╗  ██║╚██╗██╔╝
   ██║    ╚███╔╝ ███████╗██║██║  ███╗██╔██╗ ██║ ╚███╔╝
   ██║    ██╔██╗ ╚════██║██║██║   ██║██║╚██╗██║ ██╔██╗
   ██║   ██╔╝ ██╗███████║██║╚██████╔╝██║ ╚████║██╔╝ ██╗
   ╚═╝   ╚═╝  ╚═╝╚══════╝╚═╝ ╚═════╝ ╚═╝  ╚═══╝╚═╝  ╚═╝

Bitcoin transaction security before signing.
Inspect. Verify. Sign with Confidence.

v0.1.0
...
```
**Narrator**:
> *"Welcome to TxSignX — a Bitcoin Transaction Explorer and Pre-Signing Security Analyzer built in Rust. In this demo, we'll tour our consensus transaction explorer, node integration, and deterministic pre-signing policy engine."*

---

## Scene 2: Raw transaction explorer & address derivation [0:30 – 1:15]

**Visual**: Clean terminal screen.  
**Command 1**:
```sh
txsignx tx inspect "$(cat crates/txsignx-core/tests/fixtures/legacy.hex)"
```
**Expected on-screen output**:
- Header: Version 2, Locktime 42, Size: 118 bytes, Virtual size: 118 vB.
- SegWit: no, Explicit RBF: yes.
- Opcode disassembly: `scriptSig asm: PUSHBYTES_1 51`, `scriptPubKey asm: OP_DUP OP_HASH160 ...`.
- Fee: `unavailable without prevout context`.

**Narrator**:
> *"First, offline transaction exploration. TxSignX decodes consensus serialization, calculates exact virtual sizes, identifies explicit BIP 125 RBF signaling, and disassembles bytecode into opcodes.*
> *Because raw transactions omit previous output values, TxSignX refuses to invent fees—it explicitly reports fee unavailable."*

**Command 2**:
```sh
txsignx tx inspect "$(cat crates/txsignx-core/tests/fixtures/legacy.hex)" --network bitcoin
```
**Expected on-screen output**:
- Output 0 Address: `147Us9aEq2PvBC5wobBJw1yEpQEbPKzssA` (P2PKH)
- Output 1 Address: `bc1qxvenxvenxvenxvenxvenxvenxvenxven2ymjt8` (P2WPKH)

**Narrator**:
> *"When an explicit network is supplied, TxSignX derives standard Bitcoin addresses. Network is never guessed."*

---

## Scene 3: Txid inspection & prevout fee resolution [1:15 – 2:00]

**Visual**: Terminal executing automated regtest verification.  
**Command**:
```sh
python3 scripts/verify-explorer-regtest.py
```
**Expected on-screen output**:
```text
Starting isolated Regtest daemon in /tmp/txsignx-explorer-regtest-...
Isolated Regtest daemon running with networkactive=0.
Inspecting Confirmed Transaction A: ...
  Confirmed Transaction A verification: PASS
Inspecting Unconfirmed Transaction B: ...
  Unconfirmed Transaction B verification: PASS
Testing human-readable terminal output...
  Human-readable terminal output: PASS

=== ALL CAPSTONE REGTEST EXPLORER CHECKS PASSED ===
```
**Narrator**:
> *"Next, node-assisted exploration. TxSignX connects to a local Bitcoin Core daemon over loopback RPC.*
> *Here, running against an isolated Regtest instance, TxSignX fetches transactions by txid, reports confirmation status and block hash, and resolves parent outputs.*
> *With inputs resolved, it calculates the exact transaction fee—141 satoshis—and the precise fee rate of 1.00 sat/vB."*

**Fallback**: If running in an environment without `bitcoind`, display the recorded output from `crates/txsignx-cli/tests/regtest_explorer.rs` or explain that txid mode requires Bitcoin Core.

---

## Scene 4: Deterministic pre-signing policy preflight [2:00 – 2:50]

**Visual**: Preflight command executions.  
**Command 1 (PASS)**:
```sh
txsignx psbt preflight --file fixtures/policy/pass.b64
```
**Narrator**:
> *"Now for pre-signing security. For a standard payment, preflight returns PASS. Notice the coverage: fact rules pass, while unconfigured wallet and node rules are explicitly marked 'Not Evaluated'."*

**Command 2 (REVIEW)**:
```sh
txsignx psbt preflight --file fixtures/policy/unusual-sighash.b64
```
**Narrator**:
> *"When a transaction contains an unusual signature hash like SIGHASH_NONE, rule TG011 triggers REVIEW with exit code 2, warning the user before signing."*

**Command 3 (BLOCK)**:
```sh
txsignx psbt preflight --file fixtures/policy/absolute-fee.b64
```
**Narrator**:
> *"When a transaction fee exceeds configured development limits—here 200,000 satoshis against a 100,000-sat limit—rule TG002 triggers BLOCK with exit code 3, preventing accidental fee overpayments."*

---

## Scene 5: Policy registry & conclusion [2:50 – 3:30]

**Visual**: Terminal displaying policy registry.  
**Command**:
```sh
txsignx policy list
```
**Expected on-screen output**:
Structured list of rules `TG001` through `TG017` with color-coded severity tags and descriptions.

**Narrator**:
> *"All 15 policy rules are evaluated deterministically in Rust. TxSignX holds no private keys and never signs. It empowers users to inspect consensus facts and verify security policies before signing.*
> *TxSignX: Inspect. Verify. Sign with Confidence. Thank you!"*
