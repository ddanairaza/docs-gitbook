Swap API extends [Beam wallet protocol API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API)

requires: configured swap settings in `wallet.db` (by CLI or UI)

## API

API will include the following methods:

- [swap_offers_list](#swap_offers_list)
- [swap_offers_board](#swap_offers_board)
- [swap_create_offer](#swap_create_offer)
- [swap_offer_status](#swap_offer_status)
- [swap_decode_token](#swap_decode_token)
- [swap_publish_offer](#swap_publish_offer)
- [swap_accept_offer](#swap_accept_offer)
- [swap_cancel_offer](#swap_cancel_offer)
- [swap_get_balance](#swap_get_balance)
- [swap_recommended_fee_rate](#swap_recommended_fee_rate)

## swap_offers_list
Get all own swap offers with specified `status` and `swapCoin`. if `filter` not specified return all own swap offers.

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "swap_offers_list",
    "params": {}
}
```
`<--`
```json
{
    "id": 1236,
    "jsonrpc": "2.0",
    "result": [
        {
            "height_expired": 140726,
            "min_height": 140696,
            "receive_amount": 2000000000,
            "receive_currency": "BEAM",
            "send_amount": 100000000,
            "send_currency": "BTC",
            "status": 4,
            "status_string": "expired",
            "time_created": "2020.03.16 17:22:12",
            "txId": "b35fd69030694009b8bf849140d9319e"
        },
        {
            "height_expired": 136300,
            "min_height": 136270,
            "receive_amount": 2000000000,
            "receive_currency": "BEAM",
            "send_amount": 100000000,
            "send_currency": "BTC",
            "status": 4,
            "status_string": "expired",
            "time_created": "2020.03.13 15:16:07",
            "txId": "0d36d9db06f14071b18e1fdf4c429a14"
        },
        {
            "height_expired": 133125,
            "min_height": 133095,
            "receive_amount": 2000000000,
            "receive_currency": "BEAM",
            "send_amount": 100000000,
            "send_currency": "BTC",
            "status": 4,
            "status_string": "expired",
            "time_created": "2020.03.11 10:14:16",
            "txId": "f859fe65bd434522af16cfc7d31c43db"
        },
        {
            "height_expired": 123428,
            "min_height": 123398,
            "receive_amount": 200000000,
            "receive_currency": "BEAM",
            "send_amount": 100000000,
            "send_currency": "BTC",
            "status": 2,
            "status_string": "completed",
            "time_created": "2020.03.04 16:37:44",
            "txId": "d218356770b34fe4aeab01fb12c6074c"
        }
    ]
}
```
`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "swap_offers_list",
    "params": {
        "filter" : {
            "status" : 2
        }
    }
}
```
`<--`
```json
{
    "id": 1236,
    "jsonrpc": "2.0",
    "result": [
        {
            "height_expired": 123428,
            "min_height": 123398,
            "receive_amount": 200000000,
            "receive_currency": "BEAM",
            "send_amount": 100000000,
            "send_currency": "BTC",
            "status": 2,
            "status_string": "completed",
            "time_created": "2020.03.04 16:37:44",
            "txId": "d218356770b34fe4aeab01fb12c6074c"
        }
    ]
}
```

## swap_offers_board
Get all swap offers from offers board with specified `swapCoin`. if `swapCoin` not specified return all known swap offers.


`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "swap_offers_board",
    "params": {}
}
```
`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": [
    {
      "is_my_offer": false,
      "is_public": true,
      "height_expired": 103173,
      "min_height": 103143,
      "receive_amount": 100000000,
      "receive_currency": "BTC",
      "send_amount": 1200000000,
      "send_currency": "BEAM",
      "status": 0,
      "status_string": "pending",
      "time_created": "2020.03.04 16:37:44",
      "token": "6xfTV5NF6JWbcZLupbhTHNEQqnGjHQEL5L5VsT4XhAkMw9GqTzMowAVQVhpUWSAvg8cqtfV1s6BKSa75zk8vCMK7mVP7xNjATJ2pPCNcu3U8UYz4rce4gRuTBuYNDbyY7zFEEzMZL2RKKjGCtEN2bpCif8GuGUekzRc8hiL",
      "txId": "f87b06cf752040cdaefbf99831bc3e80"
    },

  ]
}
```

## swap_create_offer
Creates swap offer that can be published on a board or sent directly to other side

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "swap_create_offer",
    "params": {
        "send_amount": 44000000000,
        "send_currency": "beam",
        "receive_amount": 2000000000,
        "receive_currency": "btc",
        "beam_fee": 100,
        "fee_rate": 90000,
        "offer_expires": 30,
        "comment": "API"
    }
}
```
`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": {
    "txId": "062757f5ea62448589d977aa05c5782f",
    "token": "TJeBeBjF48BdyxTDJg8gjogn9eDKQUx1Nm18iHPw53YoFipxKKkutNUV5AkVnFyLohpSrWsrnFNLGyBakVGY4Y2URdSQ52cDACHGDuHjYVpjMz8KN7q6rJrWGEXah2wJXHxrARYbAh7dTizdtQxaNVj34dfr8wKkfPFC7vmk"
  }
}
```

## swap_offer_status
Shows your offer status or swap transaction in which you are participating

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 126,
    "method": "swap_offer_status",
    "params": {
        "tx_id": "b35fd69030694009b8bf849140d9319e"
    }
}
```
`<--`
```json
{
    "id": 126,
    "jsonrpc": "2.0",
    "result": {
        "height_expired": 140726,
        "min_height": 140696,
        "status": 4,
        "status_string": "expired",
        "time_created": "2020.03.16 17:22:12",
        "tx_id": "b35fd69030694009b8bf849140d9319e"
    }
}
```

## swap_decode_token
Decode swap token

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "swap_decode_token",
    "params": {
        "token" : "6xfNAUemTbmp7KRCRydiGStMZe6oRh59LzS7uk1V4eTrUX1mKcCGY7jdtMtSs4XLt6Ug8jWnepMEZCrqSUw7PeKRDZ8yyVZu1WHXzootpybBjX3nVxxHRSdk4ncBGDh1cssmiJhswZC9PfsaJmRKqXJM3x9tcX7EZn5Vjg8"
    }
}
```
`<--`
```json
{
    "id": 1236,
    "jsonrpc": "2.0",
    "result": {
        "height_expired": 123428,
        "is_my_offer": false,
        "is_public": false,
        "min_height": 123398,
        "receive_amount": 200000000,
        "receive_currency": "BEAM",
        "send_amount": 100000000,
        "send_currency": "BTC",
        "time_created": "2020.03.04 16:37:44",
        "tx_id": "d218356770b34fe4aeab01fb12c6074c"
    }
}
```

## swap_publish_offer
Publicates offer on a board

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 126,
    "method": "swap_publish_offer",
    "params": {
        "token": "TJeBeBjF48BdyxTDJg8gjogn9eDKQUx1Nm18iHPw53YoFipxKKkutNUV5AkVnFyLohpSrWsrnFNLGyBakVGY4Y2URdSQ52cDACHGDuHjYVpjMz8KN7q6rJrWGEXah2wJXHxrARYbAh7dTizdtQxaNVj34dfr8wKkfPFC7vmk"
    }
}
```
`<--`
```json
{
  "id": 126,
  "jsonrpc": "2.0",
  "result": {
    "is_my_offer": true,
    "is_public": true,
    "height_expired": 103173,
    "min_height": 103143
    "receive_amount": 2000000000,
    "receive_currency": "BTC",
    "send_amount": 44000000000,
    "send_currency": "BEAM",
    "status": 0,
    "status_string": "pending",
    "time_created": "2020.02.19 11:29:19",
    "token": "TJeBeBjF48BdyxTDJg8gjogn9eDKQUx1Nm18iHPw53YoFipxKKkutNUV5AkVnFyLohpSrWsrnFNLGyBakVGY4Y2URdSQ52cDACHGDuHjYVpjMz8KN7q6rJrWGEXah2wJXHxrARYbAh7dTizdtQxaNVj34dfr8wKkfPFC7vmk",
    "txId": "062757f5ea62448589d977aa05c5782f"
  }
}
```

## swap_accept_offer
Starts the swap offer published on a board or received directly from other side

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "swap_accept_offer",
    "params": {
        "beam_fee": 100,
        "fee_rate": 90000,
        "comment": "API-accept",
        "token" : "6xfTV5NF6JWbcZLupbhTHNEQqnGjHQEL5L5VsT4XhAkMw9GqTzMowAVQVhpUWSAvg8cqtfV1s6BKSa75zk8vCMK7mVP7xNjATJ2pPCNcu3U8UYz4rce4gRuTBuYNDbyY7zFEEzMZL2RKKjGCtEN2bpCif8GuGUekzRc8hiL"
    }
}
```
`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": {
    "height_expired": 103173,
    "min_height": 103143,
    "receive_amount": 100000000,
    "receive_currency": "BTC",
    "send_amount": 1200000000,
    "send_currency": "BEAM",
    "status": 1,
    "status_string": "in progress",
    "time_created": "2020.02.19 11:20:50",
    "txId": "f87b06cf752040cdaefbf99831bc3e80"
  }
}
```

## swap_cancel_offer
Cancel swap offer (cancels running transaction, return true if successfully canceled or error with the reason)

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 126,
    "method": "swap_cancel_offer",
    "params": {
        "txId" : "a13525181c0d45b0a4c5c1a697c8a7b8"
    }
}
```
`<--`
```json
{
  "id": 126,
  "jsonrpc": "2.0",
  "result": true
}
```

## swap_get_balance

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 126,
    "method": "swap_get_balance",
    "params": {
        "coin": "btc"
    }
}
```
`<--`
```json
{
    "id": 126,
    "jsonrpc": "2.0",
    "result": {
        "available": 129899985060
    }
}
```

## swap_recommended_fee_rate

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 127,
    "method": "swap_recommended_fee_rate",
    "params": {
        "coin": "btc"
    }
}
```
`<--`
```json
{
    "id": 127,
    "jsonrpc": "2.0",
    "result": {
        "feerate": 35123
    }
}
```

### Swap offers statuses description
```
Pending (0)     - waiting for someone accept your offer, or waiting when you accept offer
InProgress (1)  - to indicate that swap transaction in progress, you should stay online
Completed (2)   - a swap transaction is completed
Canceled (3)    - "Cancelled" (by You)
Expired(4)      - offer lifetime expired
Failed (5)      - failed for some reason
```