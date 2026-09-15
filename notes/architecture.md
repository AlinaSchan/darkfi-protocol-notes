# architecture

what the system is made of, where the parts live in the repository, and how a transaction moves
through it. paths point into the darkfi repository at commit `a5514cb2` (2026-09-10).

## the parts

darkfi is one rust workspace. the library in `src/` holds the p2p network, the blockchain store, the
validator, the wasm runtime, the zk virtual machine, the zkas compiler and the sdk that contracts
compile against. the daemons and tools in `bin/` link that library. (`Cargo.toml`,
`doc/src/start-here.md`)

the pieces that make up the chain itself:

- `darkfid` is the full node. it validates transactions, keeps the chain, talks to the p2p network
  and answers json-rpc. it holds no keys and no wallet. (`bin/darkfid`, `doc/src/arch/tooling.md`)
- `drk` is the command line wallet. it keeps keys and coins, subscribes to blocks from a `darkfid`,
  scans them for notes it can decrypt, builds transactions and their proofs locally, and broadcasts
  them through the node. (`bin/drk`, `doc/src/testnet/node.md`)
- mining runs outside the node. `xmrig` connects to `darkfid` over a stratum rpc and asks for block
  headers to mine. a second guide sets up merge mining with monero through `p2pool`.
  (`doc/src/testnet/node.md`, `doc/src/testnet/merge-mining.md`)
- `lilith` is the seed node that other nodes contact to fill their peer lists. (`bin/lilith`,
  `doc/src/start-here.md`)

the older overview page says every wallet is a full node that executes contracts locally; the
tooling page and the testnet guide describe the split above, and the contributor guide itself marks
the overview as out of date. (`doc/src/arch/overview.md`, `doc/src/dev/contrib/contrib.md`)

around the chain sit applications built on the same p2p library, not on the blockchain:

- `darkirc`, a chat daemon that syncs hourly message dags through the event graph and exposes a
  local irc interface (`bin/darkirc`, `src/event_graph`)
- `tau`, a task manager the project uses instead of a hosted issue tracker (`bin/tau`)
- `fud`, a "file-sharing utility daemon, using dht for records discovery"; the contributor guide
  says the dht is broken at the moment (`bin/fud/README.md`, `doc/src/dev/contrib/contrib.md`)
- `app`, the graphical application, which the website offers for android, linux, windows and
  macos. its directory arrived as `bin/darkwallet` in may 2024 and was renamed in january 2025
  (`bin/app`, git history)
- `dnet` and `deg`, viewers for p2p traffic and event graph data, used for debugging (`bin/dnet`,
  `bin/deg`)
- `explorer`, a rust service with a python web frontend (`bin/explorer`)
- `zkas`, `zkrunner` and `vanityaddr`: the compiler, the proof debugger, an address generator

every source file carries a copyright header naming the dyne.org foundation, 2020 to 2026, under
the agpl. (`src/net/metering.rs` and every other `.rs` file)

## contracts

every operation on the chain is a contract call, payments included. three native contracts ship in
`src/contract/`: `money` (transfers, token issuance under a mint authority, token freezing, atomic
swaps, fees), `dao` (an anonymous treasury with proposals and votes) and `deployooor` (deploying
non-native contracts). each is a wasm binary compiled from rust against `src/sdk`, with its zk
circuits next to it as `.zk` files. (`src/contract/README.md`, `src/contract/money/proof/`,
`src/contract/dao/proof/`)

a transaction is a tree of contract calls plus the proofs and signatures that verify them, all
bundled by the sender ahead of time. the node does not compute calldata while a contract runs, the
way an evm does: the sender unrolls the whole call graph, generates every proof, and the verifier
checks the trace. the book gives two reasons. proofs have to be computed before verification anyway,
and a fixed call tree cannot recurse without bound. (`doc/src/spec/concepts.md`,
`doc/src/arch/sc/sc.md`)

contract execution is split into sections with separate permissions. `exec` may read state
(`db_get`) but not write; `update` may write (`db_set`, `merkle_add`) but not read; `metadata`
exports the public inputs that the proofs and signatures need; `deploy` creates databases and
registers circuits. all `exec` sections of a transaction run first, then all updates apply, so one
failing call rejects the whole transaction and no contract can re-enter another mid-call.
(`doc/src/spec/concepts.md`, the host function table)

composition works through coins rather than through calls. a coin carries a `spend_hook` and a
`user_data` field. when the hook is set, spending the coin requires the named contract to appear as
the next call in the same transaction, and that contract reads the user data. the dao treasury is
built this way: coins sent to the treasury point at the dao contract, and `DAO::exec` has to stand
next to the `Money::transfer` that spends them. (`doc/src/arch/dao.md`,
`src/contract/money/proof/burn_v1.zk`)

each contract owns its own key-value databases. the money contract keeps trees named `coins`,
`coin_roots`, `nullifiers`, `nullifier_roots`, `token_freezes`, `fees` and `info`. the node keeps a
sparse merkle tree (the code calls it a monotree) over each contract's state and a global tree over
the per-contract roots; the global root goes into every block header, so a state proof can be
checked against a header. (`src/contract/money/src/lib.rs`, `doc/src/arch/consensus.md`)

the wasm runtime runs on wasmer. the git history shows the first runtime prototype in march 2022 and
the move to wasmer 3 in november 2022. (`src/runtime`, git log)

