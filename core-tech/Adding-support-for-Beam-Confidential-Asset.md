# Adding support for Beam Confidential Asset

## TLDR;

1. In version 6.0 Beam supports Confidential Assets
1. Each asset has an `asset_id`, 0 is for Beam and 1,2,3... for new Assets.
1. A complete list of existing assets is [here](https://explorer.beam.mw/assets)
1. To enable assets add the following line to the `wallet-api` config file

    ```
    enable_assets=true
    ```
1. The following API calls have been updated to support assets:
    * `wallet_status`
    * `tx_send`
    * `tx_split`
1. New API methods have been added
    * `tx_asset_issue`
    * `tx_asset_consume`
    * `tx_asset_info`
    * `get_asset_info`
    * `calc_change`

## Confidential Assets

Confidential Assets (CA) are tokens mint on the Beam blockchain. Beam supports Confidential Assets since hard fork 2, but this feature had limited ability to be used in real life applications. Starting from the version 6.0 Beam Wallet adds ability to create smart contracts which make CA more usable and very important feature in Beam infrastructure. We already have [API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API#assets-support) to work with CA, but it was disabled by default. If you wish to accept and create transactions with CA you should enable this feature in the wallet API and update your codebase to handle CA correctly. 

### Enable CA support

* run new binary with `--enable_assets`. With this flag your wallet starts to accept transactions with Confidential Assets 

    ```
    ./wallet-api --enable_assets -n <node address>
    ```
   or specify it in config file
   ```
   enable_assets=true
   ```
* to retrieve info about assets use `wallet_status` method, it will return an array of the info about assets in the wallet, including BEAM

    `-->`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 6,
        "method":"wallet_status"
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
        "available": 303000000000,
        "receiving": 123,
        "sending": 0,
        "maturing": 8000000000,
        "locked": 30,
        "totals": [
          {
            "asset_id": 0,
            "available": 303000000000,
            "available_str": "303000000000",
            "maturing": 8000000000,
            "maturing_str": "8000000000",
            "receiving": 123,
            "receiving_str": "123",
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

* For minting new asset coins use `tx_asset_issue`. You must own the asset and info about the asset should be in a local database. `tx_asset_info` to retrieve the latest asset info if necessary. Asset minting is free. You need to pay only regular transaction fee.

	`-->`
	```json
	{
	    "jsonrpc": "2.0", 
	    "id": 2,
	    "method": "tx_asset_issue", 
	    "params":
	    {
	        "value": 6,
	        "asset_id": 1
	    }
	}
	```

	* `value` how much asset to mint, in asset groth.
	* `fee` transaction fee in BEAM groth. Omit to use default fee.
	* `asset_id` asset id of the asset to mint. 
	* `txId` optional,  provide your own transaction ID.

	`<--`
	```json
	{
	    "jsonrpc": "2.0", 
	    "id": 2,
	    "result":
	    {
	        "txId" : "10c4b760c842433cb58339a0fafef3db"
	    }
	}
	```
	Returns transaction id or [error code](#Errors). 

* to burn existing asset coins use `tx_asset_consume`. You must own the asset itself as well as asset coins to burn them. You cannot burn asset coins that belong to another wallet. Info about the asset should be in a local database. 

	`-->`
	```json
	{
	    "jsonrpc":"2.0", 
	    "id": 2,
	    "method":"tx_asset_consume", 
	    "params":
	    {
	        "value": 6,
	        "asset_id": 1
	    }
	}
	```

	* `value` how much asset to burn, in asset groth.
	* `fee` transaction fee in BEAM groth, omit for a default fee.
	* `asset_id` id of the asset to consume. 
	* `txId` optional, provide your own transaction ID


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
        Returns transaction id or [error code](#Errors). 

* if yoiu want to calculate the change amount for transaction use `calc_change`, it calculates the change value for given `amount` in for given `asset_id`

	`-->`
	```json
	{
	    "jsonrpc":"2.0", 
	    "id": 4,
	    "method":"calc_change", 
	    "params":
	    {
	        "amount" : 1234,
	        "asset_id": 2,
	        "fee": 10000,
	        "is_push_transaction": true
	    }
	}
	```

	`<--`
	```json
	{
	    "jsonrpc":"2.0", 
	    "id": 4,
	    "result":
	    {
	        "asset_change": 12,
	        "asset_change_str": "12",
	        "change": 12,
	        "change_str": "12",
	        "explicit_fee": 1100000,
	        "explicit_fee_str": "1100000"
	    }
	}
	```

	where

	* `amount` is a requested amount we are going to send
	* `asset_id` asset id of the requested amount
	* `fee` explicit fee in GROTHs chosen by the user
	* `asset_change` change amount for requested `asset_id`
	* `change` change in for BEAM. `asset_change` and `change` are equal if `asset_id == 0`, i.e. BEAM
	* `explicit_fee` the fee which should be used
	* `is_push_transaction` `true` if we are going to push transaction output into the shielded pool.