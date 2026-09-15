# observable state

the question behind these notes: what can an outside observer actually see?

"outside observer" here means someone with a synced node or access to a node's public rpc, plus
whatever the p2p layer shows to a peer. no wallet keys, no special position in the network. paths
point into the darkfi repository at commit `a5514cb2` (2026-09-10).

## directly observable

- every block header: version, previous hash, height, nonce, timestamp, transactions root, state
  root, proof-of-work data; and the block's producer signature. (`doc/src/arch/consensus.md`)
- every transaction: the tree of calls with contract ids and payload bytes, the proofs, the
  signatures. (`doc/src/spec/concepts.md`, `doc/src/arch/tx_lifetime.md`)
- per money input: nullifier, the merkle root used, a value commitment, a token commitment, an
  encrypted user data field, the spend hook, the signature public key, the transaction-local flag.
  per output: coin commitment, value commitment, token commitment, an encrypted note.
  (`src/contract/money/src/model/mod.rs`)
- the fee of every transaction, in the native token, and the running fee total per block height.
  (`src/contract/money/src/entrypoint/fee_v1.rs`)
- contract state: the coins tree, the nullifier set, their roots, the token freeze table, the fee
  table, and the wasm and zkas circuits of every deployed contract, all readable over rpc
  (`blockchain.get_contract_state`, `blockchain.lookup_wasm`, `blockchain.lookup_zkas`).
  (`src/contract/money/src/lib.rs`, `bin/darkfid/src/rpc/`)
- difficulty and the current target; pending transactions and new proposals, through subscriptions
  on a node you run. (`bin/darkfid/src/rpc/`)
- on the network: the seed addresses in the default configuration, other nodes' whitelists on
  request, and each peer's version message (app name, protocol version, clock, advertised external
  addresses). (`doc/src/dep/0007.md`, `src/net/message.rs`)
- on darkirc: the events of public channels, with timestamps, layers and parent links.
  (`doc/src/misc/event_graph/network_protocol.md`)
- the source, the book, the enhancement proposals, and the task list if you run `tau`.

## derivable

- activity per application over time: contract ids are public and the function is encoded in the
  payload, so counting transfers, token mints, dao proposals, votes and executions per block is a
  parsing exercise. (`doc/src/arch/sc/sc.md`, the abi section)
- the number of coins ever created and the number spent, from the sizes of the coins tree and the
  nullifier set, hence the number of unspent coins. not their values, not their owners.
  (`src/contract/money/src/lib.rs`)
- block intervals, transactions per block, an estimate of hashrate from the difficulty, and which
  producer keys mine how often, from the block signatures. (`doc/src/arch/consensus.md`)
- the shape of a transaction as a fingerprint: a plain transfer, a fee call plus a transfer, a
  transfer next to a dao execution. the shape is public even when every value is hidden.
  (`doc/src/arch/sc/tx-lifetime.md`)
- which transports a node accepts and roughly when it is online, from the addresses it advertises
  and the `last_seen` stamps that travel with whitelists. (`doc/src/arch/net/p2p-network.md`)
- with several nodes in good positions, the peer a transaction first arrived from, since the p2p
  layer itself does not mix or delay; whether that peer is the sender depends on the transport the
  sender chose. (`doc/src/start-here.md`)

## intentionally hidden

- values and token ids of inputs and outputs (commitments). (`burn_v1.zk`, `mint_v1.zk`)
- sender and recipient keys (inside the coin commitment and the encrypted note).
  (`doc/src/arch/tx_lifetime.md`)
- which earlier output an input spends; only a nullifier and a root are revealed.
  (`doc/src/arch/anonymous_assets.md`)
- the memo. (`Note` in `doc/src/arch/tx_lifetime.md`)
- the owner behind a spend signature: the signing key is derived from a separate secret, not from
  the coin's key. (`burn_v1.zk`)
- dao membership, treasury balance, proposal contents and vote weights, by design; votes are
  encrypted to the dao's votes key and summed by whoever holds it. (`doc/src/start-here.md`,
  `doc/src/arch/dao.md`)
- a node's clearnet address, when it runs over tor, i2p or nym and its peers do too, within that
  transport's threat model. (`doc/src/start-here.md`)

## unknown or unclear from the public material

- what a dao vote transaction reveals beyond "a vote happened": whether the proposal or the dao it
  belongs to is public in the call parameters. the answer is in `vote-main.zk` and the dao model,
  which i have not traced.
- what a token mint reveals about the token id and the mint authority, and what a freeze reveals.
  the money contract keeps a `token_freezes` tree, so something about frozen tokens is public.
- what an atomic swap looks like on chain, and whether it can be told apart from a transfer with
  the same number of inputs and outputs. (`doc/src/testnet/atomic-swap.md` describes the wallet
  flow, not the on-chain form.)
- whether the fee amount, being public, correlates with anything else about the transaction beyond
  its shape. the roadmap says fees will price zk work; if so, the fee restates the shape.
- how transactions are relayed between nodes, and therefore how much a network observer can
  learn without controlling many peers.
- what the official explorer shows; `bin/explorer` has a python web frontend that i did not run.
- whether the encrypted user data field on inputs leaks structure across the calls of one
  transaction. it is a commitment with a blind, per the circuit, so probably not, but i did not
  check how contracts use it.

## a note on the two kinds of evidence

on a transparent chain the evidence is the ledger: balances, counterparties, calls. on darkfi the
ledger keeps commitments and ciphertexts, and the evidence that remains is structural: how many, how
often, in what shape, at what time, from which peer. that is a smaller set, and a different one. the
lists above are my attempt to draw the line where the public material draws it, and to mark where it
does not.
