The following series of articles starts the documentation process for Beam project.

Beam is a Mimblewimble implementation done from scratch in the C++ programming language.

[Key Terms and Concepts](https://github.com/BeamMW/beam/wiki/Programming-Beam-Terms-and-Concepts)

Beam project can be divided into the following logical modules:

1. [P2P and networking](). A lower level implementation of the communication mechanisms. There are several different types of communications: Node2Node, Node2Wallet and Wallet2Wallet. The implementation of the P2P infrastructure can be found [here](https://github.com/BeamMW/beam/tree/master/p2p).

2. [Cryptography and Core elements](). All cryptographic primitives and necessary building blocks are implemented in the [core](https://github.com/BeamMW/beam/tree/master/core) of the project

3. [Beam Node]()

4. [Beam Wallet](https://github.com/BeamMW/beam/wiki/Programming-Beam-Wallet)