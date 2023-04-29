BEAM uses various kinds of Merkle trees and proofs.

There are 2 kinds of proofs used:
* Standard proofs. Consists of an array of hashes, with the flag that specifies the hashing direction (left/right)
* Hard proofs. Consists only of hashes. The hashing direction (as long as the proof length) is deduced by the Verifier automatically.

Hard proofs are used where the client is aware of the supposed Merkle tree structure and the position of the needed element. Those proofs are not only a little smaller, but also more robust - they won't allow the Attacker to include different versions of the same element.

In addition there are combinations: a proof which begins as a standard proof, whereas for the suffix the client may deduce the hashing direction. More about this later.

# MMR

Stands for _Merkle Mountain Range_, which is a fancy term to describe a (potentially) incomplete Merkle tree.

The underlying objects that are represented by the leaf nodes are converted to hashes according to a specific scheme applicable to the specific object kind (Means - Merkle trees in BEAM never contain _raw objects_, to prevent possible ambiguity attacks).

The hashing used is SHA-256. Non-leaf node hashes are calculated according to
> `Hash ( Left-Child | Right->Child )`

The tree fill order is left-to-right, whereas at each height two adjacent nodes form a parent node. This forms a sequence of complete trees of decreasing height (a.k.a. Mountain Range). Then adjacent trees are grouped from right to left, and form a mutual parent node, whose children are the roots of those trees. So it's like a regular Merkle tree, except the fact that the root node of the right child is "promoted" to the height of the left tree.

**Note**: in our implementation this promotion doesn't involve any actions, such as hashing with itself (which is used AFAIK in some implementations). This means that the proof length may vary for different elements.

## Implementation details

The core MMR implementation is in `Mmr` abstract class. Contains the following virtual functions:
> 		virtual void LoadElement(Hash&, const Position&) const = 0;
> 		virtual void SaveElement(const Hash&, const Position&) = 0;
Whereas `Position` is a logical position of an element in the tree (Height and X-coordinate).

Based on the underlying implementation in a derived class, which is supposed to load/save an arbitrary tree element, it supports the expected functionality: appending elements, getting the root hash, getting a proof for a specific element.

There are various implementation base on `Mmr`, which differ in how they actually store the hashes.

### DMMR - Distributed MMR


A more sophisticated variant is called `DistributedMmr`. It's assumed that the tree data is not stored in one place, but distributed over multiple sites. So that whenever a new element is added to the tree - this element gets a "reference" to the existing tree data, and all the new data that should be created (new non-leaf node hashes) is stored in the context of this element only, without the modification of older ones.

This is very useful for branching, whereas one needs to keep track of different variants of what's added to the MMR. 

[More information about their structure is here.](https://github.com/beam-mw/beam/wiki/DMMR-internal-layout)


#### Use case

Every system state should have an MMR of all the inherited states to support
 * proof of state inclusion in the current state
 * Chainwork proof (variation of _FlyClient_).

Every state references the DMMR of the inherited state and adds the extra data to reflect its addition to the tree.

This allows to use the full MMR functionality for every state, no matter if it's the branch tip or not, as well as if it's a part of the current consensus branch or not.

## Multiproof

We use an efficient encoding scheme in a situation where the Prover is supposed to prove multiple elements, whereas the tree structure is known to the Verifier (i.e. the client is able to deduce the Merkle path for any element). In this scenario, during the proof interpretation, both the Prover and the Verifier process hashes up to the point where the Merkle path collides with a path of an element which was included already.

This both saves the proof size and the verification complexity.

# Radix Tree

(a.k.a. Patricia tree, Merklish (?) tree)

BEAM makes use of special Radix trees, with the following properties:
* It's a variant of a Merkle tree. Means - leaf elements represented by hashes, non-leaf nodes have exactly 2 children, and their hash is evaluated accordingly.
* It's also a binary search tree, which means:
  * logarithmic search, modification
  * Option to search by either exact or partial key, find elements within key ranges, and etc.
* The internal tree structure is fully defined by the set of contained elements. Regardless to the _history_ (i.e. in which order they were inserted/removed).

The search key is an opaque bits sequence, whereas the length of this sequence must be equal among all the elements contained in the Radix tree. It may, however, be different for different tree instances, which are supposed to contain objects of different kinds (UTXOs, kernels, etc.).

Our Radix trees have the following properties:
* 1bit - radix. 
   * Means - the key bits are evaluated one-by-one, and each bit is potentially a junction.
   * Non-leaf nodes always have exactly 2 children
* Lazy split
   * Unlike classical radix trees, our trees have non-leaf nodes only where splitting is needed.
   * In particular if there's only 1 element - there is indeed just a single leaf node.
* Lazy evaluate
   * Merkle hashes are not immediately recalculated when the tree is modified, instead they're just marked _dirty_.
   * They are re-evaluated (recursively) only when needed and only if _dirty_.
* Intrusive-container-like semantics
   * To insert an element into the tree it's first allocated with the key (as a single allocation block).
   * The key of the allocated leaf node is used by the non-leaf nodes (they have a pointer to the same key).
* No pointer to parent
   * To save space pointers to parent are omitted.
   * To allow this - during the operation we keep a _Cursor_, which contains the pointers to all the passed nodes to the point.

Speaking technically, non-leaf nodes are allocated/destroyed on-demand, they have the pointer to the key (of one of the leaf nodes contained beneath), and the bit count remaining to the next node. Either the next node is the leaf or not - is determined by the total path length - is it equal to the assumed key length (which is known and supposed to be equal for all the contained elements).

## Merkle disproof

Surprisingly using those radix trees it's possible  to prove (with some assumptions) that there is no queried element.

If there is no queried element - the Prover can present its two closest siblings (or one if it's the maximum/minimum) with their Merkle proofs, which the Verifier can verify indeed belong to adjacent elements.

The Verifier needs, however, to assume that the tree was built according to the rules. In particular when querying for multiple elements, both existing and non-existing, the Verifier should verify that indeed all the presented proofs are in agreement with the assumed element order.
