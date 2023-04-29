MW-CLA stands for Mimblewimble Confidential Lelantus Assets

This page describes our most recent system design, which will be activated on Fork2. It includes CA (confidential assets), Lelantus-MW, as well as will enable adding more features in the future.

Those are the key features:
* CA support
    * each UTXO may optionally have a blinded asset tag (similar to the Elements design by A.Poelstra).
    * There's a proof of validity of the asset tag. It's based on the 1-out-of-many Sigma protocol (by Jens Groth).
* Shielded pool (a.k.a. Lelantus-MW)
    * CA support for shielded operations too
    * Support for one-side payments and direct anonymous payments
* The system design is heterogeneous in nature:
    * All kernels carry an excess blinding factors, and (optionally) extra validation rules (height lock, relative lock, etc.)
    * Some kernels may control subsystems:
        * Asset control (creation, emission).
        * Shielded operations (mint, spend).
    * Those kernels, in addition to contributing to the balance, will have side effects as well.

# Confidential assets support
<u>Note</u>: this design is very different from the older preliminary design.

We identify each asset by `AssetID`, which is a 32-bit integer. For each asset there's an appropriate NUMS generator, which is generated deterministically from the `AssetID` (via hashing). The `AssetID == 0` is reserved for default asset (Beam).

## UTXO encoding

In MW it's possible to encode UTXOs of different kinds (asset types) by using different NUMS (nothing-up-my-sleeve) generators. The UTXO that encodes an asset consists of the following:
* Blinded generator: **H<sup>*</sup>** = **H<sub>i</sub>** + k<sub>A</sub>•**G**
* Asset surjection proof (proves that the provided generator is indeed one of the list with arbitrary blinding factor added)
* Pedersen commitment: **C** = k•**G** + v•**H<sup>*</sup>**
* Rangeproof (bulletproof) in terms of this blinded generator

### Asset surjection proof

It's based on Sigma protocol. The prover specifies a _range_ of `AssetID` values, and proves that the specified generator is one of those with arbitrary blinding factor added.

The verifier generates the list of the asset generators for this range, and subtracts (methodically) the provided blinded generator from each element in the list. Then the prover proves by the Sigma protocol that it knows the opening of one of the elements in terms of `G` generator (used for blinding factor).

## Asset control

Any user is allowed to create its asset types. Unlike Beams, which are created automatically with each new block generated, assets are emitted and burned explicitly by their owners. The life cycle of each asset goes as following:

### Asset creation.

The user sends a transaction (kernel) that asks to create its asset type. It provides the following:
* _Owner key_ - an arbitrary public key that will be associated with this asset. All further asset actions must be signed by the appropriate private key.
* _Metadata_ - arbitrary data (buffer), visible to all other users. Once asset is created this metadata can't be modified.

In addition to this, the user _Locks_ a considerable amount of Beams, i.e. this transaction implicitly consumes this amount. If successful, the system allocates the lowest available (unused) `AssetID`, and associates it with this asset. 

### Asset emission/burn.

The user sends a special asset emission kernel (emission amount may be either positive or negative). In addition to signing the kernel excess blinding factor (as usual) - it also must be signed by the appropriate private key of the asset owner.

This transaction implicitly creates/consumes some amount of this asset, which should be compensated by other transaction elements (inputs and outputs).

### Asset destroying.

To destroy an asset the user sends an appropriate asset destruction kernel (signed by the appropriate owner key). After the asset is destroyed - the `AssetID` is no longer associated with the owner, and the user gets the locked Beams back.

Note that asset can be destroyed only if:
* It is completely burned.
* Minimum lock period elapsed after it was completely burned, and not emitted since then.

This minimum lock period is needed to prevent manipulations (tampering) by the asset owner. For example, a user may want to receive a specific asset, then it must be sure that this `AssetID` will be associated with exactly this asset info (Metadata and owner key) before its transaction expires, i.e. the asset owner won't be able to destroy and re-create the asset in the meanwhile.

## Asset state.

The system state contains a commitment to the most recent assets state. It's an MMR root of all the currently active assets, with their info, which includes:
* Static info: _Metadata_ and owner public key
* Current emission value
* _Lockheight_ - the most recent height of the asset burned/not-burned transition. Based on this users can:
    * Asset owner: knows if/when it can destroy the asset, and get the locked funds back
    * Other users: estimate the minimum height range when this asset can be used safely. i.e. can it disappear due to reorgs in the past, or tampered in the future.

<u>Note</u>: The ever need to lock the funds for asset creation is needed to prevent system spamming. Not only excess of assets will make Nodes heavier, it'll also make the asset surjection proof less effective (since its anonymity set is limited).

However despite the need to lock considerable funds, this design should be ok for users that just want to experiment with assets, since they are supposed to get the funds back once they finish.
Organizations that sell their asset to users - not obvious if they will ever be able to burn their asset back (for this they first need to own all their asset). But this seems as a justified risk.

## Note on transaction repeatability

