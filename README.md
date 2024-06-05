# JAM

This is Eiger's public planning and research repository for a possible JAM implementation.
We are investigating and digging into the Graypaper deeply, with the purpose of creating a development plan for it.
The following information is frequently and constantly revised.
We are trying to share our current state of understanding, thoughts, open questions, and possible work packages.
You are invited to share your thoughts, inputs, and questions with us.

## For the Introduction

Optionally, have a look at the following videos and informational sites:
- [Polkadot's Future: The Big Jam - TOKEN2049 Dubai 2024](https://www.youtube.com/watch?v=xTMiE0UcZUo)
- [sub0 Asia 2024 keynote - Gavin Wood on JAM A-Z](https://www.youtube.com/watch?v=tdvqkKdFTlw)
- [Gavin Wood: The Gray Paper Interview](https://www.youtube.com/watch?v=O3kRAVBTkfs&t=5s)
- [Polkadot Wiki: JAM Chain](https://wiki.polkadot.network/docs/learn-jam-chain)
- [Polkadot Wiki: Agile Coretime](https://wiki.polkadot.network/docs/learn-agile-coretime-index)

## Bullet Point List

### Struct Definitions

- Header & Block
- Tickets, Judgements, Preimages, Availability, Reports
- State: core authorization pool, most recent blocks, Safrole state, prior state of service accounts, 
  entropy accumulator & epochal randomness, validator keys and metadata (VKaMD) drawing next, current VKaMD,
  prior VKaMD, pending reports per core, recent block's timeslot, authorization queue, votes about ongoing disputs,
  privileged service indices

### PVM

- __What & Why: RISC-V VM (simple to transpile and to meter)__
  - PolkaVM implementation found [here](https://github.com/koute/polkavm) (unknown if ready or not)
- __Understanding in own/other words__
  - We assume that a generic PolkaVM with RISC-V ISA is implemented but needs to be wrapped/integrated properly for JAM,
    or in other terms, to be ported to the "JAM platform".
    The PVM will need the possibility to read from and write to JAM's storage or blockchain state.
    JAM will need the chance to invoke service executions on its cores by using the PVM.
    Under the assumption that JAM is a new type of blockchain, that blockchain acts like an operating system hosting the PVM.
    The OS decides which service to be executed next.
    Services will be executed using the PVM and need a couple of "stdlib-calls" like read, write, lookup, etc.
  - Herefore, we would have to implement some generic JAM-PVM functionalities:
    - Standard Program Initialization
    - Inner Host Calls
    - Single Step State Transition $\Psi_1(\textbf{c}, \textbf{j}, \iota, \zeta, \mu)$
  - Some generic PVM -> JAM "stdlib-calls" (see appendix B.6-B.8):
    - read, write, lookup, gas, info
    - empower, assign, designate, checkpoint, new, upgrade, transfer, quit, solicit, forget, invoke
    - historical.lookup, machine, peek, poke, invoke, expunge
  - And on top of that, several JAM -> PVM invocation entry points:
    - Is-Authorized $\Psi_I(\textbf{p}, c)$, see appendix B.2
    - Refine $\Psi_R(\delta, \textbf{y}, s, c, p, \textbf{x}, \alpha, \textbf{o})$, see appendix B.3
    - Accumulate $\Psi_A(\delta^\dagger, s, g, \textbf{o})$, see appendix B.4
    - On-Transfer $\Psi_T(\delta^\ddagger, s, \textbf{t})$, see appendix B.5

### Safrole

- __What & Why: Limits forks to a large extend but allows in some situations__
  - Will be combined with Grandpa to deal with forks
  - Sassafras as template with modifications:
    - TODO: Check for differences between the Polkadot-Sassafras and the FRAME-Sassafras
    - Seems not to be finished absolutely [but close to it](https://github.com/paritytech/polkadot-sdk/issues/41);

### QUIC & NTP

- __What & Why: 1000 nodes in communication; living in 6s-epochs; started at JAM Common Era__
- NTP impls available?
- Several QUIC implementations already available, see [docs.rs](https://docs.rs)
- libp2p only supports [bidirectional connections](https://docs.libp2p.io/concepts/transports/quic/) -> extend libp2p?

### Identified Follow Up Investigations

This section is kind of a brainstorm section which collects open topics where to focus next on.
Side nodes might be also written down here to its topic.

- Parachain Service for JAM
  - Part of deliverables?
- Authorization Agent
  - Coretime will be paid via an authorisation agent in a parachain system service
  - Refine functions live outside the on-chain; they are off-chain but in-core
  - How does the authorisation agent select nodes to execute Refine?
- Networking communication among nodes/validators
  - 3 validators per core, but how do validators choose others to communicate their messages to each other?
  - Validators shall send work packages to each other validator; after 2 of them have guaranteed the validity of a work package,
    it will be sent to the next block author
