In the most common cases the contract method calls `Env::AddSig()` for each needed public key, and the app shader includes the key material for each key within the `SigRequest` structure, when it calls `Env::GenerateKernel()`.

This is a preferred method to design a contract and its application, as it has the following advantages:

* Performance: native signature is both compact and fast.
* Security: apps have no access to secret keys, they can only derive public keys.
* Simplicity.

However there are more complex situations, where either several users are required to authorize a method call, or a custom signature scheme is required (such as m-out-of-n signature).

Those may include:

1. Explicit multisignatures. Several keys are verified (mutliple calls to `Env::AddSig()`), belonging to different users.
2. Seemless multisignatures. There is a single key, but it is a sum (or any linear combination) of several keys, belonging to different users.
3. Non-standard signature is needed (ring signatures and etc.).
4. Any combination of the above

On the contract side those are supported. In addition to `Env::AddSig()` (which can be invoked arbitrary number of times) the contract has access to **Secp** primitives, using which it can implement custom cryptographic scheme verification.

For apps, however, the default functionality is not enough. Creating a multisigned transaction that is supposed to invoke such a contract assumes that different wallets should cooperate and communicate on behalf of the apps.

So we extended the BVM functionality to support all the variety of possible signature schemes. Though apps still don't get direct access to secret keys with the added functions, the risk is somewhat greater (more about this later), hence to use those extra functions the app needs an <u>elevated privilege level</u> (OFF by default).

Technically those functions are:
* Nonce Slots management functions
* getting an image of a key or a slot, with either standard (**G**) or any custom generator
* getting a blinded key
* Communication support (via SBBS system)
* `Env::GenerateKernelAdvanced()` - with more specific signature control

## Nonce slots and blinded keys

(the idea of nonce slots is the same as in our HW wallet design)

Generally our supported signatures follow this ritual: first a unique nonce is generated and its image is exposed (in some manner). Then, during the signing, a pseudo-random challenge is derived, and the prover should reveal a linear combination of the secret key and the nonce, a.k.a. blinded key.

This scheme is applicable to Schnorr's signature, Ring signature, Groth's 1-out-of-many, and probably many others.
It's **critically** important to make sure the nonce is unique, i.e. not reused for the same secret key in different signatures.

So we decided to add appropriate support functions for the apps: generate unique nonces, get their images, and get blinded keys:
* Generate unique nonce.
  * BVM generates a nonce using system random and state mutation.
  * optionally app can specify extra seed data to strengthen the nonce (i.e. less dependency on system random).
* Get nonce image (arbitrary number of times, with arbitrary generators).
* Get blinded key.
  * The app supplies the needed key material (a.k.a. `KeyID`, `SigRequest`), arbitrary challenge (scalar) and the nonce slot number.
  * Once the nonce is used - it's immediately wiped.

By such apps are given opportunity to build complex signature schemes, without either direct access to the secret keys, or the feasible opportunity to extract them.

# Building a custom signature scheme

As we mentioned, an app can create a custom signature (such as Ring signature), include it in the contract method invocation arguments, then the contract shader may verify it by implementing the needed verification scheme (in terms of **Secp** primitives).

Apps can generate such signatures by the provided functionality: obtaining nonce images, and getting blinded keys for the needed challenges.

### Risk of signature _hijacking_

