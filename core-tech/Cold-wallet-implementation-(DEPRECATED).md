# Cold wallet implementation
## Definitions
### **cold** wallet
  * can generate private keys, i.e. it has _master key_ generated from _seed phrase_
  * has no connection with internet, as a result, it cannot send transactions to the node, and it cannot get proofs and notifications from the node

### **hot** wallet 
  * cannot generate any secrets from master key by the reason that it doesn't have it
  * has internet connection and can send/get messages from the node

## Design

The goal of this design is to get cold wallet scenario with minimal changes in existing codebase.

The main idea is to split wallet database (_wallet.db_) onto two parts: public (_wallet.db_) and private (_wallet.db.private_). The last one should hold master key generated from seed phrase.
"Cold" wallet has both of these files, "hot" has public data only. To be able to make tranasctions with this setup user have to constantly move public _wallet.db_ file between "cold" and "hot" wallets. Since we override _wallet.db_ we have to stop "hot" wallet before we copy _wallet.db_ file from/into its folder 

### Changes

1. For "Cold" wallet we don't set up connection with the node.
2. "Cold" wallet stores all outgoing encrypted messages into public database (_wallet.db_), we subclass existing behavior with SBBS
3. Add a stage for processing stored incoming messages.
4. For "hot" wallet there is a need to add some checks if it has master key, also "hot" wallet has no ability to decrypt incoming SBBS messages, since it doesn't have private keys, so it has to store all incoming messages from the channel.


## Usage
To tell wallet that it works in "cold" mode you have to add `--cold_wallet` parameter

### Initialization of "cold" wallet 

`beam-wallet init --cold_wallet`

This command will create two databases: _wallet.db_ and _wallet.db.private_

### Sending from "cold" wallet

Pre-conditions:
Make sure the cold wallet is synced. In order to do so, follow the next steps:
1. copy the _wallet.db_ file to the "hot" wallet's data folder.
2. launch the "hot" wallet and wait till it's synced.
3. stop the "hot" wallet, copy the _wallet.db_ file into the "cold" wallet folder.
4. launch the "cold" wallet for listening.
`beam-wallet listen --cold_wallet`
 
Now as the "cold" wallet is synced, proceed with the next steps:
1. In the cold wallet run the command:
`beam-wallet send -a <amount> -r<receiver address> -f <fee> --cold_wallet`
Note: there is no need in node address in this case
2. copy _wallet.db_ file to "hot" wallet's data folder
3. launch "hot" wallet. It should send encrypted message to the node, also he may get encrypted message back.
4. stop "hot" wallet, copy _wallet.db_ file into "cold" wallet folder
5. launch "cold" wallet for listening 
`beam-wallet listen --cold_wallet`
it should create a signed transaction kernel
6. copy _wallet.db_ from "cold" to "hot" 
new transaction should go to the node and got confirmed
7. copy _wallet.db_ from "hot" to "cold"
"cold" wallet should have actual balance and transactions statuses

### Receiving to "cold" wallet

1. Generate new address in "cold" wallet and send it to the sender
2. copy _wallet.db_ to "hot"
3. launch "hot" wallet. Note there will be no new transactions, since "hot" wallet cannot decrypt incoming messages
4. stop "hot". copy _wallet.db_ from "hot" to "cold"
5. launch "cold" wallet for listening, it should get new transaction and accept it.
6. copy _wallet.db_ from "cold" to "hot"
7. launch "hot", wait until new transaction becomes completed
8. copy _wallet.db_ "hot" to "cold" balance and transactions statuses should be correct.


