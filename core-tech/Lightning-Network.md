## a.k.a. Laser Beam

# What is Lightning Network
As in traditional blockchains (btc-alike), in Beam lightning network two (or more) users lock some funds, and then effectively transfer them _off-chain_. This has the following advantages:
* Instant. No need to wait for transaction confirmation (after initial setup).
* No transaction fee.
* Hidden. The information about intermediate transactions is never broadcasted. The number of off-chain transactions, frequency and amounts are never revealed.

All the transfers are performed in the context of the _Lightning channel_. Its lifetime consists of the following:
1. Channel Open
   * Users prepare and broadcast the transaction that consumes their inputs, and create a multi-signed output, which can be spent collectively.
   * At this point each user <u>already has a refund transaction</u>, which can be used to get the funds back.
2. Off-chain funds transfers (arbitrary number of times).
   * Users agree on another partition of the locked funds, and generate newer refund transactions, with appropriate amounts to each, which is effectively equivalent to the funds transfer.
   * <u>Older refund transactions are revoked</u> (more about this later). Nothing is broadcasted to the network.
3. Channel close.
   * Each user any moment can broadcast the most recent refund transaction, to get the funds back.
   * All the users notice this, and the channel is considered closed.
   * In case the users cooperate and perform a _graceful_ channel closure - it'd be faster, and with less fee (more about this later).

# Implementation in Beam 

Since there are no scripts in MW, all the relevant functionality is implemented within the context of the transaction negotiation. This is called _scriptless scripts_.

The actual negotiation flow is somewhat complex, but logically it can be split into separate _building blocks_.

### MultiSig - multisigned UTXO
An UTXO created by several users, whose blinding factor consists of their blinding factor. Such an UTXO looks indistinguishable from others. The following operations can only be done collectively by all the users:
* Creating the UTXO (commitment + bulletproof)
* Transaction that creates it (has as an output)
* Transaction that spends it (has as an input)

### Relative timelock
This feature enables to create 2 dependent transactions, whereas the second one becomes valid only after the first one is already accepted in a block, and the block is mature enough. Means - there is a minimum enforced delay between the transactions.
It requires a special processing on the node side, and will be available after the planned Hard Fork 1.

Technically each kernel may optionally contain the Relative Lock info, which consists of:
* Kernel ID (hash) being referenced
* Minimum maturity (height difference)

When such a kernel is encountered the node verifies if indeed the referenced kernel exists in the blockchain, and is mature enough.

### Refund procedure
Refund procedure consists of 2 transactions, which consume the locked funds and create the agreed user outputs.
1. MultiSig.0 -> MultiSig.N
1. MultiSig.N -> Outputs.N
   * Relatively time-locked w.r.t. transaction (1).

whereas `MultiSig.0` represents the funds locked in the channel, `MultiSig.N` is an intermediate multisigned UTXO, and `Outputs.N` are all the outputs that users are supposed to get by the refund.

The transaction (2) is time-locked w.r.t. transaction (1). Means, after (1) was broadcasted, the `MultiSig.N` becomes visible in the blockchain, but the transaction (2) can't be used immediately.
As we'll see this is important. In case a malicious user would try to use not the most recently agreed refund procedure, it gives a time window for the affected user to respond.

### Refund revocation
As we said, there is a way to revoke the refund procedure. This should prevent malicious users from trying to use an older refund procedure after a newer agreement was set.

In Beam in order to revoke the N-th refund procedure, the user **reveals its blinding factor** that was used for `MultiSig.N`.

Once the blinding factor was revealed the user won't be able to use the refund procedure. If it does, then after transaction (1) the `MultiSig.N` becomes visible in the blockchain, and the peer can consume it immediately, because it knows the overall blinding factor. Just build and broadcast a transaction immediately that takes it as an input, and creates an output that belongs solely to it, i.e. punish.

This is why relative timelock is important.

### Multi-user Refund procedure
Now let's see how 2 users (A)lice and (B)ob build and use their refund procedures. We'll study the case for 2 users, but it can easily be generalized for arbitrary number of users.

