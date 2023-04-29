Linkability is the Achilles' heel of MW.

Beam has several [improvements](https://github.com/BeamMW/beam/wiki/Transaction-graph-obfuscation) to the original MW to obfuscate the transaction graph. Now we're building a hybrid, of MW and Lelantus, which should be a huge step forward in this direction.

## Disclaimer

The [Lelantus protocol](https://lelantus.io/) is the work of Zcoin's cryptographer Aram Jivanyan as part of its research to improve its privacy protocol.

Our design and implementation are based on the publicly-available Lelantus scientific paper. All our code was developed from scratch based on this paper alone. 

# Our design

To fit our needs and utilize the full power of MW we made several modifications to the original protocol.

1. Instead of focusing on transaction types described in the paper (Mint, Spend, Joint-Split) we implement this in terms of _primitives_, which can be combined and used in various transaction types.
1. Minted/spent values are never revealed.
1. We use other technique to prove the transaction balance. We removed the balance proof from the original Lelantus protocol, hence it's now closer to the original Sigma protocol by Jens Groth.

MW blockchain consists of the following objects (primitives):
1. Inputs - references to existing UTXOs, that are being spent
1. Outputs
1. Transaction kernels

Our design, which we call Lelantus-MW, keeps this structure, and the Balance-to-zero principle also holds.
All those 3 object types, however, are modified to support Lelantus.

We use the following notation in the code:
* **G** - generator (nothing-up-my-sleeve EC point) multiplied by UTXO blinding factor
* **H** - generator multiplied by UTXO Value
* **J** - generator multiplied by UTXO 2<sup>nd</sup> blinding factor (i.e. double-blinded commitment).



## Outputs

Normal MW output consists of a Pedersen commitment (EC point) and the Bulletproof signature. After validation it's added to the UTXO set, and later can be referenced as inputs in consequent blocks/transactions.

The modified output, used in Lelantus, is called _Shielded_ output. Unlike normal output, shielded output is double-blinded, hence its bulletproof is (slightly) extended.

After validation it's added to the Shielded pool (rather than UTXO set).

## Kernels

As we said, shielded outputs are double-blinded. Hence, in order to keep the balance-to-zero principle, transaction kernels can optionally contain the 2<sup>nd</sup> blinding factor excess as well. So that transactions are allowed to have excess of both blinding factors, but not the value (obviously).

Such kernels are signed by <u>generalized Schnorr's signature</u> (rather than normal Schnorr's signature), to prove that their revealed commitment is indeed of the form `k*G + s*J`

<b>Note:</b> the generalized Schnorr's signature does not reveal which part of the kernel commitment is due to each of the blinding factors. I.e. the attacker can't split it into `k*G` and `s*J`. This is important, as the 2<sup>nd</sup> blinding factor (a.k.a. serial number) eventually gets revealed, there is still no way to identify its corresponding transaction kernel.

## Inputs

Normal inputs are just commitments (EC points) that correspond to previous outputs that must be in the current UTXO set.

Shielded inputs, in addition to the commitment, have the <u>Spend Proof</u>, which proves that:
* A valid shielded element is being-spent
* No double-spend
* The specified input commitment encodes the value equal to the one being spent (with different blinding factor though).

# Spend Proof

In the original Lelantus paper the proof idea is the following:
* Convert the revealed public Spend Key into serial number `s`
* Subtract (methodically) `s*J` from each commitment in the referenced anonymity set (a.k.a. cmList)
* Prove the knowledge of the opening of one of the elements in the form of `k*G + v*H`, i.e. without the serial number
* Additionally prove that the revealed being-extracted value corresponds to the value of that element
    * For this the original Sigma protocol was modified

We modified it into the following:
* Convert the revealed public Spend Key into serial number `s`
* Prove that the revealed commitment `C` is of the form `k*G + v*H`, i.e. does not conceal additional serial number
* Subtract (methodically) `s*J + C` from each commitment in the referenced anonymity set
* Prove the knowledge of the opening of one of the elements in the form of `k*G` only, i.e. without the serial number or additional value

In such a scheme there is no need to provide additional balance proof, since the value of extracted commitment should be the same as of the element being-spent. Because of this there is also no need to prove the value is non-negative (by bulletproof), as it was already proven when the element was added to the shielded pool.

So, the whole Spend proof, in addition to the commitment being-extracted, contains the following:
* Generalized Schnorr's proof that this commitment is of the form `k*G + v*H`
* Public spend key, and the whole spend proof is signed by the appropriate private key
* Standard 1-out-of-N Sigma protocol in terms of a single `G`-generator only.

## Compared to original Lelantus paper

What is similar:
* Shielded outputs are double-blinded, with the appropriate (extended) bulletproof signature
* To spend a shielded element the public Spend Key must be revealed, and the Spend Proof must be signed by the appropriate private key
* Spend proof is also based on the 1-out-of-N Sigma protocol (by Jens Groth)


What is different:
* Rather than specifying transactions, Lelantus-MW is formulated in terms of inputs and outputs (i.e. MW-style). Hence:
   * Just a single type of shielded input and output primitive is enough
   * Transactions are easily merged (as usual)
   * Despite better size and verification time, we don't implement multi-input spend proofs.
       * Limiting all the inputs to the same anonymity set seems to be practically restricting
       * This also potentially reveals their linkability
* Added/spent values are never revealed
* Separate balance proof is not required: it's an inherent part of each MW block/transaction.
* Spend proof is improved
   * Implemented in terms of classical (unmodified) Sigma protocol
   * Significantly smaller size
   * Better verification time
      * Significant in batch mode, when all the proofs refer to the same anonymity set

# Direct anonymous payments

In classical MW payments are interactive, and this is unavoidable: payment means creating an UTXO for someone else, however in order to create such an UTXO and its bulletproof the creator needs its blinding factor, hence it owns it, means it must be the payee.

We [designed](https://github.com/BeamMW/beam/wiki/One-side-payments) a mechanism to allow one-side payments after initial setup (currently fully supported in Node, but not in the wallet), however it's less anonymous, and the functionality is limited.

Lelantus doesn't have this limitation. It's possible to create a shielded output, such that the creator itself can't spend (thanks to the idea with Spend pubkey). What's remaining is the "coloring" scheme of the shielded output, i.e. how to make it distinguishable to the payee only, but not to others, including those that pay to the same payee.

## Standard bulletproof coloring scheme

(based on the coloring scheme used by secp256k1 library)

Our standard UTXO coloring scheme allows to embed an arbitrary 255-bits long description. In practice we use 24-byte description, which we call _Coin ID_.

During bulletproof construction various nonces are generated. Those nonces are generated in a deterministic way from the public (visible) UTXO commitment, and a secret _coloring seed_ (not necessarily the one used to generate the blinding factor).

In particular the following is performed
* α - generated nonce
* ρ - generated nonce
* x - challenge
* Revealed: `μ = α + ρ*x`

In order to embed meta-data into UTXO, the creator converts it into a scalar `β` (possible for every 255-bit data), and adds it to the originally-generated nonce `α`.

So, the revealed `μ` is calculated as: `μ = α + β + ρ*x`.

In order to recognize such an UTXO one needs the same secret _coloring seed_. It generates the same `α`, `ρ`, and then recovers `β` and the embedded parameters:

The recognition process goes as following:

* Generate `α`, `ρ` from the UTXO commitment and the _coloring seed_
* Calculate the challenge `x` (from the revealed bulletproof transcript)
* Calculate `β = μ - α - ρ*x`
* Decode the _Coin ID_, see if this makes sense (in particular it may have only 24 trailing non-zero bytes)
* Recreate the UTXO commitment from the master secret and the extracted _Coin ID_
* Check if the obtained commitment is correct

## Advanced coloring

So far the above scheme is suitable for coloring the shielded double-blind UTXO, but it's not anonymous: anyone with the same _coloring seed_ can identify such an UTXO

To overcome this limitation we use an additional step. We'll encode the embedded meta-data using the Diffie-Hellman encoding scheme.

The payee creates a private/public key pair, which we call _encoding key_. The _encoding pubkey_ is given to the payer, along with the _coloring seed_.

During the bulletproof construction, among other things, the creator reveals the `T1` commitment, which is used to hide the blinding factors. In case of double-blinded bulletproof it's calculated as:
* `T1 = n1 * G + n2 * J`
* `n1`, `n2` - nonces, generated deterministically from the UTXO commitment and the _coloring seed_

Now we'll add another nonce to this:
* `T1 = (n1 + n3) * G + n2 + J`
* `n3` - **random** nonce, i.e. can't be recovered from the _coloring seed_

So, using the _coloring seed_ it's not possible to recover `n3`, but it's possible to obtain `n3 * G`.

Finally, both payer and payee calculate the same secret:
* Payer:
   * `n3` * _encoding pubkey_
* Payee:
   * `n3 * G` * _encoding private key_

Finally the scheme goes as following:

Payer:
* Generate random nonce `n3`
* Calculate shared secret point `S = n3 * encoding pubkey`
* Convert X-coordinate of `S` to a scalar `γ`
   * If it's too large and can't be converted (highly unlikely) - retry with different nonce
* Add `γ` to the encoded meta-data `β`
* Add `n3` to `n1` for the rest of the protocol
* The rest is straight-forward

Payee:
* Generate the nonces `n1`, `n2`
* Calculate the "unmodified" `T1`
* Subtract it from the revealed `T1` to obtain `n3 * G`
* Calculate shared secret point `n3 * G * encoding private key`
* Convert X-coordinate of `S` to a scalar `γ`
   * If it's too large - skip the rest
* Recover encoded meta-data `β` (as usual)
* Subtract `γ` from `β`
* The rest is straight-forward

### What about shielded inputs?

They are identified by the revealed Spend pubkey. The payee should track all the Spend pubkeys for shielded outputs it detected, and realize the spending once it sees the shielded input from the same Spend pubkey.

### Payee address

So far the payer needs the _coloring seed_ and the _encoding pubkey_. But they don't have to be different: a payee can provide the _encoding pubkey_, and the _coloring seed_ may be generated from it in a deterministic way.
Moreover, since the _encoding pubkey_ is fully controlled by the payee - it can always make sure its `Y`-coordinate is odd/even (by just negating the private key), hence providing only X-coordinate (32 bytes) is enough.

### Multiple address?

Although in the above scheme a payer can not identify payments of others to the same payee, it may still be necessary to have multiple addresses. For instance, if payers share information, it may be necessary to conceal the fact that they pay to the same payee.

The above scheme is possible to extend to multiple addresses, they payee can generate arbitrary number of addresses. However there is no unified way for the payee to identify the payments: it needs to scan all the shielded outputs by all its generated addresses.
