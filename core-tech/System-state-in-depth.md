So the system state is comprised from the existing UTXOs, Kernels, and inherited system states, each is organized in an appropriate data structure.

# UTXOs

The Node must track all the existing unspent UTXOs in the system, and the maturity of each one (i.e. the state height from which it can be spent). In addition we support duplicate UTXOs (there's a use-case for this).

UTXOs are stored in an `UtxoTree`, which is a variant of our `RadixTree`. Each UTXO is represented by an `UtxoLeaf`, which is a leaf node with a key and additional values.

The key is a 321-bit long bits sequence, consisting of (the order matters):
1. `ECC::Point m_Commitment` 257 bits
1. `Height m_Maturity` 64 bits

The value that is stored for each such a node is:
 * `Input::Count m_Count` 32 bits

So that there is a unique `UtxoLeaf` for each Commitment + Maturity combination. However if several UTXOs happen to have both those parameters identical - the information is stored within a single `UtxoLeaf` (with the `m_Count` > 1). This is an intentional design decision. To validate transactions it's essential to know the Commitment and the Maturity of the all the UTXOs, whereas other parameters are not important, and such UTXOs are considered identical.

When an input UTXO is referenced by the Commitment - the Node always looks for the UTXO with the specified Commitment, and the lowest Maturity, which also must not be higher than the current State Height (recall that `RadixTree` supports search by partially-specified key). So there is no ambiguity with the selection of the input UTXOs.

### UTXO Hash, Proof

When the `UtxoTree` hash is evaluated, the `UtxoLeaf` is hashed by the following formula:

`UtxoLeaf.Hash = Hash ( UtxoLeaf.Key | UtxoLeaf.m_Count )` (the 321-bit key is zero-padded to the byte boundary)

The client requests a Merkle proof for an UTXO - the following is specified:
* Commitment
* `MaturityMin` - Optionally the minimum Maturity to query (0 by default)

The Node response includes an <u>array</u> of the following:
* Maturity
* Count
* Merkle Proof

After receiving the result the client evaluates the `UtxoLeaf.Hash` according to the same formula, substituting the asserted Maturity and Count, and then verifies the Merkle proof.

Empty array means such an UTXO doesn't exist. The array size is limited to 20 elements, so that if there are more combinations - the client should repeat the query, settings a higher `MaturityMin`.

Sounds somewhat complicated, but this is the price of the versatility, supporting duplicates. For regular UTXOs, however, where the duplicates are not possible (unless with a negligible probability), the client won't deal with all this. There will either be a single proof, or no proof.

# Kernels

Same idea as with UTXOs, but simpler, because kernels have no Maturity, and duplicates are not supported. So - it's just hashes. The Kernel ID is evaluated, and this is the hash of the leaf node.

For the Kernel proof the client just sends its ID, and gets a single Proof iff the kernel indeed exists in the System state.

### Why Kernel proofs are important?

Unlike UTXOs, Kernels don't contain funds, and can't be used in the future transactions, so why would anyone need a proof for it?

The main reason is that Kernel proof can be used to verify that UTXO is spent. The client may ask for an UTXO proof and get an empty result, but this is not a proof! It's impossible to prove that an object does not exist in the Merkle tree.

So, Kernel proof is the best verification of the transaction, no matter if UTXOs are received or spent.

# Inherited states

Implemented in terms of the **D-MMR** (Distributed MMR). The leaf nodes are the System State Hashes (not to be confused with the System Definition Hashes). Apart from the somewhat sophisticated D-MMR, the rest is straighforward (generating and verifying proofs).

They are needed in 2 scenarios:
* Simple scenario: a client was offline for a while, then comes back, and wants to ensure the last State that it observed is still a part of the current consensus branch.
* More complex: Client requests a _ChainWork Proof_, which is a (relatively) compact proof for all the state headers, without fully downloading them all.

**Note**: In contrast to UTXOs and Kernels, the client knows the structure of the Inherited States tree, hence it receives the proof in a form of a _Hard_ proof, i.e. only the hashes, whereas the hashing direction is deduced automatically.
