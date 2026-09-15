# sources

primary sources only. everything in the notes cites one of these.

## the repository

- canonical host, as named in the readme: https://git.dark.fi/darkrenaissance/darkfi (a gogs
  instance, going by the cookie it sets)
- github copy: https://github.com/darkrenaissance/darkfi
- commit read for these notes: `a5514cb2cecdc791ed8de68440455433beeebe7a`, dated 2026-09-10, cloned
  from github on 2026-09-15
- license: `LICENSE.md` (gnu agpl 3.0); the source headers name the dyne.org foundation as
  copyright holder, 2020 to 2026

## the book

the book at https://dark.fi/book/ is built from `doc/src` in the repository. a path
`doc/src/<dir>/<page>.md` in the notes corresponds to `https://dark.fi/book/<dir>/<page>.html`.
pages cited:

- `doc/src/start-here.md`: directory structure, module overview, links to the audit and the zk
  toolchain
- `doc/src/arch/arch.md`: component status table, release cycle, mainnet roadmap
- `doc/src/arch/overview.md`: key terms (marked out of date by the contributor guide)
- `doc/src/arch/consensus.md`: mining loop, fork ranking, confirmation, block and header structure,
  the state trees
- `doc/src/arch/tx_lifetime.md`: transaction lifetime, the transaction object, the payment scheme
- `doc/src/arch/sc/sc.md` and `doc/src/arch/sc/tx-lifetime.md`: the contract model, call trees,
  sections and permissions, composition through spend hooks
- `doc/src/arch/anonymous_assets.md`: mint and burn
- `doc/src/arch/dao.md`: dao parameters, proposals, votes, execution, open design questions
- `doc/src/arch/net/p2p-network.md`: sessions, hostlists, attacks and mitigations, scoring,
  swarming
- `doc/src/arch/tooling.md`, `doc/src/arch/services.md`, `doc/src/arch/wallet.md`,
  `doc/src/arch/bridge.md`
- `doc/src/spec/concepts.md`: transactions, wasm sections, host functions
- `doc/src/spec/crypto-schemes.md`: poseidon, bullas, pallas and vesta, pedersen commitments, the
  merkle tree, note encryption, key agreement
- `doc/src/zkas/zkvm.md`, `doc/src/zkas/writing-zk-proofs.md`, `doc/src/zkas/zkas.md`
- `doc/src/misc/event_graph/event_graph.md` and `doc/src/misc/event_graph/network_protocol.md`
- `doc/src/misc/nodes/node-configurations.md`: default ports
- `doc/src/dep/0007.md` (network profiles, accepted) and `doc/src/dep/0008.md` (transaction-local
  state)
- `doc/src/testnet/node.md`, `doc/src/testnet/token.md`, `doc/src/testnet/merge-mining.md`,
  `doc/src/testnet/atomic-swap.md`
- `doc/src/dev/contrib/contrib.md`: how to contribute, areas of work

## code files cited

- `Cargo.toml` (workspace members)
- `src/contract/README.md`, `src/contract/money/src/lib.rs`, `src/contract/money/src/model/mod.rs`,
  `src/contract/money/src/entrypoint/fee_v1.rs`
- `src/contract/money/proof/burn_v1.zk`, `mint_v1.zk`, `fee_v1.zk`; `src/contract/dao/proof/`
- `src/net/message.rs`, `src/net/metering.rs`, `src/net/transport/`, `src/net/session/`,
  `src/net/protocol/`
- `bin/darkfid/src/rpc/` (json-rpc method names), `bin/fud/README.md`
- git tags and log (`git tag`, `git log`), summarized in the development history

## website

- https://dark.fi (landing page, read 2026-09-15: "an l1 for anonymous engineering"; the app
  release for android, linux, windows and macos)
- https://dark.fi/manifesto.html, linked from the readme, not used for technical claims

## audit

- zksecurity, darkfi code audit, q1 2024: https://dark.fi/zksecurity-audit-q124.pdf, linked from
  `doc/src/start-here.md`. listed, not read.

## works the book cites and i mention

- randomx: https://github.com/tevador/RandomX
- halo2: https://github.com/zcash/halo2
- zcash protocol specification, section on the group hash for pallas and vesta:
  https://zips.z.cash/protocol/protocol.pdf
- poseidon (grassi, khovratovich, rechberger, roy, schofnegger, 2019):
  https://eprint.iacr.org/2019/458.pdf
- "exploring the monero p2p network" (cao et al., 2019), the hostlist design:
  https://eprint.iacr.org/2019/411.pdf

## tooling

- repository-history-tools, run 2026-09-15 with default options against the commit above:
  https://github.com/alinaschanz/repository-history-tools
