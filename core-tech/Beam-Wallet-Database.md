Since in Mimblewimble only UTXO related information is available on chain, most of the Wallet state should be maintained locally and store in the Wallet Database, which is described in this chapter. 

Beam Wallet stores and tracks information about four key entities:
1. Coins
2. Addresses
3. Transactions
4. Blockchain State

The interface for the database is specified in the [wallet_db.h](https://github.com/BeamMW/beam/blob/56e9cdd7be211649a576fa15d3f0a97922ae2acd/wallet/wallet_db.h#L164) file

Beam currently uses sqlite relational database in the implementation

# Coins

We will start with the [definition](https://github.com/BeamMW/beam/blob/56e9cdd7be211649a576fa15d3f0a97922ae2acd/wallet/wallet_db.h#L41) of a Coin structure, which represents a UTXO as it is seen by the wallet.

```c++

    // Describes a UTXO in the context of the Wallet
    struct Coin
    {
        // Status is not stored in the database but can be
        // deduced from the current blockchain state
        enum Status
        {
            Unavailable, // initial status of a new UTXO
            Available,   // UTXO is currently present in the chain and can be spent
            Maturing,    // UTXO is present in the chain has maturity higher than current height (i.e coinbase or treasury)
            Outgoing,    // Available and participates in outgoing transaction
            Incoming,    // Outputs of incoming transaction, currently unavailable
            ChangeV0,    // deprecated.
            Spent,       // UTXO that was spent. Stored in wallet database until reset or restore

            count
        };

        Coin(Amount amount = 0, Key::Type keyType = Key::Type::Regular);
        bool operator==(const Coin&) const;
        bool operator!=(const Coin&) const;
        bool isReward() const;
        std::string toStringID() const;
        Amount getAmount() const;

        typedef Key::IDV ID; // unique identifier for the coin (including value), can be used to create blinding factor 
        ID m_ID;

        Status m_status;        // current status of the coin
        Height m_maturity;      // coin can be spent only when chain is >= this value. Valid for confirmed coins (Available, Outgoing, Incoming, Change, Spent, Maturing).

                                // The following fields are used to derive the status of the transaction
        Height m_confirmHeight; // height at which the coin was confirmed (appeared in the chain)
        Height m_spentHeight;   // height at which the coin was spent

        boost::optional<TxID> m_createTxId;  // id of the transaction which created the UTXO
        boost::optional<TxID> m_spentTxId;   // id of the transaction which spernt the UTXO
        
        uint64_t m_sessionId;   // Used in the API to lock coins for specific session (see https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API#tx_split)

        bool IsMaturityValid() const; // is/was the UTXO confirmed?
        Height get_Maturity() const; // would return MaxHeight unless the UTXO was confirmed
        
        std::string getStatusString() const;
        static boost::optional<Coin::ID> FromString(const std::string& str);
    };
```

 ## Deducing Coin Status

By monitoring the state of the blockchain, the wallet can always deduce the current status of each coin by tracking the m_confirmHeight height and m_spentHeight. This is done in the `void DeduceStatus(const IWalletDB&, Coin&, Height hTop);` method which calls the `Coin::Status GetCoinStatus(const IWalletDB&, const Coin&, Height hTop);` method that in turn returns the current status of the coin. 

```c++

Coin::Status GetCoinStatus(const IWalletDB& walletDB, const Coin& c, Height hTop)
{
    if (c.m_spentHeight != MaxHeight)
        return Coin::Status::Spent;

    if (c.m_confirmHeight != MaxHeight)
    {
        if (c.m_maturity > hTop)
            return Coin::Status::Maturing;

        if (IsOngoingTx(walletDB, c.m_spentTxId))
            return Coin::Status::Outgoing;

        return Coin::Status::Available;
    }

    if (IsOngoingTx(walletDB, c.m_createTxId))
        return Coin::Status::Incoming;

    return Coin::Status::Unavailable;
}
```

## Selecting coins for a specified amount 

Implemented in the `std::vector<Coin> selectCoins(Amount amount) override;` method. The purpose of the function is to select Coins matching a specific amount (for example in sending scenario). Selection method should minimize number of Coins and the change outputs and hence use greedy strategy with some additional heuristics. Specific strategies for coin selection are implemented in the `struct CoinSelector3` in honor of three attempts to write an effective selector for large amount of coins.

 