One of the problems specific to UTXO-based systems in general and MW particularly is repeatability. If an attacker controls all the inputs of a specific transaction where it pays someone (which is usually the case) - it can repeat this transaction later regardless to the will of the other user. Moreover, if that user later spends only the inputs received from the attacker - then the attacker can repeat those transactions too, and so on. Everything can be 'replayed' up to some depth, where more inputs are needed, that were not originally received from the attacker.

If only beams are traded, then there's no big problem. Because of those 'replays' users can only get paid, not loose their funds. But replaying asset-controlling transaction **IS** a problem. Because assets are essentially created from 'thin air', by replaying some transactions the attacker may cause extra asset emission (which is already a big problem), and may even be able to get some amount of this asset.

To mitigate this threat, starting from Fork2 duplicating kernels will be forbidden. Technically this is achieved by the following:
* There will be a new consensus parameter, `MaxKernelLifespan`, probably equivalent to ~ 1 month.
* Starting from Fork2, kernels with `HeightLock.Min` (minimum height) lower than Fork2 will be rejected.
* Each kernel, in addition to the optional `HeightLock.Max` (maximum height) will have implicit max height lock as `HeightLock.Min + MaxKernelLifespan`. This (and the previous restriction) will make repeating old kernels impossible.
* Each node will have to keep track of all the recent kernels, down to current height minus `MaxKernelLifespan`. Kernels below this height may be forgotten (for the sake of blockchain verification).
* Side effect: Kernels with relative height lock (already available on the mainnet) will not be able to reference a kernel older than `MaxKernelLifespan`. But this is ok, practically relative locks are needed for much shorter duration.

By such we will make kernel replaying illegal, whereas nodes will have to keep track only of the most recent kernels.

# Shielded pool (a.k.a. Lelantus-MW)

