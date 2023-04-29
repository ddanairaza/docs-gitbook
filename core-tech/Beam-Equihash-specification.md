# Beam Equihash specification

## Chosen parameters
`N = 150`
`K = 5`

## Hash generation convention
Since 150 in not a multiple of 8 we pick 19 full bytes and clear lower 2 bits of 18th byte.
We can get only 3 hashes from 512-bit **blake2** output.
Expected blake2 hash output size is `3 * 19 = 57` bytes


## Implementation for CPU
Beam’s Equihash implementation for CPU is based on zcash’s version.
[https://github.com/BeamMW/beam/blob/master/3rdparty/crypto/equihash_impl.cpp](https://github.com/BeamMW/beam/blob/master/3rdparty/crypto/equihash_impl.cpp)