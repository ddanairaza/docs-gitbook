Beam Clear Cathode 3.0 version includes a Hard Fork that will be activated in block 321321 (approximately around August 15th, 2019). As part of the fork, the mining algo will be updated to BeamHash II. 

Pools and Solo miners are strongly encouraged to test the version before that time using the latest versions of LolMiner and GMiner that already support BeamHash II.

To test the fork please perform the following steps:

1. Download latest Testnet binaries (or build from source)
2. Follow the instructions on [Local Setup](http://documentation.beam.mw/en/latest/rtd_pages/dev_local_setup.html) to create a stand alone network.
4. Follow the instruction in [Mining Beam](http://documentation.beam.mw/en/latest/rtd_pages/user_mining_beam.html) section, to setup the latest version of the miner. 
3. Use Fork1=`fork height` parameter in the configuration file for both node and wallet, to set a fork height.

IMPORTANT: You have to delete the node.db after the fork happens to repeat the test. Just setting a different fork height would not be enough.
