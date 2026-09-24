# development history

what the public git history of darkfi shows. the numbers come from
[repository-history-tools](https://github.com/AlinaSchan/repository-history-tools) run on
2026-09-15 against the github copy of the repository at commit `a5514cb2` (2026-09-10). dates are
committer dates in utc, rename detection is off, and merge commits count as commits without file
changes; see the tool's readme for what that does to the counts. the csv files themselves are in
[data/](../data/), so every number below can be checked or recomputed.

this page reads directories and commit messages. it does not read minds: a directory that changed a
lot changed a lot, and that is all the number says.

## the whole repository

| | |
|---|---|
| first commit | 2020-08-20, "initial commit" |
| head | 2026-09-10 |
| commits | 11,375 (144 merges) |
| authors by name, after `.mailmap` | 104 |
| authors with 100 or more commits | 19 |
| share of the five most frequent authors | 53% of commits |
| files at head | 1,688 |
| paths that ever changed | 6,032 |
| tree size at head | 28.2 mb of blobs |

commits per year: 300 in 2020 (from august), 1,885 in 2021, 3,081 in 2022, 2,348 in 2023, 1,891
in 2024, 1,125 in 2025, 745 in 2026 to 10 september. the busiest months were september and october
2021 (405 and 398 commits), august 2023 (370), january 2024 (362) and april 2022 (351).

few merge commits and a mostly linear mainline: the project rebases or commits straight to master
far more than it merges. the readme sends contributors to a self-hosted gogs instance, and the
contributor guide points at an irc channel and a p2p task manager rather than at pull requests.
(`README.md`, `doc/src/dev/contrib/contrib.md`)

## directories by commits

top-level, whole history:

| directory | commits | files ever | first | last | at head |
|---|---|---|---|---|---|
| `src` | 5,090 | 1,107 | 2020-08 | 2026-09 | yes |
| `bin` | 3,991 | 1,368 | 2021-12 | 2026-09 | yes |
| root files | 1,340 | 44 | 2020-08 | 2026-09 | yes |
| `doc` | 941 | 510 | 2021-03 | 2026-09 | yes |
| `script` | 882 | 2,015 | 2021-09 | 2026-08 | yes |
| `example` | 496 | 301 | 2021-09 | 2026-08 | yes |
| `contrib` | 293 | 182 | 2021-12 | 2026-08 | yes |
| `lisp` | 210 | 31 | 2020-10 | 2021-09 | no |
| `scripts` | 172 | 86 | 2020-09 | 2021-09 | no |
| `tests` | 131 | 17 | 2022-03 | 2026-08 | yes |
| `proof` | 77 | 23 | 2021-09 | 2026-01 | yes |

one level down, the twenty directories with the most commits:

| directory | commits | first | last | at head |
|---|---|---|---|---|
| `src/net` | 953 | 2020-12 | 2026-08 | yes |
| `doc/src` | 871 | 2022-03 | 2026-09 | yes |
| `src/contract` | 807 | 2022-11 | 2026-09 | yes |
| `script/research` | 758 | 2021-09 | 2026-08 | yes |
| `src/bin` | 656 | 2020-10 | 2022-01 | no |
| `src` (files at that level) | 608 | 2020-08 | 2026-08 | yes |
| `bin/app` | 544 | 2025-01 | 2026-09 | yes |
| `bin/darkfid` | 505 | 2022-01 | 2026-08 | yes |
| `bin/tau` | 497 | 2022-04 | 2026-08 | yes |
| `bin/drk` | 494 | 2022-01 | 2026-08 | yes |
| `src/consensus` | 449 | 2022-03 | 2024-01 | no |
| `bin/darkwallet` | 438 | 2024-05 | 2025-01 | no |
| `src/sdk` | 418 | 2022-03 | 2026-08 | yes |
| `bin/darkirc` | 408 | 2023-04 | 2026-08 | yes |
| `src/rpc` | 326 | 2021-05 | 2026-08 | yes |
| `src/util` | 317 | 2021-09 | 2026-08 | yes |
| `bin/ircd` | 311 | 2021-12 | 2023-06 | no |
| `src/validator` | 311 | 2023-06 | 2026-08 | yes |
| `src/zk` | 306 | 2022-01 | 2026-09 | yes |

further down the list: `src/blockchain` (294 commits, since 2021-06), `src/crypto` (282, from
2021-04 to 2022-11, gone), `src/wallet` (279, from 2021-05 to 2024-01, gone), `src/event_graph`
(233, since 2023-02), `src/runtime` (226, since 2022-03) and `src/zkas` (113, since 2022-01).

the single most changed files are `Cargo.lock` and `Cargo.toml` (629 and 620 commits), then
`src/error.rs` (295), the old `src/bin/darkfid.rs` (263, gone since 2022-01), the old
`src/consensus/state.rs` (248, gone since 2024-01), and the p2p files `src/net/session/
outbound_session.rs` (206), `src/net/hosts.rs` (195) and `src/net/channel.rs` (160).

what the biggest line counts are made of: `script/research` has 1,959 files and 251,000 inserted
lines, much of it simulation output (a `lotterysim` directory from spring 2023 holds several
33,000-line text files); `contrib` once held solana and erc20 token lists of 196,000 and 91,000
lines, added in december 2021 and deleted in june 2023; `bin/app` carries generated shape and
version-block data files of 10,000 to 16,000 lines each. lines inserted are a poor measure of work
here.

## what the paths show over time

the sequence below is directory births, deaths and renames plus the first commit message that
mentions a term. it is a reading of the log, not of the team's reasons.

- **2020-08 to 2020-12.** the initial commit is followed by mimc and zk experiments; a "zkvm spec"
  commit on 2020-10-15. proof generation moves from jinja templates to lisp/racket on 2020-10-17
  (`lisp/`, gone 2021-09-13). "added working p2p network" on 2020-12-20 starts `src/net`.
- **2021.** "halo2 examples" on 2021-06-27. `src/rpc` from may. the research directory starts in
  september, the busiest month of the whole history. the zkas compiler lands on 2021-10-29. the
  first tag, `v0.0.1`, is dated 2021-11-25. an irc daemon moves into `bin/ircd` on 2021-12-25.
  `contrib` starts in december with the token lists.
- **2022.** `v0.2.0` and `v0.3.0` in january; the same month `src/zk` is created and the old
  `src/bin` moves to `example/`. "separate ouroboros from streamlet" on 2022-01-30; the crypsinous
  research notes on 2022-03-28, the same day `src/consensus` is born. the wasm runtime prototype
  arrives 2022-03-16, `src/sdk` the same month, `bin/tau` in april, a sled blockchain store
  skeleton in april. the book (`doc/src`) starts on 2022-03-08. the event graph spec is written on
  2022-10-12; `src/contract/money` is outlined on 2022-11-15; "get wasm code compiling with
  wasmer3" on 2022-11-03. this is the year with the most commits.
- **2023.** `v0.4.0` on 2023-02-01 and `v0.4.1` on 2023-02-27 ("consensus: set testnet bootstrap
  timestamp"). `src/event_graph` on 2023-02-19; `ircd2` is renamed to `darkirc` on 2023-04-25 and
  the old `ircd` dropped on 2023-06-29. "set foundation for validator rewrite" on 2023-06-23. the
  first commit mentioning randomx is "research/pow: high level randomx mining layout" on
  2023-08-26, and "validator: pow module added for mining" follows on 2023-09-20. fuzzing starts
  in august. merge mining with monero appears in november ("mmproxy/monero: implement blocktemplate
  modification for merge mining").
- **2024.** `src/consensus` receives its last commit on 2024-01-29; the proof-of-stake code goes
  with it. january 2024 is the fourth busiest month. `bin/darkwallet` is added on 2024-05-03. a
  `prealpha-unstable` tag is dated 2024-10-02.
- **2025.** `bin/darkwallet` becomes `bin/app` on 2025-01-21 and the `0.1-alpha` tag follows on
  2025-01-30; `app-0.2-alpha` in may, `app-0.2.4-alpha` in june, `app-0.3-alpha` on 2025-11-01.
  the i2p transport lands on 2025-04-23, a "barebones" monotree on 2025-05-01, a proof-of-work data
  enum on the block header on 2025-05-07, and `v0.5.0` on 2025-06-06.
- **2026.** `AGENTS.md` and an `.opencode` directory on 2026-08-27 ("chore: add opencode/openspec
  agents"), an `openspec/` directory with a swarm proposal on 2026-08-30. the remote carries a
  branch named `hardening-202606`. the last commit of the clone, 2026-09-10, is on master.

## what the numbers do not say

the history does not say why proof of stake was replaced, how many people work on the project, or
what the alpha tags mean for users. author names are whatever each committer configured; 104 names
is an upper bound on people, not a count. the github copy may lag or differ from git.dark.fi, and
i compared neither the two hosts nor the branches beyond their names.
