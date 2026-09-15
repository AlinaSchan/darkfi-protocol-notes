# darkfi protocol notes

independent notes on darkfi's architecture, privacy model, network and development history.

i started looking at darkfi because i am interested in systems where privacy changes what an outside
observer can actually learn from the network.

these are personal research notes, not official darkfi documentation.

## what darkfi is

darkfi describes itself as a layer one proof-of-work blockchain for anonymous applications. every
operation on it is a smart contract call. payments, token issuance, swaps and the dao are native wasm
contracts that carry zero-knowledge proofs written in the project's own language, zkas, and checked by
a halo2 based virtual machine with no trusted setup. (`doc/src/arch/overview.md`, `doc/src/zkas/zkvm.md`)

the repository has been public since august 2020. in september 2026 the project runs an alpha testnet.
the node config has a mainnet section, and the testnet guide says mainnet is not active.
(`doc/src/testnet/node.md`, git history)

the code is rust under the agpl 3.0. the readme points to the project's own git host at git.dark.fi,
which answers as a gogs instance; the same repository is on github under `darkrenaissance/darkfi`.
(`README.md`, `LICENSE.md`)

## what these notes cover

- [architecture](notes/architecture.md): the components, where they live in the tree, the contract
  and execution model, consensus and chain structure
- [privacy](notes/privacy.md): what a transaction reveals and what it hides, the primitives, how
  coins, nullifiers and encrypted notes fit together
- [network](notes/network.md): the p2p layer, sessions and transports, peer discovery, what a peer
  learns about you
- [observable state](notes/observable-state.md): the question that started this. what can an outside
  observer actually see?
- [development history](notes/development-history.md): what the git history shows, with numbers from
  [repository-history-tools](https://github.com/alinaschanz/repository-history-tools)

[sources.md](sources.md) lists the primary sources. paths in the notes such as
`doc/src/arch/consensus.md` point into the darkfi repository at commit `a5514cb2` (2026-09-10). the
book at dark.fi/book is built from `doc/src`, so every page cited here is readable there as well.

## why it interests me

on a transparent chain you read balances, counterparties and contract calls straight from the ledger.
on darkfi most of that is a commitment or a ciphertext. what stays public (nullifiers, merkle roots,
which contract was called, the shape of the transaction) is a different kind of evidence, and i wanted
to write down which is which, and where the public material leaves the line unclear.

the second reason is the infrastructure around the chain. the project hosts its own git, runs its own
irc over its own p2p library, tracks tasks in its own p2p task manager, and mines with randomx, which
can be merge mined with monero. a large share of the code exists so that the project depends on
nobody else's service.

## how i read it

repository first, then the book, which lives in the same repository, then the code where the book and
the code disagree. where they disagree, the notes say so. where i could not confirm something, the
notes say "not clear from the public material" instead of guessing.

## disclaimer

i am not affiliated with darkfi and do not speak for the project. the notes can be wrong; the sources
are listed so you can check them. corrections are welcome as issues or pull requests.
