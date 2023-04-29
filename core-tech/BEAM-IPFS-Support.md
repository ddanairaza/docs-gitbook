**THIS DOCUMENT DESCRIBES UPCOMING AND NOT YET RELEASED v6.3 IPFS SUPPORT. Subject to change without prior notice.**

As of v6.3 BEAM adds IPFS support and IPFS can be accessed via API. Refer [version 6.3 API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API-v6.3) docs for details on supported IPFS calls. 

* in Desktop UI client IPFS is enabled by default. IPFS node is started when user launches any DApp granting full IPFS support to DApps. After IPFS node is started it continues to run until desktop client is closed.

* in wallet-api IPFS support is disabled by default, `--enable_ipfs=true` option should be specified to start IPFS node granting full IPFS support

* in Mobile clients & WASM client limited IPFS support is enabled. No local IPFS node is started, write-only methods like ipfs_add/ipfs_pin would fail. Read-only methods like ipfs_get are enabled via HTTP calls to BEAM managed IPFS nodes.

## BEAM IPFS Config

BEAM IPFS repository is located in the `[wallet data folder]/ipfs-repo` until `--ipfs_repo=[path]` option is specified for API CLI or `[ipfsnode] ipfs_repo=[path]` in Desktop `settings.ini` file. When there is no initialized IPFS repository in the given path it would be automatically initialized with the default 'server' IPFS profile for API CLI and with the default (client) profile for Desktop Client except the following custom BEAM settings:

* `Swarm.ConnMgr.LowWater` - is set to `100`. Can be changed using `--ipfs_low_water` option in API CLI and using `[ipfsnode] ipfs_low_water=` in Desktop client 'settings.ini' file.

* `Swarm.ConnMgr.HighWater` - `200` / `--ipfs_high_water`, `settings.ini -> [ipfsnode] ipfs_high_water=`

* Swarm.ConnMgr.GracePeriod - `20s` / `--ipfs_grace_period`, `settings.ini -> [ipfsnode] ipfs_grace_period=` as uint32 seconds

* Bootstrap - default BEAM bootstrap node(s) / `--ipfs_bootstrap` (space separated [multiaddr](https://github.com/multiformats/multiaddr) list), `settings.ini -> [ipfsnode] ipfs_bootstrap=`

* Addresses.Swarm - custom `10100` listening port / `--ipfs_swarm_port`, `settings.ini -> [ipfsnode] ipfs_swarm_port=`

* Addresses.API - custom `/ip4/127.0.0.1/tcp/6100` listening address for API / empty (API disabled) on Desktop / `--ipfs_api_addr`, `settings.ini -> [ipfsnode] ipfs_api_addr=`

* Addresses.Gateway - custom `/ip4/127.0.0.1/tcp/6200` listening address in API / empty (Gateway disabled) on Desktop / `--ipfs_gateway_addr`, `settings.ini -> [ipfsnode] ipfs_gateway_addr=`

* Swarm.EnableAutoRelay - false in API / `true` on Desktop / `--ipfs_auto_relay`, `settings.ini -> [ipfsnode] ipfs_auto_relay=`

* Swarm.EnableRelayHop - `false` / `--ipfs_relay_hop`, `settings.ini -> [ipfsnode] ipfs_relay_hop=`

* Swarm.Transports.Network.Relay - forced to `true` everywhere. Not adjustable at the moment.

* AutoNAT.ServiceMode - `enabled` / `--ipfs_autonat`, `settings.ini -> [ipfsnode] ipfs_autonat=` as bool

* AutoNAT.Throttle.GlobalLimit - `30` / `--ipfs_autonat_limit`, `settings.ini -> [ipfsnode] ipfs_autonat_limit=`

* AutoNAT.Throttle.PeerLimit - `3` / `--ipfs_autonat_peer_limit`, `settings.ini -> [ipfsnode] ipfs_autonat_peer_limit=`

* Datastore.StorageMax - `20GB` in API CLI / `2GB` in Desktop client / `--ipfs_storage_max`, `settings.ini -> [ipfsnode] ipfs_storage_max=`

* Routing.Type - `dhtserver` in API CLI / `dht` in Desktop client / `--ipfs_routing_type`, `settings.ini -> [ipfsnode] ipfs_routing_type=`

* swarm.key file is created to ensure connection to the BEAM private IPFS network / `--ipfs_swarm_key`, `settings.ini -> [ipfsnode] ipfs_swarm_key=` as string

* IPFS periodic GC is disabled in API CLI / launched in Desktop Client / `--ipfs_run_gc`, `settings.ini -> [ipfsnode] ipfs_run_gc=` as bool

* sockets-based activation for "io.ipfs.api" and "io.ipfs.gateway" is disabled at the moment and is not planned in the future. Contact us if you need this feature.

* WebUI is not supported and disabled, will be supported in the future

* fuse mounts for "/ipfs" and "/ipfs" are not supported and disabled, will be supported in the future

* remote pinning for MFS roots is not supported and disabled, will be supported in the future

There are no changes in default IPFS repo layout, all default config and data files are and you are able to manage the IPFS repo via default IPFS cli tool paired with the `IPFS_PATH` environment variable. Ensure that BEAM client that uses the repo is not launched when accessing it using IPFS cli. In case of Desktop client's IPFS repo `ipfs_node_api_port`/`Addresses.API` setting should be specified  before running daemon or it would crash. go-ipfs is unable to launch without API support.

### ipfs-repo/config.lock file

This section describes BEAM extension to IPFS repo & config handling.

On every start BEAM would force and overwrite the aforementioned BEAM custom settings in the `ipfs-repo/config` file. If you want to cancel this behavior, edit the `ipfs-repo/config` file manually or via cli and preserve your custom changes `ipfs-repo/config.lock` file should be created.

If `ipfs-repo/config.lock` is present BEAM would not make any changes to the `ipfs-repo/config` file. All BEAM ways to change IPFS settings are immediately blocked and ignored including `--ipfs-xxx` CLI options, any IPFS options set in `wallet_api.cfg` file, any IPFS options set via desktop UI and any IPFS options set in desktop UI `settings.ini` file.

## Useful stuff

### Access BEAM IPFS node using standard IPFS cli tool

1. Start wallet api
2. Install go-ipfs
3. Create beam-ipfs bash script (do not forget to change your paths)

```bash
#/bin/bash
export IPFS_PATH=/home/ubuntu/beam-api/ipfs-repo/
# uncomment the following line if you're using private IPFS network
# export LIBP2P_FORCE_PNET=1
/home/ubuntu/go-ipfs-node/ipfs "$@"
```

In desktop client IPFS node API is disabled by default. `ipfs_node_api_port` should be set in `settings.ini` or `Addresses.API` in `ipfs-repo/config` to access IPFS node API. 

4. Make it executable 

```bash
chmod  +x ./beam_ipfs
```

5. Execute usual ipfs commands via the script

```bash
./beam-ipfs swarm peers
```

### SystemD IPFS unit file

Example below if given for a standard go-ipfs binary. You can also use the same settings for running wallet API

```
/etc/systemd/system/ipfs.service
```

```
[Unit]
Description=GO IPFS Node
After=network.target

[Service]
Type=exec
Restart=on-failure
Environment="IPFS_PATH=/home/ubuntu/go-ipfs-node/repo"
# uncomment if private IPFS network
# Environment="LIBP2P_FORCE_PNET=1"
WorkingDirectory=/home/ubuntu/go-ipfs-node
ExecStart=/home/ubuntu/go-ipfs-node/ipfs daemon

[Install]
WantedBy=multi-user.target
```
