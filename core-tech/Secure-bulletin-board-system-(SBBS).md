# Secure bulletin board system (SBBS)
## Overview
The main goal of BBS is to allow wallets to communicate with each other in a secure and asynchronous manner. Using BBS wallets allows individuals to exchange messages, even if one of the individuals is offline. In general, BBS is a virtual board, where users can place messages, and each message is encrypted. For encryption, the public key of the recipient is used. This implies that the recipient’s public key is his address in terms of this system. Every participant who is interested in messages from this board, observes and tries to decrypt new messages with his private key, and he manages to do so only if the message has been addressed to him. It consists of server and client sides. The server is implemented as a part of the node. The client is a wallet.

## Server side
BBS is a part of our node, This implementation implies that messages are sent to n=32 channels, which can ultimately change. The channel is calculated from the public key (first 5 bits). A client has to subscribe to a specific channel, and the server will retranslate all of the new messages to subscribers. Also, the server receives and stores within the database all new messages and exchanges these messages with other servers using a P2P protocol. This logic is implemented in class Node::Peer. It represents the node’s peer and handles the node’s P2P message. BBS messages are a part of node P2P protocol.

#### BbsSubscribe
Message `tp` allows clients to subscribe/unsubscribe to notifications from the node about new messages.

* `BbsChannel m_Channel` – channel to listen.
* `Timestamp m_TimeFrom` – timestamp used to avoid sending of out-of-date messages.
* `bool m_On` – subscribe/unsubscribe flag.

When the server receives this message from the peer with `m_On == true` and `Node::Peer::m_Subscriptions` list doesn’t have it (the look up is made by channel), then the node adds a new instance of `Node::Bbs::Subscription` to this list and to `Node::m_Bbs::m_Subscribed`. If the node has messages for this channel that are sent after given timestamp, then it sends them as a new `BbsMsg` to this subscriber. If `m_On == false` and a given subscriber exists in `m_Subscriptions` list, the node simply removes it.

#### BbsMsg
A message which is used to put an encrypted message onto the board or to exchange between nodes:

* `BbsChannel m_Channel` – target channel.
* `Timestamp m_TimePosted` – timestamp, when wallet has posted message to bbs.
* `ByteBuffer m_Message` – the encrypted content.

When this message is received, server gets current timestamp. If `m_TimePosted` <= currentTime-24 hours, i.e. to old, or `m_TimePosted` > currentTime + 2 hour (someone generates messages from future), then node rejects this message. If it doesn’t have this message it stores it in database, sends `BbsHaveMsg`, to inform other peers about new bbs message, and sends `BbsMsg `to subscriberd interested in messages from `m_Channel`.

When this message is received, the server gets a current timestamp. If `m_TimePosted <= currentTime-24` hours, i.e. too old, or `m_TimePosted > currentTime + 2` hours (someone generates messages from the future), then the node rejects this message. If it doesn’t have this message, it is stored in the database, and sent `BbsHaveMsg`, in order to inform other peers about new bbs message. It sends `BbsMsg` to subscribed interested in messages from `m_Channel`.

#### BbsHaveMsg
A Message which is used by BBS server (node) to exchange information about message that it has.
* `BbsMsgID m_Key` – the message’s key. It is defined as a hash of the `m_Message` and the `m_Channel` in `BbsMsg`. In order to handle this message, the node checks if it already has this message. If it doesn’t, it stores it into waitlist `Node::m_Bbs::m_W` and sends `BbsGetMsg`.

#### BbsGetMsg
A peer server request for bbs message, which occurs as a response to `BbsHaveMsg`.
* `BbsMsgID m_Key` If the node has a bbs message with a given key, it sends it back as `BbsMsg`.

#### BbsPickChannel
The Client asks the server for recommended channels.

#### BbsPickChannelRes
Server channel recommendation
* `BbsChannel m_Channel` - recommended channel. Once in an hour `(Node::m_Cfg::m_Timeout::m_BbsCleanupPeriod_ms)` the node tries to find a channel which is populated with less than 100 listeners `(Node::m_Cfg::m_BbsIdealChannelPopulation)`. Found value is stored into `m_RecommendedChannel` and it is used as a response to `BbsPickChannel`.

## Client side
In the wallet, the BBS communication is placed in WalletNetworkViaBbs class. It allows to send a message to BBS and it manages BBS keys and timestamps. This class contains references to:
* `IWallet& m_Wallet` - which is used for callbacks to wallet object.
* `proto::FlyClient::INetwork& m_NodeNetwork` - which is used to communicate with the node.

When the wallet wants to create a new address (from cli, ui) or load already-created addresses from the database, `AddOwnAddress()` is called. This method calls `IWalletDB::calcKey()` with the key type `Bbs` to generate a private key for BBS. The public key is created using `proto::Sk2Pk()`. Index, which is used for generating a private key that is stored in `wallet.db`. The wallet will then choose a channel. If the client has not subscribed to the chosen channel, the `BbsSubscribe` message is posted to the node via `m_NodeNetwork`. If the wallet wants to send a message via Bbs it calls the overridden method `Send()`. In this method, the given message is encrypted using `proto::BbsEncrypt()` and sent to `m_NodeNetwork`. If the wallet received the bbs message it updates the timestamp for the channel and stores them to the database.

#### Usage of BBS to exchange message

There are two sides wallet **A** and wallet **B**.

##### **A** and **B**:

* picks or generates a pair of keys public and private
* choose channel//asks BBS server (node) for suitable
* sends `BbsSubscribe` to chosen channel, if needed

##### Wallet **A**:

* chooses the address of target recipient (his public key)
* creates a message, it has to contain its public key as an address for answer
* encrypts this message using pub
* creates `BbsMsg`, fills it and sends to BBS server

##### Wallet **B**:

* receives `BbsMsg`
* updates timestamps for the message channel
* tries to decrypt bbs message using the known key for message channel
* if succeeded, it notifies wallet
