# network

darkfi's peer-to-peer layer, as the book and `src/net` describe it. paths point into the repository
at commit `a5514cb2` (2026-09-10).

## one library, many daemons

`src/net` is the project's own p2p library. `darkfid`, `darkirc`, `taud`, `fud` and `lilith` each
run an instance of it; the blockchain is one application among several on the same network code.
the directory has existed since december 2020 ("added working p2p network") and is the most
changed directory of the repository below the root. (`doc/src/start-here.md`, git history)

## sessions, channels, protocols

a node runs sessions: an inbound session for connections it accepts, an outbound session that keeps
a number of slots filled with connections it makes, a seed session that contacts seed nodes to fill
the hosts pool and then stops, and manual, direct and refine sessions for configured peers, direct
connections and greylist housekeeping. every connection is wrapped in a channel, and protocols
attach to channels per session. the base protocols are version, ping, address exchange, seed and a
hole punch protocol; applications register their own. (`doc/src/arch/net/p2p-network.md`,
`src/net/session/`, `src/net/protocol/`)

## transports

the transport directory holds tcp, tls, tor, i2p (added april 2025), nym, socks5, quic and unix
sockets. addresses are urls with the transport as the scheme (`tcp+tls://`, `tor://`, `i2p://`,
`socks5://host:port/onion`). since dep 0007 each transport has its own profile with its own seeds,
peers, inbound addresses and timeouts, so that a node configured for tor does not dial clearnet
seeds by accident. the user guides cover tor, i2p and nym setups. (`src/net/transport/`,
`doc/src/dep/0007.md`, `doc/src/misc/nodes/`)

the start-here page states the privacy property with care: an anonymity transport "can hide a
node's clearnet address from application peers, subject to that transport's threat model". the p2p
layer adds no mixing or cover traffic of its own; that job is delegated to tor, i2p or nym.
(`doc/src/start-here.md`)

## peer discovery

nodes keep three lists. a greylist, up to 5,000 entries, of addresses they have heard about; a
whitelist, up to 1,000, of addresses that answered a ping; and an anchorlist of hosts they have
connected to. a `GetAddr` request is answered with the whitelist, received addresses go to the
greylist, and a housekeeping task pings random greylist entries and promotes the ones that answer.
on shutdown the whitelist is demoted to the greylist, so every entry is checked again on start. the
page credits the monero p2p design for the scheme. (`doc/src/arch/net/p2p-network.md`, "hostlist
filtering")

seed nodes are `lilith` instances. the dep 0007 example config lists two clearnet seeds under
dark.fi, two tor onion seeds and one i2p seed. (`doc/src/dep/0007.md`)

## what a peer learns about you

the version message sent on connect carries an app name, the protocol version, a unix timestamp,
the address the peer connected to (before and after resolving), the node's own external addresses
if it advertises any, a feature list, and a `node_id` string that the source comments as "only used
for debugging. compromises privacy when set". so a peer sees your transport address, your software
and version, your clock, and the addresses you accept connections on.
(`src/net/message.rs`, `VersionMessage`)

whitelists travel. when you answer `GetAddr` you hand out the addresses you consider live, and your
own address ends up in other nodes' lists once you accept inbound connections. each entry carries a
`last_seen` timestamp. (`doc/src/arch/net/p2p-network.md`)

## what the chain protocols expose

darkfid nodes relay transactions and block proposals and answer json-rpc. the rpc offers block,
transaction and contract state queries, subscriptions to blocks, proposals and transactions,
difficulty and target queries, fee calculation and transaction simulation, and management calls
such as `dnet.switch`. the port table separates a public rpc, a restricted rpc, a stratum port for
miners and a merge mining port. anyone who reaches a node's public rpc can read the confirmed chain
and watch new transactions arrive. (`bin/darkfid/src/rpc/`,
`doc/src/misc/nodes/node-configurations.md`)

`dnet` is a viewer for a node's own p2p traffic, fed by `dnet.subscribe_events` once the operator
switches it on. it exists for debugging and shows what the node sees, which is also what a node
operator can log. (`bin/dnet`, `src/net/dnet.rs`, `bin/darkfid/src/rpc/management.rs`)

## the event graph

darkirc and tau do not use the blockchain. they use the event graph: an append-only dag of events
with parents, a timestamp, a layer and content. a joining node asks its peers for their tips, walks
backwards to fetch what it lacks, and orders events by a depth-first search sorted on timestamps
with the event id as the tie break. events spread by rebroadcast, and a node refuses an event whose
parents it has not seen until it fetches them. darkirc rotates to a new dag every hour and can run
archive nodes that keep everything. (`doc/src/misc/event_graph/event_graph.md`,
`doc/src/misc/event_graph/network_protocol.md`, `doc/src/arch/arch.md`)

for an observer this means public irc channels are public, with dag structure and timestamps
visible to every node. message privacy on darkirc depends on the configured encryption and the
transport, which the start-here page says outright. (`doc/src/start-here.md`)

## security design notes

the p2p page names the attacks it cares about (sybil, eclipse, denial of service), states that node
addresses cannot be trusted because the network is meant to run over anonymity networks, and
describes a scoring subsystem with per-channel work and byte scores that decay exponentially and
end in throttling or a ban. the page marks the scoring subsystem, the abstract computer and the
controller as todo. a metering module exists in the tree and messages carry a metering
configuration; how much of the described design it covers i did not trace. the same page describes
swarming, per-application subnetworks, as a possibly post-mainnet feature, with a
`PeerDiscoveryBase` trait already in place to make room for it.
(`doc/src/arch/net/p2p-network.md`, `src/net/metering.rs`, `src/net/message.rs`)

optional rate-limit nullifiers exist for darkirc but are off by default, with "compiled
pregenerated identities rather than a public staking/free-tier workflow". (`doc/src/start-here.md`)

## not clear from the public material

- how transactions are relayed between nodes in practice (flooding, inventory messages, something
  else). the p2p page holds suggestions ("do not forward orphan txs", "limit invs to 2/s") rather
  than a protocol description, and i did not read the relay protocol in `bin/darkfid`.
- whether the testnet is reachable through nym today, or only through tor and i2p. the nym
  transport file and a nym guide exist; that is all i can confirm.
- how the hole punch protocol and the upnp module are used, and whether they are on by default.
