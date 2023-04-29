# User info
* Address (can be many): Unique ID money can be sent to/from. Each user/wallet can create and use any number of addresses. Address is assigned to one or more address categories (each category is denoted by name and color).
* Name: Meaningful name is kept for semantic identification only (in a separate Directory service, should not be unique).
* Avatar: Small and nice picture for the further identification.

# Address
* a list of categories it is assigned to
* a textual description (optional)
* additional attributes such as: single use (won't be valid after the transaction is completed or failed) or time expiration (won't be valid once expired)

# Balance
* Amounts, aggregated by transaction states:
    * Available: always shown, with LARGE font
    * In progress: Sending, Receiving, Change etc: shown when applicable (Alexandra Shelenkova do we need it?)
    * Locked: shown separately when applicable (sub-cases: "maturing" etc)
* currency is:
    * BEAM
    * GROTH (1*10^-8 of BEAM)
    * Consider estimated amount in BTC/ETH later on

# UTXO details
* ID
* Amount
* Transactions it had participated in
    * As output UTXO for incoming transaction, "earned" time is specified
    * As input UTXO for outgoing transaction, "spent" time is specified
* Coin type
    * Regular: just a regular coin, can receive most of the statuses (see below)
        * Regular (received): received upon a successful completion of an incoming transaction
        * Regular (change): received by the sender when his UTXO is split during a successfully accomplished outgoing transaction
    * Transaction fee: paid by the user, is always created with "Maturing" status
    * Coinbase: mined when miner closed the block, is always created with "Maturing" status
    * Treasury: generated for maintenance and development of the project
* Status
    * Available: not in the transaction, can be spent
    * In progress, can have the following sub-cases:
        * Incoming (draft) or Incoming: the amount is locked because of the ongoing incoming transaction
            * Example:
                * Alice has single UTXO of 100 BEAM and wants to send it fully to Bob (zero transaction fee for clarity)
                * Bob sees 100 BEAM as "Incoming (draft)" before that UTXO was registered in the node/blockchain 
                * Bob sees 100 BEAM as "Incoming" when that UTXO was recognized registered in the node/blockchain
        * Outgoing: the amount is locked because of the ongoing outgoing transaction, new UTXO will be generated as a change once the transaction will be competed
            * Example:
                * Alice has single UTXO of 100 BEAM and wants to send 20 BEAM to Bob (zero transaction fee for clarity)
                * Alice sees 100 BEAM are "outgoing (locked)" at first
                * Alice sees 80 BEAM as "incoming (change)" and Bob sees 20 BEAM "incoming"
                * Alice sees 80 BEAM as "Available" and Bob sees 20 BEAM "Available"
        * Maturing
            * Reserved till <time>: the amount will become available after the "locked" timeout will expire
            * Mined till block height <height> (similar to "Reserved", but measured in block height): the mined amount becomes available to miners only after the block height will be reached
    * Spent: gone with the completed outgoing transaction
    * Unavailable: for UTXO that cannot be spent anymore (due to transaction rollback or mining result rollback, the reason should be supplied just like for the "Locked" status)
* Incoming Transaction details (if available, for example is not mined locally)
    * Transaction ID - when clicked, should be taken to transaction list with the designated transaction details opened
    * Transaction completion time (ie time when UTXO was received)
    * Transaction Address ID (and its address group name and category if available)
* Outgoing Transaction details (if available, for example not spent yet)
    * Transaction ID - when clicked, should be taken to transaction list with the designated transaction details opened
    * Transaction completion time (ie time when UTXO was spent)
    * Transaction Address ID (and its address group name and category if available)
* A history of status changes with its respective timestamps etc

# Transaction details
* amount: shown as <currency + amount + state>
* short state (displayed in transaction status in mobile)
    * sending/receiving for Send/Receive
    * in progress for Atomic Swap
    * sent/received for Send/Receive
    * completed for Atomic Swap
    * cancelled for both
    * failed for both
    * expired for both
* detailed state (displayed on desktop or in transaction details in mobile)
    * "Waiting for network sync to complete" - to indicate that transaction can't be started before the network sync is accomplished
    * "Waiting for Sender/Waiting for Receiver" - to indicate that sender or receiver should come online to initiate the transaction
    * "Handshaking" - two wallets are negotiating the transaction details
    * "Syncing with blockchain" - the transaction is being registered with the node
    * "Sending/Receiving" - the transaction is taken care by the blockchain, some miner needs to PoW and to add it to a block, the block should be added to the blockchain
    * "Sent/Received" - the transaction is completed
    * "Cancelled" (by Sender, due to Rollback)
    * "Expired" (by timeout)
    * "Failed" (should come with the user-friendly actionable reason/description)
* time for every status change in the transaction lifecycle such as:
    * creation timestamp
    * peer online timestamp
    * peer offline timestamp
    * timestamp for each confirmation
    * deletion time (where appropriate)
    * peer addresses (sending, receiving escrow in the future etc), each address can be assigned to an address groups
    * related UTXOs (ie IDs of transactions, outputs of which serve as inputs for the current transaction)
    * textual annotation (optional), sender and receiver have separate annotations, each is stored locally



