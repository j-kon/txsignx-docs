# Live Demonstration Script

Target length: **5–7 minutes**  
Presenter: Technical speaker presenting TxSignX capstone to instructors and peers.

---

### [0:00 – 0:40] Problem and project positioning

> *"Hello everyone. Today I'm presenting **TxSignX — A Bitcoin Transaction Explorer and Pre-Signing Security Analyzer**.*
>
> *In Bitcoin, cryptographic signing is an irreversible commitment of funds. But modern wallets often force users to sign 'blindly'—relying on a high-level summary that doesn't reveal script details, exact fee mechanics, or subtle signature flag risks.*
>
> *A standard transaction explorer tells you what an existing transaction contains on-chain. TxSignX takes that exploration capability and moves it into the pre-signing stage. We combine consensus transaction decoding with deterministic security policies so users can inspect, verify, and sign with confidence."*

---

### [0:40 – 1:20] Architecture overview

> *(Show architecture diagram from [architecture.md](architecture.md))*
>
> *"Here is how TxSignX is built. At the core is `txsignx-core`, written in Rust, which decodes raw consensus transactions and PSBTs using `rust-bitcoin`.*
>
> *From there, the architecture splits into two complementary paths:*
> 1. *The **Transaction Explorer path**: operates independently to disassemble scripts, derive addresses, and resolve on-chain confirmation and fee data via a local Bitcoin Core node.*
> 2. *The **Preflight Security path**: feeds derived transaction facts, optional public descriptor wallet context, and node UTXO observations into `txsignx-policy`.*
>
> *Crucially, TxSignX is not a wallet and never touches private keys. All policy decisions are 100% deterministic Rust rules—zero AI guesswork. The final signing decision remains strictly external."*

---

### [1:20 – 2:20] Demo 1 & 2: Offline raw transaction explorer

> *(Terminal: Run `txsignx tx inspect "$(cat crates/txsignx-core/tests/fixtures/legacy.hex)"`)*
>
> *"Let's look at the Transaction Explorer in action. First, raw transaction inspection completely offline.*
>
> *Here, we inspect a raw consensus hex string. TxSignX extracts the TXID, virtual size, locktime, and explicit BIP 125 RBF signaling.*
>
> *Notice two key engineering details:*
> *First, script disassembly: rather than showing raw bytecode, TxSignX safely disassembles both `scriptSig` and `scriptPubKey` into opcodes and bounded push-bytes.*
> *Second, look at the fee: it says **'Fee: unavailable without prevout context'**. Why? Because raw Bitcoin transactions do not encode previous output amounts. We refuse to invent or guess fees.*
>
> *(Terminal: Run with `--network bitcoin`)*
>
> *"Now, when we supply `--network bitcoin`, TxSignX explicitly derives standard addresses—in this case, P2PKH and P2WPKH. Raw transactions do not encode their network, so address derivation is strictly explicit."*

---

### [2:20 – 3:15] Demo 3: Node-assisted txid inspection via Bitcoin Core

> *(Terminal: Run `python3 scripts/verify-explorer-regtest.py` or demonstrate live regtest `--txid`)*
>
> *"What if we have a transaction ID on a node? TxSignX connects to a local Bitcoin Core instance via hardened loopback RPC.*
>
> *Here in our isolated Regtest environment, we inspect by txid. Notice what happens:*
> *Bitcoin Core reports confirmation status—here confirmed with block hash and height.*
> *More importantly, TxSignX performs a bounded, single-hop lookup of the previous parent transaction to resolve input amounts.*
> *Now that input values are known, we calculate the exact fee—141 satoshis—and the precise fee rate—1.00 sat/vB.*
> *If the transaction were in the mempool, it accurately reports unconfirmed status."*

---

### [3:15 – 4:45] Demo 4: Pre-signing security preflight (PASS, REVIEW, BLOCK)

> *(Terminal or Web UI: Show PSBT preflights)*
>
> *"Now let's move to pre-signing security analysis using PSBTs.*
>
> *First, a clean transaction: `pass.b64`. Preflight outputs **PASS** with exit code 0. Notice the coverage breakdown: fact-based rules evaluated clean, while wallet and node rules are explicitly marked **'Not Evaluated'** because no wallet or node was bound. In TxSignX, PASS never pretends unrun checks passed.*
>
> *(Terminal: Run `txsignx psbt preflight --file fixtures/policy/unusual-sighash.b64`)*
>
> *"Second, an unusual transaction: `unusual-sighash.b64`. The policy engine returns **REVIEW** with exit code 2 and flags rule `TG011`. The input explicitly requests `SIGHASH_NONE`, leaving outputs uncommitted. This isn't an error—it could be a legitimate multiparty protocol—so TxSignX flags it for review rather than blocking.*
>
> *(Terminal: Run `txsignx psbt preflight --file fixtures/policy/absolute-fee.b64`)*
>
> *"Third, an overpayment: `absolute-fee.b64`. The fee is 200,000 satoshis, exceeding the 100,000-sat threshold. The engine immediately returns **BLOCK** with exit code 3, catching the fat-finger fee before any signature is applied."*

---

### [4:45 – 5:30] Policy registry and security boundaries

> *(Terminal: Run `txsignx policy list`)*
>
> *"Here is the live policy registry: 15 active deterministic rules categorized across network, fee, wallet, script, and node categories, plus two deferred codes.*
>
> *Our security boundaries are strict:*
> * Loopback-only RPC (`127.0.0.1` and `[::1]`) with cookie authentication and 1 MiB response caps.
> * Zero private key handling or seed generation.
> * Pure JSON output (`--json`) with zero ANSI escapes for automated pipeline integration."*

---

### [5:30 – 6:15] What was learned and future work

> *"Building TxSignX deepened my understanding of the exact boundary between Bitcoin consensus decoding and transaction context. Resolving prevouts, managing strict loopback transports, and maintaining deterministic policy invariant guarantees in Rust proved to be an invaluable systems engineering experience.*
>
> *Future work will include PSBT v2 support and hardware wallet companion workflows.*
>
> *Thank you, and I look forward to your questions!"*
