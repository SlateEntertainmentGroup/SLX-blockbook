**How to install**

You can install and run Blockbook on your computer. With Trezor Wallet installed and running as a local instance, you can be completely independent from connecting to SatoshiLabs servers.

To install it on Linux, please use this step-by-step guide:

1. To install Blockbook, you will need to use Linux Debian version 9 (Stretch) or later.

Before you start installing Blockbook, please check the latest blockchain size for your cryptocurrency and be sure to have this amount + approximately 60% - 70% of the size of the blockchain for the blockbook available on your hard drive.

Switch to root privileges before proceeding with the installation.

2. Install docker using this guide:

https://docs.docker.com/install/linux/docker-ce/debian/

3. Clone blockbook git:

         git clone https://github.com/trezor/blockbook

4. Go to blockbook directory and run:

         make all-<coin> (e.g., make all-bitcoin)
  
5. Go to blockbook/build directory and run:

         apt install <package name> (e.g., apt install backend-bitcoin_0.16.1-satoshilabs1_amd64.deb)
  
6. Now you can start synchronizing with the network:

         systemctl start backend-<coin>.service (e.g., systemctl start backend-bitcoin.service)
  
7. If you want to check the status of your synchronizing go to /opt/coins/data/<coin>/backend (eg., /opt/coins/data/bitcoin/backend) and check the status in debug.log file.

8. If the blockchain is fully synchronized, you can start installing your Blockbook. Go to the directory blockbook/build and run:

         apt install <blockbook package> (e.g., apt install blockbook-bitcoin_0.0.6_amd64.deb)
  
9. Run Blockbook:

        systemctl start blockbook-<coin>.service (e.g., systemctl start blockbook-bitcoin.service)
  
10. Blockbook is now synchronizing with your blockchain, you can check the status in /opt/coins/blockbook/<coin>/logs/blockbook.INFO (eg. /opt/coins/blockbook/bitcoin/logs/blockbook.INFO) or by visiting https://localhost:<blockbook public port> (e.g., https://localhost:9130 for bitcoin)

11. After full synchronization, your Blockbook is now running at the localhost port.

12. Now you can connect your Trezor Wallet to your local Blockbook instance by using a custom backend:

Wallet settings - Bitcore Server URL - https://localhost:<blockbook public port> (e.g., https://localhost:9130 for Bitcoin)
ImportantBlockbook uses a self-signed certificate. It is necessary to go to the address in your browser, confirm the certificate, and then add the address to your wallet.

[![Go Report Card](https://goreportcard.com/badge/trezor/blockbook)](https://goreportcard.com/report/trezor/blockbook)

***To disable SSL certificate***

     systemctl edit --full blockbook-bitcoin.service

and remove the --certfile option. And now it is running with no SSL

#########################################################################################################################

**Blockbook** is back-end service for Trezor wallet. Main features of **Blockbook** are:

- index of addresses and address balances of the connected block chain
- fast searches in the indexes
- simple blockchain explorer
- websocket, API and legacy Bitcore Insight compatible socket.io interfaces
- support of multiple coins (Bitcoin and Ethereum type), with easy extensibility for other coins
- scripts for easy creation of debian packages for backend and blockbook

## Build and installation instructions

Officially supported platform is **Debian Linux** and **AMD64** architecture.

Memory and disk requirements for initial synchronization of **Bitcoin mainnet** are around 32 GB RAM and over 180 GB of disk space. After initial synchronization, fully synchronized instance uses about 10 GB RAM.
Other coins should have lower requirements, depending on the size of their block chain. Note that fast SSD disks are highly
recommended.

User installation guide is [here](https://wiki.trezor.io/User_manual:Running_a_local_instance_of_Trezor_Wallet_backend_(Blockbook)).

Developer build guide is [here](/docs/build.md).

Contribution guide is [here](CONTRIBUTING.md).

## Implemented coins

Blockbook currently supports over 30 coins. The Trezor team implemented 

- Bitcoin, Bitcoin Cash, Zcash, Dash, Litecoin, Bitcoin Gold, Ethereum, Ethereum Classic, Dogecoin, Namecoin, Vertcoin, DigiByte, Liquid

the rest of coins were implemented by the community.

Testnets for some coins are also supported, for example:
- Bitcoin Testnet, Bitcoin Cash Testnet, ZCash Testnet, Ethereum Testnet Ropsten

List of all implemented coins is in [the registry of ports](/docs/ports.md).

## Common issues when running Blockbook or implementing additional coins

#### Out of memory when doing initial synchronization

How to reduce memory footprint of the initial sync: 

- disable rocksdb cache by parameter `-dbcache=0`, the default size is 500MB
- run blockbook with parameter `-workers=1`. This disables bulk import mode, which caches a lot of data in memory (not in rocksdb cache). It will run about twice as slowly but especially for smaller blockchains it is no problem at all.

Please add your experience to this [issue](https://github.com/trezor/blockbook/issues/43).

#### Error `internalState: database is in inconsistent state and cannot be used`

Blockbook was killed during the initial import, most commonly by OOM killer. By default, Blockbook performs the initial import in bulk import mode, which for performance reasons does not store all the data immediately to the database. If Blockbook is killed during this phase, the database is left in an inconsistent state. 

See above how to reduce the memory footprint, delete the database files and run the import again. 

Check [this](https://github.com/trezor/blockbook/issues/89) or [this](https://github.com/trezor/blockbook/issues/147) issue for more info.

#### Running on Ubuntu

[This issue](https://github.com/trezor/blockbook/issues/45) discusses how to run Blockbook on Ubuntu. If you have some additional experience with Blockbook on Ubuntu, please add it to [this issue](https://github.com/trezor/blockbook/issues/45).

#### My coin implementation is reporting parse errors when importing blockchain

Your coin's block/transaction data may not be compatible with `BitcoinParser` `ParseBlock`/`ParseTx`, which is used by default. In that case, implement your coin in a similar way we used in case of [zcash](https://github.com/trezor/blockbook/tree/master/bchain/coins/zec) and some other coins. The principle is not to parse the block/transaction data in Blockbook but instead to get parsed transactions as json from the backend.

## Data storage in RocksDB

Blockbook stores data the key-value store RocksDB. Database format is described [here](/docs/rocksdb.md).

## API

Blockbook API is described [here](/docs/api.md).
