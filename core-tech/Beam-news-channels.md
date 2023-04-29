Beam able to provide actual news and exchange rates to wallet users.
This implemented using signed messages of different types transmitted over the Bulletin Board System (BBS).
Such messages make possible for instance to notify wallet user with popup on new wallet application version release.
Each message is broadcasted over the network to wallet applications.
Broadcasted messages has to be signed with apropriate key to verify publisher. Wallet applications have publisher key to check if messages are valid.

## Broadcaster utility
The utility is used for news message dispatching.
Also utility has ability to generate new random publisher key pair. Private key should be keept in secret and used to sign broadcasted messages. Public key has to be embedded to the wallet application and can be shared to anyone who want to listen broadcast messages.

## Commands
`--command` - option used to specify command:

`generate_keys` - used to generate new random publisher key pair

`transmit` - used to send broadcast message

Option `--command` can be omitted. In such case `transmit` is considered as default action.

## Transmit command
The utility needs the address of a node (to serve as an entry point for the message) and the publisher private key specified in hex-format (to sign the message) always to be specified.

### Mandatory options
`-n` - address of node

`--key` - 64 characters long private key in HEX format

`--msg_type` - message type (`'update'`, `'exchange'`)

_Example:_ `-n "eu-node02.masternet.beam.mw:8100" --key "f70c36f2d8342b66e3081ea4d87543566d6ad242c6e61dbf926d57ff42de0c59" --msg_type 'exchange'`

Only messages signed with valid publisher keys are processed by wallet applications.

### Options for specific message types
Specific options have to be passed for each message type.

**In case of the message notifying about new wallet application release:**

`--msg_type 'update'`

`--upd_ver` - new application version in format [x.y.z]

`--upd_type` - released application type ('desktop', 'android' or 'ios')

_Example:_ `--msg_type 'update' --upd_ver '1.8.9' --upd_type 'desktop'` will say new desktop wallet release v1.8.9 is available

**In case of message distributing some currency exchange rate:**

`--msg_type 'exchange'`

`--exch_curr` - currency name ('beam', 'btc', 'ltc', 'qtum')

`--exch_rate` - exchange rate value in fixed point format (100,000,000 = 1 unit)

`--exch_unit` - second currency name which states unit of exchange rate value ('btc', 'usd' - default)

_Example 1:_ `--msg_type 'exchange' --exch_curr 'beam' --exch_rate '789654123'` will say 1 Beam = 7.89654123 USD

_Example 2:_ `--msg_type 'exchange' --exch_curr 'beam' --exch_rate '123456789' --exch_unit 'btc'` will say 1 Beam = 1.23456789 BTC

_Common example:_
`./broadcaster.exe -n "eu-node02.masternet.beam.mw:8100" --key "f70c36f2d8342b66e3081ea4d87543566d6ad242c6e61dbf926d57ff42de0c59" --msg_type 'update' --upd_ver '1.8.9' --upd_type 'desktop'`
