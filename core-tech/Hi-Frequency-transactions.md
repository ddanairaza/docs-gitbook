## Standard transaction handling

Standard Beam transactions are considered independent. After they are broadcast to the network (in the _fluff_ phase) they are kept in a _Transaction pool_ in each node, until they either expire or included in a block. Miners select the transactions according to their Fee/Size ratio, to maximize the reward.

The transaction lifespan (e.i. _height range_) is controlled by its creator, but usually order of several hours. Once the transaction is broadcast, the wallet monitors the new blocks, until either it notices the transaction is included in a block (succeeded), or expires (failed).


## DeFi

While the above scheme is OK for normal transactions, smart contracts demand a different design, because transactions are potentially <u>dependent</u>, their order and block height may have consequences for users, or even render some of them invalid.

So we designed a concept of _dependent_ transactions. Those that insist to be included in a specified block and at the specified position, or not included at all. 
In contrast to normal transactions, that are stored in a _flat_ transaction pool, dependent transactions are stored in a special tree. Then in order to assemble a block, the miner picks the branch with maximum total fees, and includes all the appropriate transactions in this exact order.

When working in the HFT mode, the wallet not only gets notifications about new blocks, but also about the changes in the dependent transactions tree. Then, in order to build a transaction, it does the following:
* Gets the information about the current transaction branch with the maximum total fee (the one that is most likely to be selected by the miner)
* Specifies this branch ID when querying the appropriate contract information. By such it can foresee the effects of all the preceding transactions in the branch, and build a new transactions accordingly.
* When ready, broadcast this transaction to the node, with the branch ID that defines uniquely the position where this transaction should be included.
* Even before the new block is mined, the wallet gets notification about the transaction tree updates. If for some reason the current transaction is unlikely to be mined (another competing branch has more total fee) - the wallet may rebuild and resend transaction according to the new context.
* Optionally this may be repeated continuously, until the transaction is mined.

By such we achieve the following goals:
1. Strict transaction inclusion rule. Protect the user from unexpected side effects (a.k.a. front running attack).
2. Give users prompt pseudo-confirmation for their transactions.
3. Give opportunity for several users to see effects of other's transactions, hence include otherwise conflicting transactions in the same block.

# Technical design

(in progress)

