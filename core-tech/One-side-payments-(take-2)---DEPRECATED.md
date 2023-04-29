# -- DEPRECATED --

***



In MW transactions are built interactively, means sender and receiver must collaborate to build a transaction. Here we'll describe a scheme where the sender can pay the receiver without the latter being involved during the payment.

Previously we described such a scheme that allowed one-side payments, which demanded our extension to MW which we called _kernel fusion_. The idea was that the receiver prepares in advance its UTXO + kernel that compensates for its blinding factor. Then in order to pay the sender creates a transaction with this UTXO appended, and another kernel _fused_ with that given by the receiver. The drawback of this scheme is that it was possible to transfer fixed values only (that corresponded to the prepared UTXOs).

Here we describe a scheme without this drawback, i.e. where the sender can transfer any amount to the receiver.
The idea is to _fuse_ UTXOs rather than kernel. The receiver should prepare the UTXO only partially, then the sender would finalize it to accomplish the payment.

# Detailed description

The UTXO is redefined, and may optionally contain an _extra excess_, which is an arbitrary EC point with the Schnorr's signature that proves there's no value hidden (similar to kernel). So that when the UTXO signature (bulletproof) is created - it signs the visible UTXO Commitment minus the _extra excess_, however this _excess_ is accounted for in the signature, so that it's not possible to remove it (this is the fusion).

The receiver picks an arbitrary excess, signs both this excess and the kernel that would compensate for it. In addition it picks the KDF (key deviation function) parameters.


# Comparison of both schemes

Kernel fusion
* 
