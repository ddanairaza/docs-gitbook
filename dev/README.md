# Getting started

## Welcome to Beam Developer Portal

<mark style="color:red;">**NOTE: Beam Developer Portal is currently under construction and is subject to changes**</mark>

Beam is a scalable and confidential cryptocurrency and DeFi platform implemented from scratch in C++ programming language. Since its inception in 2018, Beam has evolved a lot and is currently a large and complex project.

In this developer documentation we will provide information about the development of Beam itself  and Smart Contracts (aka Shaders ) on Beam blockchain.

Before we dive in, here is a short list of essential resources you might need in your journey:

* [Beam Source Code](https://github.com/BeamMW)
* [Developer Support Channel on Telegram](https://t.me/beamdevsupport) - for any questions you might have along the way
* [Building Beam](guides/building-beam.md) - covers the process of building all Beam components from source

The documentation will cover the following list of topics:

* [Core Technology Overview](./#core-technology-overview)
* [Confidential Assets](./#confidential-assets)
* [Smart Contracts (aka Shaders)](./#smart-contracts)&#x20;
* [Wallet API (and integration guides)](./#wallet-api)
* Web Wallet&#x20;
* Desktop Wallet
* Node



### Core Technology Overview

&#x20;Initially, Beam started as an implementation of the [Mimblewimble ](beam-technology/mimblewimble.md)([original whitepaper](https://docs.beam.mw/Mimblewimble.pdf)) protocol and, from version 5.0, added support for a [Lelantus](https://github.com/BeamMW/beam/wiki/Lelantus-MW) shielded pool. This hybrid approach was named [LelantusMW](beam-technology/lelantusmw.md), and it is now powering the Beam Privacy layer one.

On the network level, Beam uses Dandelion protocol with non interactive transaction merging along the stem phase. Technical details of the implementation are available [here](https://github.com/BeamMW/beam/wiki/Transaction-graph-obfuscation).&#x20;

One of the unique features of Beam are [Atomic Swaps](beam-technology/atomic-swaps.md). They allow to exchange Beam for Bitcoin, Ethereum and several other currencies without any trusted third party.&#x20;

Another interesting technology is Laser Beam, which is an implementation of direct payment channels, similar to those used in the Lightning Network.&#x20;

Last but not least is the [SBBS ](beam-technology/sbbs.md)- Secure Bulletin Board System - a decentralized encrypted messaging between Beam wallets, used to create interactive transactions and broadcast information.&#x20;

### Confidential Assets

Beam supports an ability to create new types of assets on the blockchain, called Confidential Assets (CA for short).  Unlike Ethereum ERC-20 tokens, Beam CA do not require Smart Contracts due to a Layer 1 implementation that designates each new asset type as a first-class citizen in Beam nodes.&#x20;

This approach has many advantages in both security and privacy. Beam Tokens are **as private as Beam itself** and offer native support with Beam wallets and DAPPs. Confidential Assets can be either emitted manually by a user or controlled by smart contracts within Beam Virtual Machine.

For more information please refer to the dedicated [Confidential Assets](broken-reference) section of this guide.

### Beam Shaders

Starting with version 6.0 Beam has added support for Smart Contracts (a.k.a Beam Shaders) by adding a bespoke Beam Virtual Machine (BVM). BVM natively runs WebAssembly binary code while the contracts themselves can be implemented in any programming language that can be compiled into WebAssembly such as C++, Rust or Go.&#x20;

Beam Shaders, BVM and the related tools are collectively called Confidential DeFi Platform and and described in a [dedicated section](broken-reference) of this guide.



### Wallet API

Beam provides an extensive wallet API to support integration with exchanges, payment services and third party applications. Unlike most other networks that provide API directly from the node, the fact that Beam is a confidential cryptocurrency requires a wallet with secret keys for most operations. Which is why Beam API is based on top of a Beam wallet code.

More information about using Wallet API is described in the [Beam wiki](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API).

### Web Wallet

### Desktop Wallet

### Node