When using native signature (`SigRequest`/`AddSig()`), it's included in the kernel multi-signature, which also includes its blinding factor. Other users can't take it as-is and use in their transactions (it's not feasible to build a transaction with unknown blinding factor).

In contrast, the custom signature is not automatically bound to the kernel blinding factor. Hence any other user that monitors transactions can build its own transaction that invokes the same method with exactly the same arguments.

To mitigate this threat we recommend the following scheme:

Normally challenges for signatures are derived using _Random Oracle Model_, which is initialized with the message being-signed, this binds the signature for this specific message.

So we recommend deriving an arbitrary ephemeral (unique) key. Its image (pubKey) should be included in the method arguments, and initialize the _Oracle_ (challenges derivation), **AND** it also should be included in the <u>native</u> kernel signature (i.e. `Env::AddSig()`).

So we recommend using a hybrid of a custom and native signatures. Instead of binding the signature to a message, we recommend binding it to an ephemeral key, whose signature in turn is bound to the kernel blidning factor. By such it's no more feasible to hijack the signature.

# Building a mutli-signed transaction using native signature

In simple scenario the app calls `Env::GenerateKernel()`, and supplies the info about how it should be signed (gives array of `SigRequest`), the signing itself is deferred until a later stage. Hence some kernel parameters may still vary: min/max height, fee.

For multi-signed transactions this is different. All the kernel parameters must be decided in advance, so that all the challenges can be derived.
Those include:

* Standard parameters: **contractID**, method number, args
* min/max heights
* kernel fee
* Funds balance
* Kernel blinding factor image
   * together with Funds balance is used to derive kernel `Commitment`
* Total nonce image, which includes
   * internal BVM nonce for the kernel blinding factor
   * all the nonces the app uses for the keys used in the signature
   * all the nonces the apps running in different wallets use for their keys

From all this the challenges are derived.

Then it's the apps's task to calculate the signature: the sum of all the needed blinded keys (its own, as well as of other users).

Technically the flow goes like this:
* Decide auxiliary kernel parameters (heights, fee)
* get nonce images
* get the image of additional nonce, which would be used by the BVM to blind the kernel blinding factor
* for all the co-signers:
  * co-signers send their (cumulative) nonces for their keys
* sum all the nonces
* Call Env::GenerateKernelAdvanced() for the 1st time to derive the challenges
* Extract the blinded keys, using the appropriate nonces and derived challenges
* for all the co-signers:
  * Signer sends them the total nonce and the kernel blidning factor, so that they cal also derive the challenges
  * co-Signers reply with the sum of their blinded keys
* sum those blinded keys to get a multi-signature
* Invoke Env::GenerateKernelAdvanced() for the 2nd time, to finally build the transaction.

Some examples of this flow:
* **vault** app: example of a seamless multisignature, where a vault `PubKey` is actually a sum of the keys of 2 users. The contract code is unmodified (i.e. doesn't care if this is a multi-owned account).
* **upgradable2** contract and app: example of an explicit multisignature, where the contract demands authorization by multiple keys.
* more examples to come, to demonstrate custom signature schemes

# Security considerations

When app is invoked in the wallet, it's possible to define its _privilege level_, which may restrict its access to potentially dangerous functionality.

* Level 0: apps have no access to any user-specific info, and no permission to create transactions. They can only interpret and display the blockchain status information. This is considered safe (though of course the info that app displays may be misleading).

* Level 1 (default): Apps have an access to user's public keys (only those related to contracts), and can ask to create transactions. For the transactions the user sees the resulting balance (funds in/out), and they need user approval. There are 2 risks to consider:
  * There is no reliable way to know it the funds are spent as supposed. A malicious app can steal those funds by sending them such that they won't be accessible by the user in the future.
  * App may try to deanonymize the user. It may derive multiple public keys, which are normally intended for different contracts, and expose them in some transaction in a seamless way (a disguised parameter to some contract, recognized by the attacker).
* Level 2: Apps have access to blinded keys.
  * A malicious app can sign **arbitrary** transaction to steal the user funds.
  * Although such a risk already exists at privilege level 1, here it's somewhat greater, because the leaked signature may be used later, not necessarily during the app invocation.
* Level 3: Apps have access to inter-wallet communication.
  * This somewhat increases the risk of disguised signature leakage: Rather than hiding the leaked signature for the attacker on-chain in the explicit tx, a malicious app can now communicate it directly to the attacker.

## Conclusion
At any privilege level above 0 there is a risk of deanonymization and funds theft. So we strongly suggest using only trusted apps: either reviewed and built by the user, or signed by trusted 3rd-party developers.

Anyway the deanonymization and theft risks are bounded by the contracts. Apps have no access to other wallet keys (UTXOs, shielded outputs, onwer key, and etc.
