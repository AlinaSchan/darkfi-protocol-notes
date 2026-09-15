# privacy

what a darkfi transaction shows, what it hides, and which primitives do the hiding. paths point into
the darkfi repository at commit `a5514cb2` (2026-09-10); the `.zk` files are the circuits as they
exist at that commit.

## the model in one paragraph

darkfi payments follow the sapling scheme from zcash. a coin is a poseidon commitment to its owner's
public key, a value, a token id, a spend hook, a user data field and a random blind. spending a coin
reveals a nullifier derived from the owner's secret and the coin, plus the merkle root against which
the proof showed that the coin exists; creating a coin reveals the new commitment. nothing on chain
links a nullifier to the commitment it came from, so inputs and outputs are unlinkable to each other
and to their history. (`doc/src/arch/tx_lifetime.md`, `doc/src/arch/anonymous_assets.md`,
`src/contract/money/proof/burn_v1.zk`)

## what stays public in a transfer

per input the chain sees: a pedersen commitment to the value, a poseidon commitment to the token id,
the nullifier, the merkle root the proof used, an encrypted user data field, the spend hook in the
clear (zero when unused), the public key that signs the transaction, and a flag that says whether the
input is transaction-local. (`src/contract/money/src/model/mod.rs`, `Input`)

per output: the value commitment, the token commitment, the coin commitment, and an encrypted note
(ciphertext plus an ephemeral public key) that only the recipient can open. (`Output`, same file)

for the transaction as a whole: the list of calls with their contract ids and payloads, the proofs,
and the schnorr signatures over everything but the signatures. the number of inputs and outputs is
plain in the payload. (`doc/src/arch/tx_lifetime.md`, `doc/src/spec/concepts.md`)

## what stays hidden

the value and token of every input and output, the sender's and the recipient's keys, which earlier
output an input spends, and the memo. the note that carries the serial, value, token id, both
blinds and the memo is encrypted to the recipient's public key with an ephemeral key exchange, so
the recipient finds their coins by trial decryption while scanning blocks.
(`doc/src/arch/tx_lifetime.md`, `Note` and `EncryptedNote`; `doc/src/arch/wallet.md`)

the signature key deserves a line of its own. the burn circuit takes a separate `signature_secret`
witness and reveals the public key derived from it, next to the `coin_secret` that derives the
nullifier and the coin. so the key that signs a spend is not the key that owns the coin. the older
transactions page shows pseudo-code in which one secret does both; the circuit is the one to trust.
(`src/contract/money/proof/burn_v1.zk`, `doc/src/arch/tx_lifetime.md`)

## primitives

- curves: pallas and vesta, the 2-cycle used by halo2 and zcash orchard. circuits work in the pallas
  base field. (`doc/src/spec/crypto-schemes.md`)
- proof system: halo2 in the zcash implementation, one circuit configured by zkas bincode, no
  trusted setup. (`doc/src/zkas/zkvm.md`)
- in-circuit hash: poseidon with the halo2 parameters (x^5 s-box, 8 full rounds, 56 partial
  rounds). coins, nullifiers, token commitments and "bullas" (the project's word for a commitment to
  an object's parameters plus a blind) are all poseidon hashes. (`doc/src/spec/crypto-schemes.md`)
- value commitments: homomorphic pedersen commitments on pallas, with generators from the zcash
  group hash under the "z.cash:Orchard-cv" domain. the homomorphism lets the verifier check that
  inputs and outputs balance without seeing a single value. (`doc/src/spec/crypto-schemes.md`,
  `doc/src/arch/sc/tx-lifetime.md`)
- token commitments: the spec page still describes them as pedersen; the current circuit hashes
  the token id with a blind, with the comment that poseidon is cheaper and the token commitment
  needs no homomorphic property. (`burn_v1.zk`, `doc/src/spec/crypto-schemes.md`)
- the coin set: an append-only incremental merkle tree of depth 32 over pallas field elements.
  (`doc/src/spec/crypto-schemes.md`)
- note encryption: chacha20-poly1305 with a key from blake2b over the diffie-hellman shared secret
  and the ephemeral public key. a second scheme, elgamal-style with poseidon, is verifiable inside
  a circuit and serves where a contract must prove something about an encrypted value, such as dao
  votes. (`doc/src/spec/crypto-schemes.md`, `src/sdk/src/crypto/note.rs`)
- hashes outside circuits: blake2b in the spec, blake3 for event ids and for the state trees.
  (`doc/src/spec/crypto-schemes.md`, `doc/src/misc/event_graph/network_protocol.md`,
  `doc/src/arch/consensus.md`)

## composition without opening the coin

a coin can carry a `spend_hook` naming a contract and a `user_data` field for that contract. the
hook is revealed when the coin is spent, the user data leaves the circuit as a commitment, and the
named contract must appear as the next call. the dao treasury works this way: its coins name the dao
contract, and `DAO::exec` proves, without opening the coins, that a proposal to spend them passed.
(`doc/src/arch/dao.md`, `burn_v1.zk`)

## the dao

the dao page describes the dao as a bulla, a poseidon commitment over its parameters: proposer
limit, quorum, early execution quorum, approval ratio, governance token, and six public keys for
viewing notes, proposing, viewing proposals, viewing votes, executing and early executing.
proposals are bullas too. votes are additively homomorphic commitments, each encrypted to the dao's
votes key; whoever holds that key sums the votes and proves the result in zero knowledge.
(`doc/src/arch/dao.md`, `src/contract/dao/proof/mint.zk`)

the start-here page states the goal plainly: "you cannot see who is in the dao", and neither the
treasury's balance nor the vote weights. how much of the voting design on the dao page is
implemented as written (the page weighs three ways of proving coin ownership at vote time) is not
clear from the public material; the nine dao circuits in the tree are the concrete part.
(`doc/src/start-here.md`, `src/contract/dao/proof/`)

## fees

`Money::Fee` is its own call with its own circuit. it burns one coin, mints one change coin, and
takes the fee amount and its blind as parameters in the clear. the contract accepts only the native
token, rejects a fee of zero, checks that the value commitments balance against a commitment to the
public fee, and adds the amount to a per-block-height total. so the fee of every transaction is
public, while the change output stays hidden. (`src/contract/money/proof/fee_v1.zk`,
`src/contract/money/src/entrypoint/fee_v1.rs`, `MoneyFeeParamsV1`)

## what the model does not hide

- the transaction shape: how many calls, which contracts, how many inputs and outputs, the proof
  sizes. a transfer with one input and two outputs looks like every other such transfer, but a dao
  execution looks like a dao execution.
- timing: the block a transaction landed in, and the block's timestamp.
- the miner, by key: every block carries a producer signature and the proof-of-work data.
- the fee, as above.
- network metadata: who broadcast the transaction first, and from which address, is a property of
  the p2p layer and the chosen transport, not of the proofs. see [network](network.md).
- viewing keys: the contributor guide lists "Money::transfer() contract viewing keys" among the
  harder open crypto tasks, so disclosure to an auditor without handing over the spending key is not
  a documented feature at this commit. (`doc/src/dev/contrib/contrib.md`)

## audits

the start-here page links one crypto audit, by zksecurity, filed as q1 2024. i have not read it;
the link is in [sources](../sources.md). (`doc/src/start-here.md`)