To build N-th refund procedure (A) and (B) negotiate to build the following transactions:
* Refund.N.A.1
   * MultiSig.0 -> MultiSig.N.A
* Refund.N.A.2
   * MultiSig.N.A -> Outputs.N
* Refund.N.B.1
   * MultiSig.0 -> MultiSig.N.B
* Refund.N.B.2
   * MultiSig.B.A -> Outputs.N

So there are 4 transactions overall, all of them are built collectively. At the end <u>each user has its own refund procedure</u>, with different intermediate `MultiSig.N.X`, but the final `Outputs.N` are the same.

Note the following:
* The transaction (1) of each refund procedure is kept **private**. Means - only (A) has the Refund.N.A.1, and only (B) has the Refund.N.B.1
* The transaction (2) of each refund procedure is **public**. Means - both (A) and (B) have Refund.N.A.2 and Refund.N.B.2
* The order of transaction building is important. Both (A) and (B) should **NOT** complete their parts of transaction (1) for the peer, before they have the appropriate transaction (2).
   * Otherwise a malicious user can just use the transaction (1) to lock the funds permanently.

So, the idea is that (A) and (B) have their own refund procedures, partly kept private. This separation is important, since in case they come to another agreement and create a newer refund procedures - they will need to <u>revoke</u> the older ones, which means the appropriate blinding factors of `MultiSig.N.A` and `MultiSig.N.B` must be revealed. As we already mentioned, using a compromised transaction (1) would lead to loss of funds. But, importantly, we must also guarantee that user that gets the revealed blinding factor can't initiate the compromised path on its own. This is the reason why the appropriate transactions (1) are private.

# Lightning channel from the building blocks.

Conceptually the lightning channel operates the following way:

### Channel open
1. Users agree on how much funds each of them locks.
1. Create the `MultiSig.0`
1. Build `Refund.1`
1. Create a transaction `Inputs` -> `MultiSig.0` + `Changes`
1. Wait for confirmation

Of course the order is important. Users should not make (4) available before they have (3).

### Off-chain funds transfer
1. Users agree on newer partition of the locked funds.
2. Negotiate to build `Refund.N`
3. Revoke the previous `Refund.N-1`

### Graceful channel closure.
1. Users agree to close the channel gracefully.
1. Create a transaction `MultiSig.0` -> `Outputs.N`

This scheme makes the withdrawal immediate. No timelocks are needed.

