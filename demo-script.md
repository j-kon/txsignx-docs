# Live Demonstration Script

Target length: **5–7 minutes**
Presenter: Technical speaker presenting TxSignX capstone to instructors and peers.
Format: Live Web Interface presentation supported by the local TxSignX HTTP API and terminal CLI.

---

### [0:00 – 0:45] Problem and project positioning

> *"Hello everyone. Today I am presenting **TxSignX — A Bitcoin Transaction Explorer and Pre-Signing Security Analyzer**.*
>
> *In Bitcoin, cryptographic signing is an irreversible commitment of funds. But modern wallets often force users to sign 'blindly'—relying on a high-level summary that hides script mechanics, fee shares, or subtle signature flag risks.*
>
> *A standard transaction explorer tells you what an existing transaction contains on-chain. TxSignX brings that deep exploration capability directly into the pre-signing stage. We combine consensus transaction decoding with deterministic security policies so users can inspect, verify, and sign with confidence.*
>
> *Today I'll demonstrate our completed capstone across both our web application and Rust engine."*

---

### [0:45 – 1:30] Architecture and security boundaries

> *(Show architecture diagram from [architecture.md](architecture.md))*
>
> *"Here is how TxSignX is architected:*
>
> *At the foundation is `txsignx-core`, written in Rust, which decodes raw consensus transactions and PSBTs using `rust-bitcoin`.*
>
> *The application divides into two distinct responsibilities:*
> 1. *The **Transaction Explorer**: purely factual consensus decoding, opcode disassembly, address derivation, and node-assisted prevout resolution.*
> 2. *The **Preflight Security Analyzer**: evaluates derived transaction facts against 15 deterministic active policy rules in `txsignx-policy`.*
>
> *Our web interface is strictly presentation—it communicates with `txsignx-api` over HTTP. Crucially, Bitcoin Core credentials never touch the browser; they remain isolated on the server. And TxSignX holds zero private keys, performs no signing, and uses zero AI guesswork."*

---

### [1:30 – 2:45] Demo 1: Web Raw Transaction Explorer & Address Derivation

> *(Browser: Open TxSignX Web Inspector at `http://127.0.0.1:5173` and select the **Raw Transaction** tab)*
>
> *"Let's start with our Web Transaction Explorer in **Raw Transaction** mode.*
>
> *(Paste raw hex from `crates/txsignx-core/tests/fixtures/legacy.hex`)*
>
> *First, we leave the network selector set to **No network** and click **Inspect Transaction**.*
>
> *Notice what the explorer renders:*
> - *Consensus parameters: Version 2, locktime 42, size 118 bytes, virtual size 118 vB.*
> - *Identifies explicit BIP 125 RBF signaling (`nSequence < 0xFFFFFFFE`).*
> - *Disassembles both `scriptSig` and `scriptPubKey` into human-readable opcodes and push bytes.*
>
> *Now, look closely at the fee card: it says **'unavailable without prevout context'**.*
> *Why? Because raw consensus transactions do not encode previous output amounts. TxSignX operates on honest consensus facts—we refuse to guess or invent fees.*
>
> *(Change Network selector to **Bitcoin (Mainnet)** and re-inspect)*
>
> *"When we explicitly declare the network, TxSignX derives standard Bitcoin addresses—here a legacy P2PKH and a native SegWit P2WPKH address. Because raw transactions contain no network byte, network is never assumed silently."*

---

### [2:45 – 3:50] Demo 2: Web Transaction ID Explorer & Node Context

> *(Browser: Switch to the **Transaction ID** tab)*
>
> *"Next, let's explore transaction inspection by **Transaction ID**.*
>
> *In txid mode, the browser submits only the 64-character hash to `txsignx-api`. The server queries its local Bitcoin Core node over loopback RPC. Notice that the browser never prompts for RPC passwords or cookie files.*
>
> *(If node is unconfigured: show the `node_not_configured` informational state explaining that Bitcoin Core is required for TXID lookup).*
>
> *(When connected to local node, e.g. Regtest daemon: paste confirmed txid)*
>
> *"Here, connected to our local Bitcoin Core node, TxSignX fetches the transaction and performs a bounded, single-hop lookup of the parent transactions.*
>
> *Look at the rich factual report:*
> - *Confirmation status: confirmed with block hash and confirmation count.*
> - *Previous outputs: resolved parent amounts and scripts.*
> - *With inputs resolved, TxSignX computes the authoritative fee—141 satoshis—and the precise fee rate of 1.00 sat/vB.*
>
> *If the transaction were unconfirmed in the mempool, it explicitly reflects mempool status without fabricating confirmation data."*

---

### [3:50 – 5:10] Demo 3: PSBT Preflight (PASS / REVIEW / BLOCK) & Separation

> *(Browser: Switch to the **PSBT** tab)*
>
> *"Now let's examine pre-signing security analysis using BIP 174 PSBT v0.*
>
> *(Paste `pass.b64` from `fixtures/policy/pass.b64` and click Analyze)*
>
> *"First, a clean transaction: `pass.b64`. The policy engine returns **PASS**. Look at the coverage breakdown: fact-based rules evaluated clean, while unconfigured wallet and node rules are transparently marked **'Not Evaluated'**. In TxSignX, PASS means no active evaluated rule requires review or blocking; it never pretends unrun checks passed.*
>
> *(Paste `unusual-sighash.b64` from `fixtures/policy/unusual-sighash.b64` and click Analyze)*
>
> *"Second, an unusual transaction: `unusual-sighash.b64`. The engine returns **REVIEW** and flags rule `TG011`. The input specifies `SIGHASH_NONE`, leaving outputs uncommitted. Because this can be intentional in multiparty protocols, TxSignX flags it for human review rather than blocking.*
>
> *(Paste `absolute-fee.b64` from `fixtures/policy/absolute-fee.b64` and click Analyze)*
>
> *"Third, an excessive fee: `absolute-fee.b64`. The fee is 200,000 satoshis, exceeding the 100,000-sat development limit. The engine immediately returns **BLOCK** under rule `TG002`, stopping a fat-finger loss before signing.*
>
> *Notice the conceptual separation: Transaction Explorer modes display objective consensus facts; PSBT Preflight applies deterministic policy rules. The explorer never shows policy badges, and preflight never replaces consensus facts."*

---

### [5:10 – 6:00] Summary, Hardened Boundaries, and Verification

> *"To summarize:*
> - *TxSignX satisfies every requirement of the Transaction Explorer capstone across CLI, HTTP API, and Web.*
> - *Our security boundaries are strict: zero private key handling, literal loopback RPC, server-isolated node credentials, and zero AI heuristics.*
> - *The implementation is verified by **345 Rust workspace tests** across 6 crates, **48 Web tests**, zero Clippy warnings, and zero audit vulnerabilities.*
>
> *Thank you, and I look forward to your questions!"*
