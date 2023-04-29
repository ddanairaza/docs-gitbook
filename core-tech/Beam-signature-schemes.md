## Oracles
Oracles are objects used to generate challenges needed to construct/verify non-interactive proofs. They interpret exposed transcript data in a standard way, and produce the challenges in a deterministic way.

	oracle <-- exposed_data_1
	oracle --> challenge_1
	oracle --> challenge_2
	oracle <-- exposed_data_2
	oracle --> challenge_3
	oracle --> challenge_4
	// ...

## Schnorr's signature
We use Schnorr's signatures throughout the code. We prefer the form `[N, k]`, whereas `N` is the "public nonce", because it's batch-verifications compatible.
	
In addition to the standard Schnorr's signature we use the following variations:
1. Signatures within a specific context (transcript). Instead of signing a specific message the signature operates on the given Oracle, which means it signs a specific transcript.
2. Generalized Schnorr's signatures, i.e. argument of knowledge of the commitment opening in terms of several generators.
3. Schnorr's multi-signatures. For a number of signatures in terms of the same set of generators we use a scheme of the form `[N, k1, k2, ..., kn]`. That is, only a single `N`. For the verification n challenges are derived. The soundness of such a signature can be shown by the same "extractor" technique used for standard Schnorr's signature for each secret key in reverse order.
		
Worth to note: we also use the (3) in the context of a more complex transcript, where a complex proof proves, among other things, knowledge of various secret keys (such as Lelantus spend proof).

# Biased Sigma protocol.
prove the knowledge of opening of 1-out-of-N, after a _Bias_ is subtracted (methodically) from all the elements.

Inputs:
* Bias (EC point)
* Set of N elements (EC points).
	
The proof is similar to the standard Sigma protocol, as described by Jens Groth. The differences are the following:
1. It operates on Oracle. Means - it's a part of a specific transcript, and it's not possible to separate it or tamper with it.
2. For performance reasons the _Bias_ is not technically subtracted from the set. Instead its cumulative coefficient (multiplier) is calculated, so that the _Bias_ is added later to the equation.
3. Since because of (2) the set is not actually modified - the whole proof is very batch-friendly. So in a similar way the coefficients of the elements in the set are updated after each individual proof, but their multi-exponentiation is deferred.

### Padding

If there are not enough elements in the set we use zeroes (points at infinity). For original Sigma protocol that'd be insecure (opening of point at infinity is trivial), however for all our use-cases it's fine because of the Bias, since all our proofs are about validation	of the Bias, showing that it's a legit commitment of G (blinding factor) and something else. Using the Bias that consists of blinding factor only makes no sense for the attacker.

## Asset proof
Proves that a given blinded generator H' satisfies H' = k•G + H<sub>i</sub>, whereas H<sub>i</sub> is a legit generator defined as:
* i > 0: H<sub>i</sub> = CreateGenerator("B.Asset.Gen.V1 | i);
* i = 0: H<sub>i</sub> = H (standard H-generator used for Beams)
	
The proof is based on the biased Sigma proof described above. The prover chooses the window containing its generator by specifying the window first element. The window size is fixed in the consensus rules. Padding is not applicable, since H<sub>i</sub> is assumed to be infinite series.
	
* The Sigma protocol's proof is in terms of `G`-generator only
* Bias := H'
* Witness data: `i`, `k` (the blinding factor of the generator)

### Question
the H' currently is NOT exposed to the Oracle used in Biased Sigma protocol. Means - an attacker can theoretically craft a false Sigma proof, and then substitute an appropriate H' that would fit the needed equation and make the proof valid.

However I can't see how the attacker can use such an H' generator. Since it comes-out as a random EC point, it has no direct relation to any other generator, and can't be used to conceal negative values, serial number or etc.
	
Should we expose H' to the oracle? Is it "nice-to-have", or essential?

## Lelantus spend proof

Proves that a legit element is withdrawn from the shielded pool.

Inputs:
* `SpendPk` - spend public key, from which the serial number `s` is derived in a deterministic way.
* Optional: blinded Asset generator `H'` and _Asset Proof_, which proves its validity.
* `Commitment` - commitment to the element being-withdrawn.
* Shielded pool window - the list of commitments (EC points).

Proves the following:
* The spend is signed by the secret key, the pre-image of `SpendPk`.
* `Commitment` is a commitment of the form k•G + H'•v. The validity of `v` (i.e. rangeproof) is not necessary.
* Serial number `s` (derived from `SpendPk`) corresponds to an element in the specified window.
* `Commitment` commits to the same value and asset type (H<sub>i</sub>, v).

### Methodically

* _Asset Proof_ is verified if specified. Otherwise the asset generator is assumed `H` (used for Beams).
* Generalized Schnorr's signature proves `Commitment` is indeed of the form k•G + H'•v.
* Serial number `s` is derived from `SpendPk`
* Biased Sigma protocol is used for the rest:
   * The Bias is: `Commitment + s•J`
   * The proof is in terms of `G`-generator only.
   * Witness data is the blinding factor difference.

### Technically

Technically there are 2 Schnorr's proofs here: validity of `Commitment`, and knowledge of pre-image of `SpendPk`. They are compressed into a single generalized multi-signature, i.e. the prover knows the opening of both `Commitment` and `SpendPk` in terms of `G` and `H'`.

<u>Note:</u> Normally the `SpendPk` must be of the form k•G, and in our scheme we weaken this by allowing additional generator `H'`. But there seems to be no problem here if someone decides to used `SpendPk` that contains also the asset generator used for this element.

In addition the `Commitment` is used twice in the protocol: in the generalized Schnorr's signature, and as a part of the Bias for the Sigma protocol. So instead of using it twice we aggregate its coefficient, and is it once in the equation.

Finally the transcript is the following:

* oracle <-- Sigma parameters (`n`,`M`)
* oracle <-- `Commitment'
* oracle <-- `SpendPk'
* oracle <-- `N' (public nonce of the Schnorr's multi-signature)
* oracle --> Challenge for `Commitment`
* oracle --> Challenge for `SpendPk
* <-- Schnorr's multi-signature: `kG`, `kH`
* oracle <-- Sigma protocol part 1 (A, B, C, D, G-vector)
* oracle --> Challenge for Sigma protocol
* <-- Sigma protocol part 2 (a, c, r, f-vector)

