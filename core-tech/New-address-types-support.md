# Beam Fierce Fermion 6.0 Upgrade Guide for pools and exchanges
## Confidential Assets

Confidential Assets (CA) are tokens mint on the Beam blockchain. Beam supports Confidential Assets since hard fork 2, but this feature had limited ability to be used in real life applications. Starting from the version 6.0 Beam Wallet adds ability to create smart contracts which make CA more usable and very important feature in Beam infrastructure. We already have [API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API#assets-support) to work with CA, but it was disabled by default. If you wish to accept and create transactions with CA you should enable this feature in the wallet API and update your codebase to handle CA correctly. 

## New transaction types

New types of transaction are available in the wallet API:
* Maximum privacy transaction
* Offline transaction

In order to receive these transactions there are no need to make any changes, but if you want to send them you have to explicitly enable them in the wallet, and distinguish new address types.

## New address types

We introduced new address types in CLI and Desktop wallet version 5.0, now they are available in API. We highly recommend to use these new addresses, this should bring better user experience both for sending and receiving. For backward compatibility we preserved old address type.
The difference between new addresses and old are the following
* new addresses are `base58` encoded and have relatively long, old addresses are `hex` encoded and have length 64-67 characters
* new addresses can hold additional data for transaction, this allows us to make new type of transactions mentioned before and add more when it will be needed


## Updating wallet API
### Download new binaries of the wallet API

### Enable CA support

* run new binary with `--enable_assets`. With this flag your wallet starts to accept transactions with Confidential Assets 

    ```
        ./wallet-api --enable_assets -n <node address>
    ```

* updated the codebase. Methods `get_utxo`, `tx_list` and `wallet_status` now has `"assets"` parameter, if you pass `"assets": true` these methods return data related to CA. If you wish to send or split CA you should specify `"asset_id"` parameter. Also there are some new methods which provide CA specific functionality (`get_asset_info`, `tx_asset_info`, `tx_asset_issue`, `tx_asset_consume`). Please, see [wallet API documentation](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API) for more details.
* add `"assets": true` parameter to `get_utxo` method to view assets 

    `-->`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 6,
        "method":"get_utxo",
        "params" :
        {
            "count": 10,
            "skip": 0,
            "assets": true,           // <------------ report assets
            "sort": {
                "field": "amount",
                "direction": "asc"
            },
            "filter": {
                "asset_id": 1         // <------------ filter assets with ID 1
            }
        }
    }
    ```
* add `"assets": true` parameter to `tx_list` method, to view report transaction list including transactions with assets

    `-->`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 8,
        "method":"tx_list",
        "params":
        {
            "filter" :             // <--------------- you can filter buy asset_id here
            {
                "status":4,
                "height":1055,
            },
            "skip" : 0,
            "count" : 10,
            "assets": true         // <--------------- report assets too
        }
    }
    ```
  
* to retrieve info about assets in your wallet pass `"assets": true` parameter to `wallet_status` method, to get status of the wallet and its balance in different CAs and BEAM

    `-->`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 6,
        "method":"wallet_status",
            "params": {
                  "assets" : true
            }
    }
    ```

    `<--`
    ```json
    {
      "id": 1236,
      "jsonrpc": "2.0",
      "result": {
        "current_height": 112,
        "current_state_hash": "b9e8b868de60f28e553a1499a569f481991e4cff9fe2191d09d71a03c7708296",
        "difficulty": 378.36236572265625,
        "prev_state_hash": "3f84da0b0390deaca908603b6061867def987575a1af9311248ffb01503a0f02",
        "totals": [
          {
            "asset_id": 0,
            "available": 303000000000,
            "available_str": "303000000000",
            "maturing": 8000000000,
            "maturing_str": "8000000000",
            "receiving": 0,
            "receiving_str": "0",
            "sending": 0,
            "sending_str": "0"
          },
          {
            "asset_id": 1,                    // <--------------- this could be used to send/split CA or to retrieve extended info
            "available": 2000000000,
            "available_str": "2000000000",
            "maturing": 0,
            "maturing_str": "0",
            "receiving": 0,
            "receiving_str": "0",
            "sending": 0,
            "sending_str": "0"
          }
        ]
      }
    }
    ```

* if you want to send or split CA you should specify `"asset_id"` in parameters

    `-->`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 2,
        "method":"tx_send", 
        "params":
        {
            "value": 12342342,
            "fee": 2,
            "from": "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67",
            "address": "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67",
            "comment": "thank you!",
            "asset_id": 1                   <------------------ NEW
        }
    }
    ```

