# RAM == WIRE == HASH: cell gossip as a below-text benchmark

A benchmark proposal for nullmodem: measure a binary cell exchange against English and
against abios on one shared task, and report bytes on the wire and time to converge.

Labels used below. *Source claim* means the statement comes from the spliceware corpus.
*Proposed* means this note proposes it and it has no standing until someone runs it.

The session behind the corpus was captured by mithrax-exo in November 2026. Spliceware
links pin commit `c1329b2`, the tip of the corpus consolidation branch:
https://github.com/Moissanite-Labs/spliceware/pull/4

## What the corpus claims

**The cell.** One struct of 1,088 bytes: 32 bytes of `left`, 32 bytes of `right`, 1,024
bytes of `body` (source claim,
[Era IV](https://github.com/Moissanite-Labs/spliceware/blob/c1329b2471c81087e36ff11c15309d6b28a8247e/docs/04-cellular-substrate-and-sacred-splicer.md)).
A single 256-bit ARX sponge hashes the whole cell. The hash addresses the cell in an
open-addressed in-memory arena. RAM, the wire format and the hash preimage are the same
bytes, and the later C11 generations pin the struct at 1,088 with a `_Static_assert`
(source claim).

**The datagram.** A cell fits one unfragmented UDP datagram: 1,088 payload bytes against
a 1,472-byte ceiling for a 1,500-byte MTU (source claim). There are no frame headers and
no version byte. Receiver code dispatches on datagram length. Thirty-two bytes is a peer
asking for a hash it does not hold. The larger size is the cell itself (source claim).
There is no control packet in either direction, because the cell carries its own check.

**The fetch.** A hash miss does not block or throw. Inside `resolve()`, a miss puts the
32-byte hash on the socket and returns. A later tick walks the same call path into
opcode execution once the cell lands (source claim,
[v3_evogenesis.c](https://github.com/Moissanite-Labs/spliceware/blob/c1329b2471c81087e36ff11c15309d6b28a8247e/substrates/c11/v3_evogenesis.c)).
Peers exchange only their root hash and pull missing ancestors the same way, so arrival
order does not matter (source claim).

**Lineage.** `left` and `right` are Merkle links to the two parent cells, `ancestor_a`
and `ancestor_b`. A cell names its own history (source claim).

**Behaviour.** Eight bytes of `body` hold four `(in_byte, expected_out_byte)` anchor
pairs. A node checks them on arrival, with no external test packets and no recursive
lookups. A candidate with 4 of 4 anchors and a better fuel margin is promoted to
champion and broadcast. A candidate with 3 or more anchors enters the donor pool as a
crossover partner (source claim,
[Era V](https://github.com/Moissanite-Labs/spliceware/blob/c1329b2471c81087e36ff11c15309d6b28a8247e/docs/05-adaptive-priors-and-lexicase-selection.md)).

**Search state.** An adaptive prior vector sits in the cell body. Winning codons get
their sampling weight raised by a multiplicative update, so the mutation prior travels
with the genome instead of living in an external controller (source claim).

## The structured alternative

Before the collapse to C11, the same design was a Rust/Wasm triad: Extism sandboxes for
hot-swappable Wasm organs keyed by BLAKE3 digest, Iroh for content-addressed QUIC blob
and ticket gossip, and Loro for a causal CRDT with zero-copy forks (source claim,
[Era III](https://github.com/Moissanite-Labs/spliceware/blob/c1329b2471c81087e36ff11c15309d6b28a8247e/docs/03-extiroh-wasm-substrate.md)).
That path carries ordering, retransmission, encryption and CRDT merge, and it is the
harder one to measure. The
[substrate folder](https://github.com/Moissanite-Labs/spliceware/blob/c1329b2471c81087e36ff11c15309d6b28a8247e/substrates/extiroh/README.md)
holds an architecture sketch, not a crate that builds.

## Why measure it here

The charter asks for measurements against a text baseline on the same task. The
interesting quantity is bytes on the wire, and header overhead is part of it. A
1,088-byte payload costs 1,116 bytes with IPv4 and UDP headers. A 32-byte hash query
costs 60. Against a text message of a few hundred bytes the cell is not smaller. Its
claim is that one cell replaces a message plus a schema plus a test exchange, and that
the receiver can verify it alone. That claim is measurable (proposed).

## The benchmark

*Proposed.* One task, three arms, same seeds, same tick count.

**Task.** Two nodes running the evolutionary tick loop already in the repo:
`substrates/c11/run_peers.sh` with `v2_adaptive_loop`, one peer on each port. Each tick
the node steps its champion, tries a splice, scores candidates by fuel and anchors, and
gossips the winner. Run a fixed number of ticks, log every send and receive at the
socket, and repeat each arm three times on loopback and once across two machines on a
LAN.

**Arms.**

| Arm | Unit of exchange | Payload bytes | With IPv4 and UDP headers |
| :--- | :--- | :--- | :--- |
| a. English JSON | one JSON agent message per state change | measured in the run | payload + 28 |
| b. abios SYM-2P v1 | one canonical JSON packet line | 481 for the briefing template | 509 |
| b. abios SYM-2P v2 draft | one positional symbolic line | 101 for the same template | 129 |
| c. cell gossip | 32-byte hash query, 1,088-byte cell reply | 32 and 1,088 | 60 and 1,116 |
| c. cell gossip, gated | same, with 4/4 and 3/4 anchor admission | 32 and 1,088 | 60 and 1,116 |

The abios sizes come from section 8 of
[the v2 draft spec](https://github.com/Moissanite-Labs/abios/blob/main/SPEC-V2-DRAFT.md),
which states the 481-byte and 101-byte lines. The `-rcp` edition of that line adds 17
bytes for a checksum. v1 is the shipped profile, so it is the baseline. The v2 figures
are a draft and belong in the results as a second row, not as the control.

**Recorded per tick, per node.** Bytes sent and received counted at the socket,
including headers. Message or datagram count. Wall time from tick start to applied
state. Ticks to convergence. The final champion hash on each node. Packet loss and
retransmits, where the arm has them.

**What counts as a result.** Fewer bytes for the same applied state, at no worse
convergence time. The correctness check is already printed: both nodes end on the same
champion hash. A run where they disagree is a failed run, not a fast one.

## Limits

Cells are genomes, not prose. A cell carries bytecode, lineage, anchors and priors.
Comparability holds for structured coordination payloads: a state change, a hash, an
accept or reject. It does not extend to open-ended dialogue, negotiation, or any
exchange a person has to read to follow. A result from this benchmark says nothing about
replacing English conversation (proposed).

The C11 substrate is a sketch. It builds and runs and nothing more: no test suite beyond
the tick loop, no fuzzing, no timing. Checked on 2026-11-18, `make -C substrates/c11`
builds four binaries clean under `-Wall -Wextra`, and a two-peer loopback run reached
eight ticks on each node with both ending on the same champion hash. That is a smoke
check. It is not evidence of correctness.

The transport is bare UDP. No delivery guarantee, no ordering, no retransmission. The
design treats the next tick as the recovery path. Loss belongs in the results, not in
the assumptions.

The sketch sends plaintext. An earlier era describes a 44-byte encrypted query carrying
a nonce and the encrypted hash. The C11 code sends the raw 32 bytes and the raw cell,
with no signature. A benchmark run therefore measures an unauthenticated channel, and
the audit story for a real deployment needs signing work that does not exist yet.

The 8-byte anchor block is a narrow check. Four input and output pairs show a candidate
did not break four known cases. They do not show it behaves on other inputs, and they do
not stop a candidate that is wrong in a way the anchors miss. Admission by anchors is a
filter, not a test suite.

Iroh and Loro have no buildable code in the corpus. That path is cited as the structured
alternative, not run as an arm. Comparing cell gossip to a CRDT would need a working
client first.

The sources pin a branch tip that is still in review, so line numbers and file layout
may move.

## What a result would settle

A win on bytes for the same applied state moves the question to oversight, which the
charter already sets as a success condition: every non-text exchange needs a readable
audit form. For cells, the cheap version is a renderer that turns one cell into a line a
person can read, with its hash, its two parents, its anchors and its opcodes. Without
that, a faster channel is also a less visible one.

A loss is a published negative result. The charter asks for those too.

The corpus map and provenance table are in
[the corpus index](https://github.com/Moissanite-Labs/spliceware/blob/c1329b2471c81087e36ff11c15309d6b28a8247e/docs/INDEX.md).
