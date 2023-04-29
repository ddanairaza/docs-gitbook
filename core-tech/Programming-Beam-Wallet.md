> The following document is still under construction and is subject to changes

## Key Differences of Mimblewimble Wallets

Since Beam is an implementation of the Mimblewimble protocol, it has all the nuances that are specific to this protocol and have a huge impact on the wallet architecture, making it very different in comparison to most existing cryptocurrencies. Such changes include:

1. Not all the information can be extracted from the blockchain. In fact, only the UTXOs are stored in the blockchain, hence all additional information regarding transactions, addresses etc... has to be locally stored in the wallet. 

2. Transaction is created interactively by both the sender and the receiver. Hence, a state machine and communication model is required between two wallets, a problem which does not exist in Bitcoin and most other cryprocurrencies where transaction can be created by a single wallet and immediately sent to the network. 

This means that Beam Wallet has several different responsibilities:

1. Storing and managing the list of Coins (UTXO set)
2. Storing and managing SBBS Addresses (described below) for communication between wallets
3. Allow wallet to wallet (W2W) communications and interactive creation of new transactions
4. Handle wallet to node (W2N) communications, send transactions to the node, request proofs for kernels and UTXOs and process blockchain updates and manage transaction and Coin state

## Wallet Code Structure and High Level Architecture 

Beam Wallet has the following conceptual modules (located in the wallet folder)

* [core]()
* [database](https://github.com/BeamMW/beam/wiki/Beam-Wallet-Database)
* [transactions](https://github.com/BeamMW/beam/wiki/Programming-Beam-Wallet-Transactions)
* [wallet]()
* [wallet_client]()

The structure of the files and folders is currently being refactored to match the structure described above



