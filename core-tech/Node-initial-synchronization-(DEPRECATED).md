# -- DEPRECATED -- 

BEAM Node initially operates in **Sync** mode, then eventually it switches to the **Standard** mode.

Initial state selection is performed according to the following logic:
* If the current blockchain tip is non-empty, i.e. contains at least the genesis block **Standard** mode is selected.
* If there is a _Treasury_ block(s) defined - the Node is supposed to create the genesis block. It switches to the **Standard** mode, mines the genesis block(s), and eventually broadcasts it to the network.
* Otherwise **Sync** mode is activated.

The goal of the **Sync** mode is to allow the fast-Sync via _Macroblock_ whenever possible. It consists of three phases.
1. _Detection phase_. Node analyzes its peers, and selects the most suitable _Macroblock_ to download.
1. _Download phase_. Node downloads the _Macroblock_ (by portions), from one or several peers.
1. _Import phase_. Once fully downloaded - the Node uses it to import, and then switches to the **Standard** mode.

### Synchronization via Macroblock

The _Macroblock_ is a "compressed" blockchain history, which includes the following:
1. All the block headers, with PoW.
2. All the transaction kernels.
3. The unspent UTXO set.

In contrast to the original blocks the macroblock doesn't contain any info about spent UTXOs. Once downloaded, the Node verifies the following:
1. All the headers are valid (sane, have valid PoW, form a valid blockchain)
2. All the original kernels are included in the macroblock
   * This is done by verifying groups of kernels corresponding to a specific block w.r.t. kernel commitment in the appropriate header.
   * This proves that all the original transactions are included.
3. The resulting state of the system is valid:
   * Unspent UTXOs with all the kernels form a valid transformation from the genesis to the asserted height.
   * The overall value equals what it should be according to the emission schedule.
   * Recent coinbase UTXOs are time-locked, according to the system rules.

## Detection phase

For each peer with non-empty tip the Node requests the following:
* _Chainwork proof_ (to ensure the credibility of the reported tip)
* Information about the most recently generated _Macroblock_ (its State ID).

This continues until either enough such peers are examined, or the timeout expires (since the first suitable peer was found). The macroblock of the peer with the highest _Chainwork_ is selected.
If there is no macroblock to download (which is typically the case until blockchain height is low) - the Node switches to the **Standard** mode, otherwise the _Download phase_ is activated.

## Download phase

During this phase the _Macroblock_ is downloaded by portions. Each time it gets a portion - the next portion is requested from the same peer. In case the peer goes down or has no macroblock for upload - another peer is selected.

In case of shutdown/restart - the Node will continue the download from where it stopped.

## Import phase

The Node performs the import from the downloaded _Macroblock_, and then switches to the **Standard** mode.

# Notes

In case something goes wrong during the **Sync** mode - there is no automatic retry/restart. User intervention is required. Whereas in typical scenarios problems are unlikely, possible abnormal cases are:

* No Macroblock detected. Can happen if the peer with the highest (proven) chainwork reports than it has no _Macroblock_. In this case Node goes straight to the **Standard** mode to sync, which is slower, and may not succeed in case old original blocks are already deleted in all peers.

* Macroblock download stuck: Node attempts to download a _Macroblock_ which no more exists in any of its peers. Can happen if the download takes insanely long, and during this time all the peers generated many newer _Macroblocks_. Typically each node keeps several most recently-generated _Macroblocks_ (7 by default), and eventually deletes older ones.

* Macroblock import error (data corruped, inconsistent state). Most probably the result of DoS attack,  or less probably - some bug or the storage corruption.

In case of an error it's possible to reset the Node state, and retry the synchronization. Or, alternatively, the most recent _Macroblock_ may be downloaded manually, and given to the Node for explicit initialization
