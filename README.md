## Lightning Network Daemon SET UP GUIDE WITH FOR-PROFIT IMPLEMENTATION 

[![Release build](https://github.com/lightningnetwork/lnd/actions/workflows/release.yaml/badge.svg)](https://github.com/lightningnetwork/lnd/actions/workflows/release.yaml)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/lightningnetwork/lnd/blob/master/LICENSE)
[![Irc](https://img.shields.io/badge/chat-on%20libera-brightgreen.svg)](https://web.libera.chat/#lnd)
[![Godoc](https://godoc.org/github.com/lightningnetwork/lnd?status.svg)](https://godoc.org/github.com/lightningnetwork/lnd)
[![Go Report Card](https://goreportcard.com/badge/github.com/lightningnetwork/lnd)](https://goreportcard.com/report/github.com/lightningnetwork/lnd)

<img src="logo.png">

## This guide thoroughly explains how to set up an LND node based on bitcoind (with links to testnet faucets if needed) 

# Part 1: Install Bitcoind
```
wgt https://bitcoincore.org/bin/bitcoin-core-xx.x/bitcoin-xx.x-x86_64-linux-gnu.tar.gz
tar -xzf bitcoin-xx.x-x86_64-linux-gnu.tar.gz
sudo cp bitcoin-xx.x/bin/* /usr/local/bin/
```
# Essential testnet config
Create a config file for your bitcoind
```
mkdir ~/.bitcoin/bitcoin.conf
nano ~/.bitcoin/bitcoin.conf
```
Put this into the file and press ctrl+O:

```
#Enable testnet modestnet=1

d
#if you wnana just into mainnet immidietlya#mainnet=1emon=1

# R# RPC settings for API accesse[test]crpcuser = createuserhere(laterusedinlnd)crpcpassword = createpasswordhere(laterusedinlnd)callowip=127.0.0.1
rpcport=18332
deprecatedrpc=warnings
rpcworkqueue=100
# Connection settings
maxconnections=10
listen=1
zmqpubrawblock=tcp://127.0.0.1:28332

# Notifies of raw, new transactions on port 28333
zmqpubrawtx=tcp://127.0.0.1:28333

# L# Logging usefull for debuggingbug=1
printtoconsole=1



```
# Start the Bitcoin Core 

```
bitcoin-qt
```
Now you will need to wait until the core syncs up with the the bitcoin testnet, until then no transactions (thus connections with other lightning nodes can be done).
In the mean time you can set up the lnd node (not be able to do anything with it due to the unsynced state) or set up a bitcoin wallet either in the UI or with this command:
```
# Create new wallet
bitcoin-cli -testnet createwallet "testwallet"

# Create encrypted wallet (recommended)
bitcoin-cli -testnet createwallet "testwallet" false false "your_passphrase"

```
### Part 2: setting up LND node
```
# Download LND binary
wget https://github.com/lightningnetwork/lnd/releases/download/v0.17.4-beta/lnd-linux-amd64-v0.17.4-beta.tar.gz

# Extract files
tar -xzf lnd-linux-amd64-v0.17.4-beta.tar.gz

# Move to system path
sudo cp lnd-linux-amd64-v0.17.4-beta/lnd /usr/local/bin/
sudo cp lnd-linux-amd64-v0.17.4-beta/lncli /usr/local/bin/

# Verify installation
lnd --version

```
