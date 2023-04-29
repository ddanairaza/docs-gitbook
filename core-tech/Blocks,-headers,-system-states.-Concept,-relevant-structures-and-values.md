First let's define terminology.

* System state - is a _valid_ state of the system, which is fully defined by all the existing data (more about this later)
* Block - is an information used to perform a _transition_ from one system state to another.

To avoid ambiguity we refrain from saying "block hash" or "block height", because, strictly speaking, those are properties of the System state. OTOH blocks are just transactions, which may be combined.

Each state has a header, and there are several structures related to it.

### SystemState::ID

Consists of:
* `Merkle::Hash m_Hash`
* `Height m_Height`

Used to denote an existing known state. (Hash would be enough, but we decided to include the height as well)

### SystemState::Full

A full header bound to the system state. Consists of:
* `Height m_Height`
   * Our convention: `m_Height = 0` - is for initial system state, no data and nothing is interpreted yet. The next state, which is achieved after interpreting the genesis block, has `m_Height = 1`.
* `Merkle::Hash m_Prev`
   * explicit reference to prev
* `Difficulty::Raw m_ChainWork`
   * Cumulative chainwork (sum of difficulties) <u>including</u> the difficulty of this state.
* `Merkle::Hash m_Definition`
   * The Hash of the full system definition in this state (more about this later)
* `Timestamp m_TimeStamp`
* `PoW m_PoW`
   * Proof-of-Work. An equihash solution.
   * Contains the `Difficulty` of this specific state.

The _System State Hash_ is calculated from all the header parameters, <u>including PoW</u>. Not to be confused with `m_Definition`, it's different.

Note that when many such headers should be presented for consecutive states - there are 3 redundant elements:
* `m_Prev` - obviously equals to the Hash of the previous state
* `m_Height` - just increased by 1
* `m_ChainWork` - can be calculated from previous header, after adding the difficulty of this one.

So for those cases there are appropriate structures defined: `SystemState::Sequence::Prefix` and `SystemState::Sequence::Element`. And the `SystemState::Full` just inherits them both.

### PoW

BEAM uses equihash mining algorithm. It includes the following:
* `Difficulty m_Difficulty`
* `m_Nonce` - arbitrary 64-bit values
* `m_Indices` - the solution, array of sorted indexes.

The input for the solver/verifier is constructed from all the Header fields, except the PoW solution itself. But it does include the `m_Nonce` and the `m_Difficulty`. So that the difficulty must be selected before the mining, and can't be adjusted a-posteriori, if the solution eventually could reach higher difficulty (reached a smaller target).

**Note**: The whole PoW parameters, including the solution, are accounted for in calculating the System State Hash. This is an intentional design decision, which ensures it's not possible to construct the chain of the system state headers without actually mining them, and then be able to mine only some specific states on-demand. This assumption is essential for _FlyClient_ protocol.

# System Definition Hash

One of the most important parameters. The complete system state is constructed from the following data:
* All the existing non-spent UTXOs, stored with the relevant parameters in an `UtxoTree`, which is a variant of the `RadixTree`.
* All the existing Kernels, stored in the `RadixTree`.
* MMR of all the inherited system states.

The System Definition Hash is defined as:

`System-Definition-Hash = Hash [ InheritedStates.Root | Hash (UTXOs.Root | Kernels.Root) ]`

After interpreting the appropriate block, the Full Node evaluates the actual System-Definition-Hash, and compares it with value in the State Header.

Note also that it's actually a root hash of the Merkle tree, whose elements are another Merkle trees. Using this property it's possible to generate and verify Merkle proofs for:
* Existing UTXO in the current system state
* Existing Kernel
* Inherited System State

All the Verifier needs is the proof, and the System Definition Hash. In addition to verifying the proof, the Verifier ensures the proof suffix is in accordance to the known part of the Merkle tree structure. Means:
* For UTXOs and Kernels the Verifier knows the hashing direction of the last 2 elements.
* For inherited System State the Verifier knows the whole Merkle path, hence all the hashing directions are deduced automatically.

# History compression, Macroblocks

BEAM fully supports the excellent MW feature of history compression with removal of the spent outputs. However unlike what was written in the original MW whitepaper (published by monsieur Tom Elvis Jedusor) - there is no need to provide Merkle proofs or other supplementary info.

The "compressed history" in BEAM is actually just a one huge block, which we call a _Macroblock_, which contains a single huge transaction, which is interpreted (almost) as a regular block/transaction. And this is where the System Definition Hash is of critical importance.

No matter how big the compressed history is, the only thing that matters is the final system state, and it's verified according to the expected System Definition Hash. This is how the authenticity of the compressed history is verified.

# How Macroblocks are generated and used in BEAM

There is an implementation of the merging two consequent (macro)blocks. It's a sort of a "merge sort" of all the transaction elements (which must be sorted both in input and the resulting blocks), with removal of spent elements. Using this implementation an arbitrary number of blocks may be merged by recursive halving.

Our initial design was to generate and keep a _cascade_ of macroblocks. Means, the Node each time merges several blocks at the tail, and then they are iteratively merged to the last merged macroblock, only if they are of the same complexity. This means at any moment we have a list of macroblocks of decreasing size, overall O(log(Height)) such blocks.

Then, to import a compressed history, the (other) Node downloads the most recent list of the consecutive macroblocks and interprets them according to their order.

But later we abandoned this idea, mostly because of the hassles for the client to download a list of the macroblocks, especially given the fact they are constantly created and deleted by the generating Node.

So currently the Node just creates a single macroblock once in a while (1 day by default), whereas the cascade-merge is used only internally by the Node to generate each time the new macroblock incrementally.
Worth to note that during the macroblock generation, which can be a time-consuming operation, the Node is not paralyzed, and works as usual.
