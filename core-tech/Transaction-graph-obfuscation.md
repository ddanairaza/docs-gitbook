MW offers great anonymity out-of-the-box, because of the following:
1. No addresses
1. Transaction values are blinded
1. Transaction graph is <u>obfuscated</u>.

Now, it may sound surprising, but the (3) is actually a **misconception**.

### What? Why?

As we know, original transactions are merged in a block non-interactively, so that the block is one big transaction, and there is no feasible way to recover the original transaction graph. All the UTXOs look just as arbitrary EC points, and every input can literally correspond to any output. And the bigger the block is - the higher is the obfuscation degree.

But let's think how the transactions make their way to the block. First they are prepared by the participants, and then they are _broadcasted_ to the network. And the participants are interested to broadcast it to the entire network, to as many nodes as possible, to increase the probability of it being accepted in a block, and the block mined eventually.

If this is indeed the case, i.e. transactions are just broadcasted as-is to the entire network, then the attacker can easily see the original transaction graph with just a <u>single</u> malicious node, and the later obfuscation in the block doesn't matter anymore.

### So what? No addresses, so why bother?

Is the transaction graph really important? It is. Moreover, in MW hiding the transaction graph seems to be way more important than hiding the user identities.

MW transaction is anonymous, but it reveals one important thing: there is clearly a <u>relation</u> between the users. So looking at the transaction graph attacker sees the "relation" graph. If arbitrary user gets revealed (for whatever reason) - he can disclose the related users, and the attacker knows for sure there is a relation.

#### Example

Suppose Bob has a store, and Alice is his rival, she wants to know Bob's supplier. So she pays Bob (buys something from him), then Bob pays his supplier Charlie, later Charlie pays Dan, Dan pays Erin. Alice sees all those transactions, but has no idea of user identities.

Eventually Erin gets revealed - buys something from Alice for instance. Alice kindly asks [bribes / threatens / tortures] Erin to tell her who did he get that UTXO from, this way Dan gets revealed. And so on. At every step Alice is certain there is a relation to the next user.

In contrast, if we assume that the user identity are not well concealed, but the transaction graph is obfuscated - then there is almost no problem. Attacker only gathers the information that a specific user performs a transaction, without any knowledge of the transaction amount, and who is that designated for.

### Are there existing solutions?

Of course. There are known solutions: CoinShuffle, ValueShuffle, but they are not perfect.
* This requires groups of unrelated people to cooperate
* Attacker may create many malicious users "for free", that would pretend to participate in CoinShuffle, but in practice can:
   * Learn the transaction graph. Would act as unrelated users, but actually belong to the attacker.
   * DoS attack: created invalid transactions (reference non-existing inputs, etc.)

Those techniques may be useful, but may cause hassles for the users: more preparation time, higher chance to spoil the transaction.
This means in turn that the majority of users may decide to skip this obfuscation, and broadcast the original transaction as-is, because "they have nothing to hide". This naturally would affect the privacy of other users (anonymity works best when obeyed by all the users).

## Proposition: In-Node obfuscation

Given the fact that MW transactions are merged non-interactively - Nodes can automatically obfuscate the original transaction graph up to some degree. It may not necessarily replace, but complement transparently the obfuscation done by the users.

This can easily be done in a modified Dandelion. Though originally developed to conceal the user identity, it can be adopted to obfuscate transaction graph during the stem phase, where the transactions get passed through several nodes, but before they are broadcasted to the entire network.

### Non-interactive merge

That is, while in the stem phase, instead of immediately passing the transaction to a single peer, the Node may wait for some reasonable timeout to try to merge it with another one. So that the transaction grows like a snowball.

The following however should be taken into consideration
* To be "fair" Node should only merge transactions with comparable fee/size ratio. Otherwise this would reduce the motivation for users to increase the fee (if it'll be diffused anyway).
* In particular a Node can just  "hijack" the fee: append its transaction without any fee to another one.
   * There's nothing can be done to prevent this, but users may see the unfair behavior a-posteriori, and ban that Node.
* Malicious Node can record the transactions that it passes in the stem phase, and then see the difference when they're broadcasted in the fluff phase.
   * To minimize the amount of leaked info Nodes should merge only transactions of comparable size, instead of incrementally add small transactions.
* To prevent DoS attacks Nodes should ensure there are no conflicts. Means:
   * All the being-merged transactions are valid, and reference existing inputs
   * No double-spends or etc.

### Dummy UTXOs

Another possibility: any Node can easily "enlarge" a transaction by appending one or several dummy outputs to it. By "dummy" we mean an UTXO which encodes zero value, but looks just as good as the others.

For every such an appended dummy UTXO the Node sets a random timer (in terms of blocks num), after which it would append the same UTXO as an input in a random transaction later, which (probably) has no relation to the original one.

By this the Node creates a "background activity", which is mixed with and should be indistinguishable from the real activity.

The obvious disadvantage of this scheme is the creation of dummy UTXOs, waste of the block space, and network traffic + resources for validating it. But the good part is that it won't affect the scalability in the long-run, since kernels aren't created, and all the dummy UTXOs are spent eventually.

In practice it seems that a combination of both schemes should be used: merge real transactions whenever possible, or add dummies.

### How much should the transaction graph be obfuscated

No exact numbers yet. But it seems that even a slight obfuscation has a dramatic positive effect on the anonymity. For instance, if we merge just 2 transactions at once, it already creates roughly 1/2 uncertainty for the input-output relation. So that if an attacker reveals a user after 10 hops - there is only roughly a 10^-3 probability for the user relation.

Since the transaction being-passed doesn't have an indication is it original or not, the obfuscation criteria should be based on the visible transaction parameters. Which comes down to the number of inputs and outputs.
