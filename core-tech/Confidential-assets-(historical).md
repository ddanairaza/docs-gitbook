Consult the [MW CLA](https://github.com/BeamMW/beam/wiki/MW-CLA) section for Confidential Assets implementation details

Consult the [Confidential assets (BETA)
](https://github.com/BeamMW/beam/wiki/Confidential-assets-(BETA)) section for the Confidential Assets tutorial

**THE TEXT BELOW IS KEPT FOR HISTORICAL REASONS**

***

MW can be extended to allows encoding multiple types of assets to be traded on the same blockchain. And it will only need a slight modifications to actually allow this.
There are two types of assets that can be implemented: _predefined_ and _custom_. Each type has its advantages and limitations.

## Basic idea

The UTXO is an EC point, which is a linear combination of two nums (nothing-up-my-sleeve) generators: `G` and `H`, whereas `G` is multiplied by the secret key (blinding factor), and the `H` - by the value.

To allow multiple types of assets it's sufficient to use different generators (one per asset type) instead of the single `H`. There may be different schemes to represent such UTXOs, but in any of them the following should be taken into consideration:

* Those <u>**must**</u> be nums-generators, and the verifier should be able to verify this.
* The verifier should be able to verify the rangeproof. Means - the bulletproof should be adjusted accordingly.
* There should be a brief scheme for the emission of the assets.

## Predefined asset types

This idea belongs to Andrew Poelstra. Described [here](https://blockstream.com/2017/04/03/blockstream-releases-elements-confidential-assets.html).

Each UTXO should carry a _tag_, which is an EC point, which defines the asset type. The great advantage of this scheme is that all the tags are **blinded**. Means - anyone can verify that this tag corresponds to one of the defined asset types, but not to which of them exactly. This is achieved by using Andrew Poelstra's <u>Asset Surjection Proof</u>, which has a modest size compared to the bulletproof for a reasonably-small set of asset types.

The set of the asset types, as well as their emission schedule, must be defined for the blockchain. Any change to this will require a fork.

### Another variant

Another possible way to implement this is to encode all the asset types within a single UTXO. That is, each UTXO is presumably a linear combination of all the generators at once. In this design _tags_ are not needed.

The drawback here is the increased complexity and size of the bulletproofs, which seem to be dramatic. So that the idea with _tags_, whereas an UTXO encodes only a single asset type - seems to be better.

## Custom asset types

In addition there is a possibility to allow _custom_ assets, which any user can emit and trade. As in the previous scheme, such UTXOs should carry a _tag_, which corresponds to the asset type. But this time those tags can't be blinded perfectly. All the user can do is present a set of tags, and prove that the used tag is one of them.

So that custom tags should either be visible, or partially obfuscated. The encoded amount, naturally, is fully concealed.

Now, since there are no predefined generators used for custom asset types, there should be a way for the verified to make sure each such a generator is actually a nums-generator. This is addressed by the following scheme.

### Asset control

To create a custom asset type the user generates a public/private key pair. The public key serves as an _Asset ID_, and the generator used for this asset type is derived from the ID via hashing, so that it may be considered as a sound nums-generator.

The user controls the emission and collection of the asset. The user can _convert_ some amount of the master asset type (i.e. BEAM) into his/her type by a special instruction, which is signed by the corresponding **private** key. For convenience it can be embedded into the transaction kernel.

**Note:** The conversion is only needed to prevent bloat. The user effectively _buys_ his/her coins, but they are refundable. Only the user is be able to trade the unspent assets back to collect the refund. If the bloat is not an issue - it's possible to allow the emission for free.

# Final design

The BEAM should support both the predefined and custom assets.

There should be several predefined asset types, which are automatically emitted by every new block. Their difference is the <u>emission schedule</u>. There should be one with constant emission, one with capped emission. Possibly another one with declining but non-converging (e.i. not capped) emission.

This is equivalent to having several coin types suitable for instant payment, store of value, and etc.

Custom asset tags are not emitted automatically, they're explicitly traded for one of the predefined types.

The modifications needed to support all this is considerable, but straightforward. The following should be modified:
* Tags should be added to UTXOs
* For predefined types: Assert Surjection proofs should be added to UTXOs
* For custom types:
   * Assert Surjection proofs may optionally be used, in addition to a list of the possible asset types.
   * Another possibility - they should not be blinded, and contain the _Asset ID_ explicitly.
* Transaction/Block verification:
   * Bulletproof code should be modified to support custom generators (probably slight performance degradation)
   * Custom type conversion instruction should be supported, but this is straightforward (only alters the summation to zero criteria)
   * Block emission should be handled w.r.t. predefined types and their emission schedule.