* if you want to get info about CA use

    `-->`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 2,
        "method": "tx_asset_info",
        "params": 
        {
            "asset_id": 1
        }
    }
    ```
    `<--`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 2,
        "result":
        {
            "txId" : "10c4b760c842433cb58339a0fafef3db"
        }
    }
    ```
* after this transaction become completed, you can read CA info from the local database

    `-->`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 6,
        "method": "get_asset_info",
        "params" :
        {
            "asset_id": 1
        }
    }
    ```

    * `asset_id` asset id to retrieve info about. Can be used for any asset even if you don't own it.

    `<--`
    ```json
    {
        "id": 1236,
        "jsonrpc": "2.0",
        "result": 
        {
            "asset_id": 1,
            "emission": 2000000000,
            "emission_str": "2000000000",
            "isOwned": 1,
            "lockHeight": 39,
            "metadata": "STD:N=NAME;SN=SNM;UN=UNIT;NTHUN=NTHUNIT",
            "ownerId": "0ae08a49e018e98177774294107dc033790b87538e54a20e99c6b98f1dbd39ce",
            "refreshHeight": 927
        }
    }
    ```
    Returns full asset info or [error code](#Errors).

    * `asset_id` asset id
    * `metadata` asset metadata
    * `emission` & `emission_str` total asset emission. Maximum asset emission is 2<sup>128</sup>-1. To ensure compatibility with JavaScript raw number returned only if it is less than or equal to `Number.MAX_SAFE_INTEGER` (2<sup>53</sup>-1). If asset emission is greater than `Number.MAX_SAFE_INTEGER` only corresponding string representation is returned.
    * `isOwned` is 1 if you own this asset
    * `lockHeight` last block when asset emission turned to/from 0.
    * `refreshHeight` block at which asset information has been received. Please note, that all returned fields are valid only for this and previous blocks. In next blocks emission might change, asset become unregistered &c. Use [tx_asset_info](#tx_asset_info) to retrieve the most recent info.


### Enable new types of transaction
* run new binary with `--enable_lelantus`. With this flag your wallet is able to send UTXO via shielded pool.

    ```
        ./wallet-api --enable_lelantus -n <node address>
    ```

### Newer address type support
* Update address verification. Now, `validate_address` method return additional info about address:

    `<--`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 1,
        "result" : 
        {
            "is_valid" : true,
            "is_mine" : false,
            "type": "offline",  // <---------- NEW
            "payments": 3       // <---------- NEW
        }
    }
    ```

    * `type` the type of address, possible values are
        * `regular` - regular old-style `hex` encoded address, default
      * `offline` - new-style `base58` encoded address for _offline_ transaction, could be used several times, it has no strong guaranties for privacy
      * `max_privacy` - new-style `base58` encoded address for _maximum privacy_ transaction, could be used once
      * `public_offline` - new-style `base58` encoded address for _offline_ transaction, it's constant, it provided less privacy, you could publish it, designed for donations
      * `regular_new` - new-style `base58` encoded address for regular mimblewimble transaction

    * `payments` - the number of payments left in case of `offline` address

If you don't want to support newer type of transactions you could simply allow `regular` addresses.


* regex for the newer address is:
    ```
    /[0-9a-zA-Z]{1,1000}/
    ```
it allows both old-style and new-style addresses.
* in order to generate new-style address pass `"type"` parameter to `create_address` method

    `-->`
    ```json
    {
        "jsonrpc" : "2.0", 
        "id" : 1,
        "method" : "create_address", 
        "params" :
        {
            "type": "max_privacy",  // <----- NEW
            "expiration" : "auto",
            "comment" : "John Smith"
        }
    }
    ```
The values for `"type"` parameter are the same as described earlier. If you don't specify `"type"` `"regular"` is used by default to keep backward compatibility.




   
 