### One-side channel closure.
1. User decides to invoke its refund procedure (in case there's no cooperation)
1. User uses its latest refund procedure (all the others are revoked).

### Blockchain status monitoring (a.k.a. watch towers)
In addition to the voluntary actions, all the users should monitor the blockchain to detect if/when any of them made one-side actions.
Depends on the agreed timelock (which is applied in all the relative locks of all the refund procedures) - the monitoring doesn't have to be for each new block. It can be once in many blocks as well.

Once a new block arrives and the user decides to check the status:
* Does `MultiSig.0` present?
   * Yes - channel is open. No further actions.
   * No
      * Was the channel open already?
         * No - we're still opening it (waiting for the 1st tx).
         * Yes - the channel is being closed!
            * Find one of he `MultiSig.N` that was created for the refund procedures.
            * Does it belong to the revoked refund?
               * Yes - cheat attempt detected!!!
                  * Claim all the funds immediately.
               * No - valid withdrawal triggered
                  * Wait until the timelock expires
                  * Broadcast the appropriate transaction (2)

## Code design - Minimizing the number of negotiation roundtrips.

As we saw, each operation on the Lightning channel requires to prepare many different _building blocks_, with dependencies. Some are literally dependent on each other (some of their results are needed by others), whereas for some there are artificial order restrictions to prevent malicious users from doing harm.

Nevertheless, many such negotiations may run in parallel. At least partially, up to the point where their dependencies come into play.

To allow this Beam code infrastructure allows the negotiations in terms of _primitives_, each is responsible for a well-defined functionality (such as creating a MultiSig, or a transaction), each has an interface to load/store the parameters and transfer the data to the peer.
When those primitives are aggregated to create a more complex negotiation scheme - their inputs/outputs are "re-routed", to reflect their dependency.

By such the code remains relatively not too complex (readable), the dependencies are visible and can be verified, whereas on the other hand the negotiations effectively run in parallel, and each action is performed in a minimum number of negotiation roundtrips.

# How typical negotiations look

Below are the real negotiations produced by our code.

### MultiSig
        A -> B 115 bytes
             Partial Commitment
             Bulletproof T1,T2
        B -> A 155 bytes
             Partial Commitment
             Bulletproof T1,T2
             Bulletproof TauX
        B done
        A done

Overall 1 roundtrip, assuming both users get the commitment, but only A has the valid bulletproof

### Refund, one-sided

        A -> B 279 bytes
             MultiSig.Partial Commitment
             MultiSig.Bulletproof T1,T2
             Tx-TLock.Excess Commitment
             Tx-TLock.Nonce Commitment
             Tx-Final.Excess Commitment
             Tx-Final.Nonce Commitment
        B -> A 1158 bytes
             MultiSig.Partial Commitment
             MultiSig.Bulletproof T1,T2
             MultiSig.Bulletproof TauX
             Tx-TLock.Excess Commitment
             Tx-TLock.Nonce Commitment
             Tx-TLock.Partial Kernel Signature
             Tx-Final.Excess Commitment
             Tx-Final.Nonce Commitment
             Tx-Final.Partial Kernel Signature
             Tx-Final.Partial Transaction
        A -> B 925 bytes
             Tx-Final.Partial Transaction
        B -> A 52 bytes
             Tx-TLock.Partial Transaction
        B done
        A done
Overall 2 roundtrips. Note that B delays the completion of `Tx-TLock` for A until it gets and validates the `Tx-Final` from it.


Now let's see how the Lightning channel operations are negotiated.

## Lightning channel open
        A -> B 591 bytes
             MultiSig.Partial Commitment
             MultiSig.Bulletproof T1,T2
             Tx-Open.Excess Commitment
             Tx-Open.Nonce Commitment
             Exit-A.MultiSig.Partial Commitment
             Exit-A.MultiSig.Bulletproof T1,T2
             Exit-A.Tx-TLock.Excess Commitment
             Exit-A.Tx-TLock.Nonce Commitment
             Exit-A.Tx-Final.Excess Commitment
             Exit-A.Tx-Final.Nonce Commitment
             Exit-B.MultiSig.Partial Commitment
             Exit-B.MultiSig.Bulletproof T1,T2
        B -> A 1754 bytes
             MultiSig.Partial Commitment
             MultiSig.Bulletproof T1,T2
             MultiSig.Bulletproof TauX
             Tx-Open.Excess Commitment
             Tx-Open.Nonce Commitment
             Tx-Open.Partial Kernel Signature
             Exit-A.MultiSig.Partial Commitment
             Exit-A.MultiSig.Bulletproof T1,T2
             Exit-A.MultiSig.Bulletproof TauX
             Exit-A.Tx-TLock.Excess Commitment
             Exit-A.Tx-TLock.Nonce Commitment
             Exit-A.Tx-TLock.Partial Kernel Signature
             Exit-A.Tx-Final.Excess Commitment
             Exit-A.Tx-Final.Nonce Commitment
             Exit-A.Tx-Final.Partial Kernel Signature
             Exit-A.Tx-Final.Partial Transaction
             Exit-B.MultiSig.Partial Commitment
             Exit-B.MultiSig.Bulletproof T1,T2
             Exit-B.MultiSig.Bulletproof TauX
             Exit-B.Tx-TLock.Excess Commitment
             Exit-B.Tx-TLock.Nonce Commitment
             Exit-B.Tx-Final.Excess Commitment
             Exit-B.Tx-Final.Nonce Commitment
        A -> B 1968 bytes
             Exit-A.Tx-Final.Partial Transaction
             Exit-B.MultiSig.Bulletproof TauX
             Exit-B.Tx-TLock.Excess Commitment
             Exit-B.Tx-TLock.Nonce Commitment
             Exit-B.Tx-TLock.Partial Kernel Signature
             Exit-B.Tx-Final.Excess Commitment
             Exit-B.Tx-Final.Nonce Commitment
             Exit-B.Tx-Final.Partial Kernel Signature
             Exit-B.Tx-Final.Partial Transaction
        B -> A 977 bytes
             Exit-A.Tx-TLock.Partial Transaction
             Exit-B.Tx-Final.Partial Transaction
        A -> B 52 bytes
             Exit-B.Tx-TLock.Partial Transaction
        B -> A 85 bytes
             Tx-Open.Partial Transaction
        B done
        A done

## Lightning channel update (funds transfer)
        A -> B 394 bytes
             Exit-A.MultiSig.Partial Commitment
             Exit-A.MultiSig.Bulletproof T1,T2
             Exit-A.Tx-TLock.Excess Commitment
             Exit-A.Tx-TLock.Nonce Commitment
             Exit-A.Tx-Final.Excess Commitment
             Exit-A.Tx-Final.Nonce Commitment
             Exit-B.MultiSig.Partial Commitment
             Exit-B.MultiSig.Bulletproof T1,T2
        B -> A 1477 bytes
             Exit-A.MultiSig.Partial Commitment
             Exit-A.MultiSig.Bulletproof T1,T2
             Exit-A.MultiSig.Bulletproof TauX
             Exit-A.Tx-TLock.Excess Commitment
             Exit-A.Tx-TLock.Nonce Commitment
             Exit-A.Tx-TLock.Partial Kernel Signature
             Exit-A.Tx-Final.Excess Commitment
             Exit-A.Tx-Final.Nonce Commitment
             Exit-A.Tx-Final.Partial Kernel Signature
             Exit-A.Tx-Final.Partial Transaction
             Exit-B.MultiSig.Partial Commitment
             Exit-B.MultiSig.Bulletproof T1,T2
             Exit-B.MultiSig.Bulletproof TauX
             Exit-B.Tx-TLock.Excess Commitment
             Exit-B.Tx-TLock.Nonce Commitment
             Exit-B.Tx-Final.Excess Commitment
             Exit-B.Tx-Final.Nonce Commitment
        A -> B 1968 bytes
             Exit-A.Tx-Final.Partial Transaction
             Exit-B.MultiSig.Bulletproof TauX
             Exit-B.Tx-TLock.Excess Commitment
             Exit-B.Tx-TLock.Nonce Commitment
             Exit-B.Tx-TLock.Partial Kernel Signature
             Exit-B.Tx-Final.Excess Commitment
             Exit-B.Tx-Final.Nonce Commitment
             Exit-B.Tx-Final.Partial Kernel Signature
             Exit-B.Tx-Final.Partial Transaction
        B -> A 977 bytes
             Exit-A.Tx-TLock.Partial Transaction
             Exit-B.Tx-Final.Partial Transaction
        A -> B 92 bytes
             Reveal Previous Blinding Factor
             Exit-B.Tx-TLock.Partial Transaction
        B -> A 40 bytes
             Reveal Previous Blinding Factor
        B done
        A done

So, both channel open and funds transfer negotiations are completed in 3 full roundtrips, whereas all the dependencies are observed.

# Demo

There's a working demo of the Lightning channel in our codebase here: `beam/node/laser_beam_demo`

Various scenarios are emulated, such as graceful channel opening and closure, one-side channel closure and appropriate user responses, and the cheat attempt and punishing.

The node used in the demo is the standard Beam node (no hacks, workarounds, or other tricks specifically for the demo), configured to generate fake PoW. All the broadcasted transactions and timelocks are fully validated.