## zero-knowledge toolchain

proofs are written in zkas, a small assembly-like language with `constant`, `witness` and `circuit`
sections and a `k` parameter that sets the number of rows to 2^k. the compiler in `src/zkas` emits
bincode; the zkvm in `src/zk` is a single halo2 circuit that reads the bincode and configures
itself per proof, so there is one proving system for every contract and no trusted setup. the field
is pallas, and the gadgets come from the halo2 library of the zcash project.
(`doc/src/zkas/zkvm.md`, `doc/src/zkas/writing-zk-proofs.md`, `doc/src/zkas/zkas.md`)

circuits in the tree at this commit: six for money (mint, burn, fee, token mint, mint
authorization, token freeze), nine for the dao (mint, propose main and input, vote main and input,
exec, early exec, and two that authorize money transfers), three under the event graph, twelve
examples in `proof/`, and a larger set under `script/research/zkvm-metering`. (`find . -name
'*.zk'`)

the node verifies proofs and signatures outside the wasm vm, in parallel, because the `metadata`
section exports the public inputs before execution. (`doc/src/arch/sc/sc.md`)

## consensus and chain structure

darkfi mines with randomx, the proof-of-work function monero uses. a miner extends the best ranking
fork it knows with transactions from its mempool, searches for a nonce that puts the header hash
below the difficulty target, broadcasts the block proposal, and runs a confirmation check.
(`doc/src/arch/consensus.md`)

nodes keep several forks in memory. a proposal that extends a known fork at its tip is appended; one
that extends it in the middle creates a new fork; one that extends the confirmed chain starts a new
fork; anything else triggers a reorg check. (`doc/src/arch/consensus.md`, "fork extension")

forks are ranked by work, not by length. each block's rank is the squared distance of its target
from the largest 256-bit integer, with the squared distance of its hash as the tie breaker; a fork's
rank is the sum over its blocks, which always grows when a block is appended. the appendix argues
the tie break from the one-way property of the hash. (`doc/src/arch/consensus.md`, "ranking")

confirmation is a threshold on fork length. once the best fork is longer than the security
threshold, the node moves the fork's first proposal into the confirmed chain and drops every fork
that does not start with it. the page compares this to confirmation counts in other proof-of-work
chains and walks through an example with a threshold of three. the threshold used on the testnet is
not stated on that page. (`doc/src/arch/consensus.md`, "confirmation")

a block header carries the version, previous hash, height, nonce, timestamp, transactions root,
state root and the proof-of-work data; the block carries the header hash, the transaction hashes and
a producer signature. the appendix sizes the indices (`u32` height, `u16` transaction, `u8` call)
and notes that a 90 second block time would take 12,000 years to exhaust the height.
(`doc/src/arch/consensus.md`, appendix)

proof of work replaced an earlier design. `src/consensus`, alive from march 2022 to january 2024,
held a proof-of-stake implementation with leader election over "lead coins" and lead proofs; the
research notes next to it are filed under crypsinous, and the commit history mentions ouroboros and
streamlet. the validator rewrite began in june 2023 and the randomx mining module landed in
september 2023. see [development history](development-history.md). (git history,
`script/research/crypsinous`)

## transaction flow

the sender builds the transaction and its proofs in the wallet and broadcasts it through a node.
nodes validate the state transition against their best fork and keep the transaction in a mempool.
a miner includes it in a proposal; other nodes validate the proposal's transactions against the
extended fork; when the fork confirms, every node validates its transactions in sequence against the
confirmed state and writes the updates. the book separates miners from "spectator" nodes that only
relay and keep transactions in their mempool for a while as spam protection.
(`doc/src/arch/tx_lifetime.md`)

fees are paid in the native token drk through a `Money::Fee` call attached to the transaction. the
fee contract accepts only the native token, checks that the fee is not zero, and adds it to a
per-block-height total in its `fees` tree. the node exposes `tx.calculate_fee` and `tx.simulate`
over rpc. pricing zk work into fees is on the roadmap ("all the zkvm opcodes need to be
benchmarked ... to properly price them for verification fees"). (`doc/src/testnet/token.md`,
`src/contract/money/src/entrypoint/fee_v1.rs`, `doc/src/arch/arch.md`)

## what i could not settle from the public material

- the status table on the architecture page marks every component alpha and the bridge "none". the
  bridge page is a draft about verifiable secret sharing for ethereum deposits, with open questions.
  whether any of it has code is not clear from the public material. (`doc/src/arch/arch.md`,
  `doc/src/arch/bridge.md`)
- the testnet guide carries a disclaimer dated 21 may 2026 that functionality may be missing from
  the deployed alpha testnet. which parts are live at a given moment is not stated anywhere i found.
  (`doc/src/testnet/node.md`)
- "swarming", a per-application subnetwork on top of the p2p layer, is described as a possibly
  post-mainnet feature. an `openspec/` directory with a swarm proposal appeared in august 2026; i
  did not read past the proposal. (`doc/src/arch/net/p2p-network.md`, git history)
- dep 0008, transaction-local state, lets a call spend a coin created earlier in the same
  transaction against a small local merkle tree. the book's table of contents lists it as a draft,
  the page itself says accepted, and `tx_local` flags exist on money inputs and outputs. how far
  the implementation goes i did not trace. (`doc/src/dep/0008.md`,
  `src/contract/money/src/model/mod.rs`)
