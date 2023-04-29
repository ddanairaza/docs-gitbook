# This documentation is obsolete

Please refer to [Beam Node User Guide](https://beam-docs.readthedocs.io/en/latest/rtd_pages/user_beam_node_guide.html)

# Running a node

* [Node source code](https://github.com/BeamMW/beam/tree/master/beam)
* Node binaries for Windows, Mac and Linux can be downloaded from the Beam website (http://beam.mw/downloads) or GitHub (https://github.com/BeamMW/beam/releases).
* Node parameters can be either passed through command line or specified in the configuration file called `beam-node.cfg` and located in the same folder the node resides in.
* For every parameter, the value in the command line prevails over configuration file.
* The log files are located in the `logs` folder, which in turn resides in the node folder.
* Once started, the node will create a `node.db` file in the same folder it is located. This file will store an internal state of the node.
* Upon the first launch, the node will download current blockchain history in batch mode as a single large macroblock. After the initial sync is complete, the node will continue to sync blocks and individual transactions from the current blockchain Tip (height) and onwards. This can be seen in the log entry: `My Tip: 24704-f2ab414ba6430d85`. 

# Command line execution example:

``` sh
./beam-node --peer 104.248.159.154:8100 --mining_threads 1 --file_log_level debug --key_mine=Ipte/cVRHvS72U4h66KIDDILwtIhpsRugTWycwnDUqwOqFq+qmcVEuWhjPJ2OwBn6ZkxKKpLX0W9PvHhGIUVPz8d6CL5CSB8fjt4kA== --key_view=xj7uVZh7kWaY2U4h66KIDDILwtIhpsRugTWycwnDUqwOqFq+qmcVEuVqiydGxomYaF2lQcc92Rzm3HBqsvk9LrFZlrksacvpDgteNLqxQJ4DUD+iKiDYvXy+VLCui125Rw69lO+8gxnxMM5j1rk= --pass=123
```

## Node command line parameters explained:
| Name | Description |
|------|-------------|
| `peer` | Specifies a comma-separated list of peers the node will initially connect to. After the connection is established, the node will get an updated list of peers from other nodes, along with peer ratings and from that moment the node will manage its connections on its own. |
| `key_mine` | A password-protected secret key, exported by the wallet, that should be used for standalone mining (when the owner wallet is offline). If not specified - mining would be possible only when the owner wallet is online |
| `key_view` | A semicolon-separated list of password-protected viewer keys, exported by the wallet, that should be monitored by the node. The wallet master viewer key is also needed for node-wallet authentication. The `key_mine`, if set, is also included implicitly |
| `pass` | A password that should be used to access they encoded keys. |
| `mining_treads` | Specifies the number of CPU cores utilized for mining. If set to 0, node acts as a validating node only. |
| `file_log_level` | Allows to raise the debug level when a deeper investigation is required. |

# Config file example:

``` sh
# port to start server on
port=10000

# secret key for mining.
key_mine=Ipte/cVRHvS72U4h66KIDDILwtIhpsRugTWycwnDUqwOqFq+qmcVEuWhjPJ2OwBn6ZkxKKpLX0W9PvHhGIUVPz8d6CL5CSB8fjt4kA==

# All the viewer keys.
--key_view=xj7uVZh7kWaY2U4h66KIDDILwtIhpsRugTWycwnDUqwOqFq+qmcVEuVqiydGxomYaF2lQcc92Rzm3HBqsvk9LrFZlrksacvpDgteNLqxQJ4DUD+iKiDYvXy+VLCui125Rw69lO+8gxnxMM5j1rk=

# Password. If required (at least 1 key specified) and not set - will be asked after launch.
--pass=123

# number of mining threads(there is no mining if 0)
mining_threads=1

# peers
peer=104.248.159.154:8100

# miner type
miner_type=cpu
```

# The full list of options supported by the node

## General options:

| Name | Description |
|------|-------------|
| `h` or `help`| list of all options |
| `p arg` or `port arg (=10000)` | port to start the server on |
| `log_level arg` | log level `[info/debug/verbose]` |
| `file_log_level arg` | file log level `[info/debug/verbose]` |
| `v` or `version` | return project version |
| `git_commit_hash` | return commit hash |

## Node options:
| Name | Description |
|------|-------------|
| `storage arg (=node.db)` | node storage path |
| `history_dir arg (=./)` | directory for compressed history |
| `temp_dir arg (=/tmp/)` | temp directory for compressed history, must be on the same volume |
| `miner_type arg (=cpu)` | miner type [cpu/gpu] |
| `mining_threads arg (=0)` | number of mining threads(there is no mining if 0) |
| `verification_threads arg (=-1)` | number of threads for cryptographic verifications (0 = single thread, -1 = auto) |
| `miner_id arg (=0)` | seed for miner nonce generation |
| `peer arg` | nodes to connect to |
| `import arg (=0)` | specify the blockchain height to import, the compressed history is assumed to be downloaded to the specified directory |

## Rules configuration:

| Name | Description |
|------|-------------|
| `CoinbaseEmission arg (=80000000)` | coinbase emission in a single block |
| `MaturityCoinbase arg (=60)` | Number of blocks before coinbase UTXO can be spent |
| `MaturityStd arg (=0)` | Number of blocks before non-coinbase UTXO can be spent |
| `MaxBodySize arg (=1048576)` | Max block body size in `[bytes]` |
| `DesiredRate_s arg (=60)` | Desired rate of generated blocks in `[seconds]` |
| `DifficultyReviewCycle arg (=1440)` | Number of blocks after which the mining difficulty can be adjusted |
| `MaxDifficultyChange arg (=2)` |  Max difficulty change after each cycle (each step is roughly x2 complexity) |
| `TimestampAheadThreshold_s arg (=7200)` | Block timestamp tolerance in `[seconds]` |
| `WindowForMedian arg (=25)` | How many blocks are considered in calculating the timestamp median |
| `AllowPublicUtxos arg (=0)` | Set to allow regular (non-coinbase) UTXO to have non-confidential signature |
| `FakePoW arg (=0)`| Don't verify PoW. Mining is simulated by the timer. For tests only |

# Running Beam Node with Stratum Server

* Beam node implements Stratum protocol for connecting external miner clients. Clients open a TCP connection to the node though which they receive jobs to mine blocks using Equihash mining protocol.

# Command line execution example

``` sh
./beam-node --port {NODE_PORT} --peer {PEER_IP} --stratum_port {STRATUM_PORT} --stratum_secrets_path {DIRECTORY} ----key_view=xj7uVZh7kWaY2U4h66KIDDILwtIhpsRugTWycwnDUqwOqFq+qmcVEuVqiydGxomYaF2lQcc92Rzm3HBqsvk9LrFZlrksacvpDgteNLqxQJ4DUD+"
```

## Command line parameters

| Name | Description |
|------|-------------|
| `port` | port to start the node server on |
| `peer` | nodes to connect to (in example remote node from masternet) |
| `stratum_port` | stratum server port (should be >0) |
| `stratum_secrets_path` | folder where configuration files are located |

## Configuration files

| Name | Description |
|------|-------------|
| `stratum.crt` | TLS certificate |
| `stratum.key` | private key for TLS |
| `stratum.api.keys` | file with allowed api keys |

For testing purposes ONLY `stratum.crt` and `stratum.key` can be downloaded from:

* https://github.com/BeamMW/beam/blob/master/utility/unittest/test.crt
* https://github.com/BeamMW/beam/blob/master/utility/unittest/test.key

The stratum.api.keys file (which you create by yourself) should contain any number of strings at least 7 symbols long without spaces each:

```
12345678
sfdskjhfdksk
984398349834
```

If there is no `stratum.api.keys`, ACL will be switched off.

Please note that `stratum.api.key` file is reloaded by the server every 5 seconds, if you want to add/delete/edit  key you shouldn't restart `beam-node`, just edit file `stratum.api.keys` file.

# Known limitations
* CPU without SSE3 instruction set are not supported
