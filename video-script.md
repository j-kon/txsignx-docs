# Video Demonstration Script

Target length: **3–4 minutes**
Recording format: Web interface capture (1080p, 60fps) with terminal split/cutaways, clean browser window, large font, high contrast.
Audio: Clear voiceover narration.

---

## Scene 1: Introduction and overview [0:00 – 0:35]

**Visual**: Clean browser showing TxSignX Web application (`http://127.0.0.1:5173`) with tagline: *Inspect. Verify. Sign with Confidence.*
**Cutaway / Split**: Quick terminal view showing `txsignx` CLI banner.

**Narrator**:
> *"Welcome to TxSignX — a Bitcoin Transaction Explorer and Pre-Signing Security Analyzer built in Rust. In Bitcoin, signing is an irreversible commitment. Wallets often force users to sign blindly, hiding critical script details and fee shares.*
> *TxSignX brings deep transaction exploration into the pre-signing stage across the web and terminal. In this demo, we'll tour our raw transaction explorer, node integration, and deterministic pre-signing policy engine."*

---

## Scene 2: Raw transaction explorer & address derivation [0:35 – 1:25]

**Visual**: TxSignX Web Inspector on the **Raw Transaction** tab.
**Action 1**: Paste raw consensus hex from `legacy.hex` with network set to **No network**, click **Inspect Transaction**.
**Expected on-screen UI**:
- Headers: Version 2, Locktime 42, Size 118 bytes, Virtual size 118 vB.
- Explicit RBF signaling: Detected (`nSequence: 0xfffffffd`).
- Opcode disassembly: `scriptSig` and `scriptPubKey` disassembled into readable opcodes.
- Fee card: Explicitly displays **unavailable without prevout context**.

**Narrator**:
> *"First, raw transaction inspection completely offline. TxSignX decodes consensus serialization, computes exact virtual sizes, identifies explicit BIP 125 RBF signaling, and disassembles bytecode into opcodes.*
> *Notice the fee: raw Bitcoin transactions omit previous output values, so TxSignX refuses to invent or guess fees—it reports fee unavailable."*

**Action 2**: Switch network dropdown to **Bitcoin (Mainnet)** and re-inspect.
**Expected on-screen UI**:
- Standard addresses rendered: Legacy P2PKH (`147Us9a...`) and SegWit P2WPKH (`bc1qxve...`).

**Narrator**:
> *"When an explicit network is selected, standard addresses are derived. Because consensus transactions carry no network identifier, network is never assumed silently."*

---

## Scene 3: Txid inspection & node-assisted prevout resolution [1:25 – 2:15]

**Visual**: Switch to **Transaction ID** tab in Web Inspector.
**Action**: Paste confirmed transaction ID from Regtest demo (or display node-assisted lookup).
**Expected on-screen UI**:
- Confirmation status: `confirmed` with block hash and confirmation count.
- Resolved previous outputs: Parent transaction amounts and locking scripts.
- Authoritative fees: Exact fee (141 satoshis) and virtual size fee rate (1.00 sat/vB).
- Security highlight callout: Node credentials remain isolated on the server; the browser never prompts for or stores RPC credentials.

**Narrator**:
> *"Next, node-assisted exploration. In txid mode, the browser sends only the 64-character hash to our HTTP API. Bitcoin Core credentials remain strictly isolated on the server.*
> *TxSignX connects to the node over loopback RPC, fetches confirmation status, and performs a single-hop lookup of parent transactions.*
> *With inputs resolved, it calculates the exact fee—141 satoshis—and the precise fee rate of 1.00 sat/vB. If the transaction were in the mempool, it accurately reports unconfirmed mempool status."*

---

## Scene 4: Pre-signing security preflight (PASS, REVIEW, BLOCK) [2:15 – 3:15]

**Visual**: Switch to **PSBT** tab in Web Inspector.
**Action 1**: Paste `pass.b64` and click **Analyze PSBT**.
**Expected on-screen UI**: Green **PASS** verdict banner with transparent coverage metrics.

**Narrator**:
> *"Now for pre-signing security using BIP 174 PSBT v0. For a clean payment, preflight returns PASS. In TxSignX, PASS means no active evaluated rule requires review or blocking; unconfigured wallet and node checks are transparently marked 'Not Evaluated'."*

**Action 2**: Paste `unusual-sighash.b64` and click **Analyze PSBT**.
**Expected on-screen UI**: Amber **REVIEW** banner with rule `TG011` flagged (`SIGHASH_NONE`).

**Narrator**:
> *"When a transaction contains an unusual sighash like SIGHASH_NONE, rule TG011 triggers REVIEW, warning the user that outputs are uncommitted before they sign."*

**Action 3**: Paste `absolute-fee.b64` and click **Analyze PSBT**.
**Expected on-screen UI**: Red **BLOCK** banner with rule `TG002` flagged (excessive fee over threshold).

**Narrator**:
> *"When a transaction fee breaches configured limits—here 200,000 satoshis—rule TG002 triggers BLOCK, preventing catastrophic fee overpayments before any signature is applied.*
> *Notice that Explorer facts and Preflight verdicts remain strictly separate: the explorer never invents policies, and preflight never replaces consensus facts."*

---

## Scene 5: Verification, security boundaries & conclusion [3:15 – 3:45]

**Visual**: Split screen: terminal showing verified test suites alongside GitHub repository links.
- Rust workspace: 345 passed tests across 6 crates.
- Web: 48 passed tests, zero lint warnings, zero audit vulnerabilities.

**Narrator**:
> *"TxSignX holds zero private keys, performs no signing, and uses 100% deterministic Rust rules with zero AI guesswork.*
> *Verified across 345 Rust tests and 48 web tests, TxSignX delivers an auditable explorer and security analyzer for the Bitcoin ecosystem.*
> *TxSignX: Inspect. Verify. Sign with Confidence. Thank you!"*