**Disclaimer:** The [Lelantus Protocol](https://lelantus.io/) is the work of Zcoin's cryptographer Aram Jivanyan as part of its research to improve its privacy protocol. Our design and implementation are based on the publicly-available Lelantus scientific paper. All our code was developed from scratch based on this paper alone.

In order to solve the MW linkability problem, users will be able to recycle their funds via shielded pool. Our design is different from the original Lelantus protocol in the following ways:
* Transaction values are never revealed
* Instead of transactions, it's formulated in terms of mint/spend _primitives_, and the final transactions are composed of MW and shielded parts in any combinations, keeping the balance-to-zero principle (MW-style).
* CA are naturally supported

Technically in addition to standard transaction elements, the following are supported:

* Shielded output - transfers some amount from MW into shielded pool
* Shielded input - withdraws some amount from the shielded pool back into MW

Both elements are encoded as special transaction kernels.



In addition to the standard MW blinding factor generator **G**, there's an additional generator **J** for the secondary blinding factor, a.k.a. _serial number_.

## Shielded output

Consists of the following:
* Blinded serial number commitment: **C<sub>s</sub>** = k<sub>s</sub>•**G** + s•**J**
* Generalized Schnorr's signature that proves the above commitment is indeed of this form
* Optionally asset info: the blinded asset generator + asset surjection proof.
* UTXO commitment **C<sub>MW</sub>** = k<sub>MW</sub>•**G** + v•**H**
* Rangeproof

In order to verify the overall transaction balance - only the UTXO commitment **C<sub>MW</sub>** (without the serial number) is accounted for. After verification, instead of going to the UTXO set, the following double-blinded commitment goes into the shielded pool:
* **C** = **C<sub>s</sub>** + **C<sub>MW</sub>** = s•**J** + (k<sub>s</sub> + k<sub>MW</sub>)•**G** + v•**H**

The shielded outputs in the pool form a sequence of commitments (EC points).

Note: the serial number *s* is derived from another public key *SpendKey*, which will need to be revealed during spending. In addition the prover will need to prove the knowledge of the appropriate private key.

In addition, the **C<sub>s</sub>** commitment must be unique. This is to prevent accidental misuse, which will make subsequent element withdrawal impossible.

## Shielded input

Consists of the following:
* Range within the shielded pool, that contains the being-spent element.
* *SpendKey* is revealed, and the whole shielded input is signed by the appropriate private key
* Optionally asset info: the blinded asset generator + asset surjection proof.
* Output commitment **C<sub>out</sub>** = k<sub>out</sub>•**G** + v•**H**
    * It should commit to the same value, but the blinding factor k<sub>out</sub> is different from that used in shielded output.
* Generalized Schnorr's signature, that proves the **C<sub>out</sub>** is indeed of this form.
* Sigma proof for the rest

The *SpendKey* must be unique, this way double-spend is prevented.

During the verification, the verifier computes the _serial number_ *s* from the *SpendKey*. Then the following is calculated:
* **C** = **C<sub>out</sub>** + s•**J**
This EC point is subtracted (methodically) from all the elements in the referenced range of the shielded pool. If everything is correct, then the element being-spent turns into:
* **C** = (k<sub>s</sub> + k<sub>MW</sub> - k<sub>out</sub>)•**G**

Note that both asset and serial number generators **H** and **J** are eliminated. The prover then proves knowledge of opening of one of the elements in the range in terms of **G**-generator only.

## One-side payments, and direct anonymous payments.

In addition to solving the linkability problem, shielded pool allows one-side payments (normally in MW transactions are built mutually). This is due to the fact that _serial number_ is derived from an arbitrary public key *SpendKey*, which, after initial setup, may be calculated by the sender alone, without the knowledge of the appropriate private key (and, hence, the ability to spend it).

This already provides the one-side payments ability. However it's not completely anonymous: since the sender knows the *SpendKey* - it can see when the receiver spends it.
But this can be solved too, due to the fact that the shielded output consists of 2 parts: the **C<sub>s</sub>** and **C<sub>MW</sub>**. During the initial setup the receiver generates and sends arbitrary number of different **C<sub>s</sub>** elements (with their Schnorr's signatures). The sender will use them as-is in the shielded output, without the knowledge of the *serial number*.

We incorporated a scheme by which the receiver detects all its shielded outputs by scanning the blockchain (i.e. no auxiliary channel is needed to notify the receiver). For **C<sub>s</sub>** all the owner info is embedded within the Schnorr's signature (which has a degree of freedom). For the **C<sub>MW</sub>** all the needed info is recovered from the bulletproof.

At the end the following information is recovered:
* All the relevant parameters: blinding factor, SpendKey, value, `AssetID`
* Is it visible to the sender, i.e. was the **C<sub>s</sub>** created by the sender or the receiver in advance.
* Sender ID (a public key belonging to the sender)
* Arbitrary 32-byte message

Note also that this info can be obtained by the so-called *Owner key*, but still in order to spend it - the master key is required. This allows to use the owner key in owned nodes to detect owned TXOs and shielded elements, without the risk of loosing the funds if the node is compromised.

## implications and constraints

The Lelantus is a great technology, but it comes at a price.

* Scalability (size)
    * Obviously no cut-through for the shielded inputs/outputs
        * Shielded output ~800 bytes
        * Shielded input ~1.6KB, depends on the anonymity set size
        * If asset type is blinded: 2 more asset proofs (for output and input), another ~2K
    * But cut-through is still applied on the MW part.
* Verification time
    * ~1sec for 64K elements (very big)
    * easily parallelized
    * only 10ms for each additional proof for the same anonymity set (batch verification)
    * During initial sync many blocks can be batch-verified at once as well.

So, in order to build a sane system, which enjoys the benefits of MW, but helps break the linkability, we design it this way:
* Most of transactions should remain in MW
* Max number of shielded inputs/outputs in a block is limited. Users will have to compete for them (fee market)
* The spend window (anonymity set size) is limited, and dramatically decreased if the element being-spent is not one of the most recent.

The maximum spend window (anonymity set size) will probably be ~50K - 100K (not decided yet). The maximum number of shielded elements in a block will be tuned such that this window will be created within at least several days.

Another important restriction: users will be able to spend their shielded element with the maximum spend window only if it references the most recent elements. It won't be possible to specify a large spend window, that covers a range older than twice this window size.

In simple words, users will have a time window to spend their element "nicely". If they miss their opportunity - they'll have to spend it in a dramatically smaller spend window (~1K elements), but then they will be able to recycle it through shielded pool again.

By such we expect to keep <u>good scalability and performance</u>:
* Not too many elements that can't be cut-through
* Reasonable verification times: shielded inputs will have large overlap.

But importantly those restrictions will also lead to <u>better privacy</u>. Here's why.

## Privacy

To understand which privacy is achieved while hiding in a crowd, let's first define *absolute* and *relative* anonymity sets.
* The *absolute* anonymity set size is the net size of the set chosen by the user.
* The *relative* anonymity set size is the ratio of the chosen *absolute* set size, to the weighted overall set, where the user could potentially hide, with appropriate probabilities.

Speaking simply, the *relative* set size is a probability of a user to choose a specific absolute set.

To achieve high privacy <u>both</u> the absolute and the relative sets should be maximized.

* Obviously if the absolute set size is small, then the user is already suspected.
* If the relative set size is small then the user can be deanonymized by a number of recurring transactions, even if the absolute anonymity set is big! [A good explanation by Ian Miers is here.](https://www.zfnd.org/blog/blockchain-privacy/)

Because the anonymity set size in Lelantus is finite, we need a compromise.
* If too few users use it, then every user is already is a suspect.
* If too many users use it, then the window is filled within shorter time period, which means smaller relative set (smaller probability of an unrelated user to fall into the same set).

The systems with unlimited anonymity set size (like Zcash) have an advantage here. However, speaking practically, the difference may be not that big. Although theoretically users can spend any element, practically they probably spend their recent outputs anyway (because of the usage nature). So the information leaked in Lelantus is considerable, but could be assumed by the attacker with significant probability anyway.

Probably real-world usage data is needed to estimate the practical privacy of the system