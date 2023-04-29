# Atomic swap token
## Overview
Swap token is intended to allow users to exchange info about transactions(swap) via any side channels.
In general, token is serialized list of transaction parameters, presented as a base58 encoded byte buffer.

## Description
Token has the following fields:

`- flags              // 1 byte`

`- optional TxID      // 1 or 17 bytes`

`- list of parameters // not less than 34 byte since transaction should have at least one address parameter`


### flags
Flags is a 1-byte field where only the highest bit is defined the others are reserved. 
0x80 means that this is a token.


### tx ID
This is an optional transaction ID. All swap transaction should have this value. It is optional for the cases if we will use the token structure for other types of transaction.
There are two possible values:

`|0x01| 16 byte UUID|`

or

`|0x00| `

### Transaction parameters
Transaction parameter is a pair of the parameter key and value. 
The key is 1 byte value and its meaning is defined by `TxParameterID `enum from [wallet/common.h](https://github.com/BeamMW/beam/blob/master/wallet/common.h)
The values are arrays of bytes and contain serialized values.
The value type and size is defined by the key meaning. All integer types are big-endian.






