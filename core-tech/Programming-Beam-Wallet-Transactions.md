This section describes the structure of Beam transactions. Transactions are created and managed locally in the Beam wallet. 

## Transaction parameters

Transactions are defined and stored in the database as a set of transaction parameters, thus allowing a lot of flexibility.

The complete list of transaction parameters is defined [here](https://github.com/BeamMW/beam/blob/edf0bb8a16626f6e1160bc09a7af4e0a40adb813/wallet/common.h#L226)

Transaction parameters are divided into two groups. **Public** parameters can be set during the interaction between wallets. Once public parameter is set, it can not be changed. **Private** parameters, can be set and reset at any time within the wallet and are not sent outside the wallet. The reason for this separation is to maintain a clear distinction between the parameters that are revealed to or received from the other wallet and can thus irreversibly affect the state of negotiation between the wallets creating the transaction.

## Transaction id

TxID, [defined here](https://github.com/BeamMW/beam/blob/edf0bb8a16626f6e1160bc09a7af4e0a40adb813/wallet/common.h#L37) is the unique id of the transaction

## ITransaction interface [ [definition](https://github.com/BeamMW/beam/blob/edf0bb8a16626f6e1160bc09a7af4e0a40adb813/wallet/base_transaction.h#L28) ]

Defines basic set of operations for each transaction, Update, Cancel and Rollback

```c++
struct ITransaction
    {
        using Ptr = std::shared_ptr<ITransaction>;
        virtual TxType GetType() const = 0;
        virtual void Update() = 0;
        virtual void Cancel() = 0;
        virtual bool Rollback(Height height) = 0;
    };

```

## BaseTransaction [ [definition](https://github.com/BeamMW/beam/blob/edf0bb8a16626f6e1160bc09a7af4e0a40adb813/wallet/base_transaction.h#L55) ]

Implements the ITransaction interface and provides the state machine for creation of transactions. Constructor for BaseTransaction receives three parameters:

```c++

BaseTransaction(INegotiatorGateway& gateway, beam::IWalletDB::Ptr walletDB, const TxID& txID);

```


## BaseTxBuilder

This class encapsulates all the methods necessary for transaction creation following the protocol described [here](https://github.com/BeamMW/beam/wiki/Transaction-creation-protocol).



## INegotiatorGateway [ [definition](https://github.com/BeamMW/beam/blob/edf0bb8a16626f6e1160bc09a7af4e0a40adb813/wallet/common.h#L409) ]

Negotiator Gateway provides an abstraction of the messaging between the node or another wallet during the transaction creation. The INegotiatorGateway interface is implemented by the Wallet class.


## BaseTransaction::Update method

The Update method describes generic logic of asynchronous transaction update. The update logic is implemented in the UpdateImpl method described below. 

```c++

void BaseTransaction::Update()
    {
        AsyncContextHolder async(m_Gateway);
        try
        {
            if (CheckExternalFailures())
            {
                return;
            }

            UpdateImpl();

            CheckExpired();
        }
        catch (const TransactionFailedException& ex)
        {
            LOG_ERROR() << GetTxID() << " exception msg: " << ex.what();
            OnFailed(ex.GetReason(), ex.ShouldNofify());
        }
        catch (const exception& ex)
        {
            LOG_ERROR() << GetTxID() << " exception msg: " << ex.what();
            OnFailed(TxFailureReason::Unknown);
        }
    }

```

## SimpleTransaction ( [wallet_transaction.h](https://github.com/BeamMW/beam/blob/edf0bb8a16626f6e1160bc09a7af4e0a40adb813/wallet/wallet_transaction.h#L1) )

SimpleTransaction extends BaseTransaction and provides an implementation of the basic confidential transaction between two wallets. The key part of the implementation is the `void SimpleTransaction::UpdateImpl()` method that provides the state machine for the transaction creation logic.

//TODO: Add detailed explanation for the transaction creation logic


