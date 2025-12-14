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
#Enable testnet
testnet=1

#if you wanna enable mainnet
#mainnet=1
daemon=1

#RPC settings for API access
[test]
rpcuser = createuserhere(laterusedinlnd)
rpcpassword = createpasswordhere(laterusedinlnd)
rpcallowip=127.0.0.1

# HAS TO MATCH THE ONE IN THE LND CONFIG
rpcport=18332

deprecatedrpc=warnings
rpcworkqueue=100
# Connection settings
maxconnections=10
listen=1
zmqpubrawblock=tcp://127.0.0.1:28332

# Notifies of raw, new transactions on port 28333
zmqpubrawtx=tcp://127.0.0.1:28333

# Logging usefull for debugging
debug=1
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
bitcoin-cli -testnet createwallet "YOUR_WALLET_NAME"

# Create encrypted wallet (recommended)
bitcoin-cli -testnet createwallet "YOUR_WALLET_NAME" false false "YOUR_PASSPHRASE"

```
# Part 2: setting up LND node

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
Now do the same thing as with bitcoind, set up a config file: 

```
mkdir ~/.lnd/lnd.conf
nano ~/.lnd/lnd.conf
```
Copy this config, edit it for yourself and save with CTRL + O:

```
[Application Options]
debuglevel=info
maxpendingchannels=5
alias=yournodesalias
color=#3399ff

[Bitcoin]
bitcoin.active=1
bitcoin.testnet=1
bitcoin.node=bitcoind

[Bitcoind]
# you can set up any port you want for the rpchost, but it has to match the on ein bitcoind
bitcoind.rpchost=localhost:18332

bitcoind.rpcuser = youruserthatyousetup
bitcoind.rpcpass = yourpasswordthatyousetup
bitcoind.zmqpubrawblock=tcp://127.0.0.1:28332
bitcoind.zmqpubrawtx=tcp://127.0.0.1:28333

```
Now start up your lightning node fund it (as long as your bitcoind is synced with testnet (mainnet)):

```
lnd
```
Open a separate terminal window and create the lightning node wallet:
!VERY IMPORTANT WARNING BEFORE CONTINUING, IF YOU WORK ON TESTNET, U MIGHT HAVE TO USE THE ``` lncli --network=testnet ``` FOR EVERY COMMAND!

### DISCLAIMER IN THIS STEP IT IS ESSENMTIAL THAT YOU SAVE THE KEY PHRASES TO RESTORE YOUYR WALLET IF EITHER HARDWARE OR SOFTWARE FAILS
```
lncli create
```
Go through the process of setting it up and:

```
lncli unlock
```
and then get the address you will send the SAT onto:

```
# Get new Bitcoin address
lncli newaddress p2wkh
```
Now obviously you have two options either sendind your own bitcoin with mainnet or getting a faucet you can get testnet SAT from.

# Faucet for testnet
I mostly use https://coinfaucet.eu/en/btc-testnet/ its the most rilable source for testnet, however there are many you can find on reddit (surprisingly).

# Part 3: Open Connection
When you have LND set up and your balance isnt 0 (At least 100000 SAT) you can set up a connection with a node and set up an open channel.

## First you have to find a node

For mainnet, I don't usually know any mainnet node websites where you can get them. With testnet https://1ml.com/testnet/ this websites provides you with many different options.

## Get the public key

The website I provided has a public key for every node that you can find on the top of the page. Now set up a connection with them:

```
lncli connect PUBLIC_KEY
```
And then you can open the channel with them (usually you have to put this command immidietly after connecting to the node):
```
lncli --network=testnet openchannel \
    --node_key=026c0f266268c1edc4e1f7e17ba6aa18979fee47f059e46a40cafbbcc56ab3df57 \
    --local_amt=50000 \
    --push_amt=10000
```
50000 SAT is the amount of SAT you want the connection to work with (the transaction in the connection does not go other 50000) and 10000 is the SAT that is basically taken of you as a collateral in case you decide to drop out of the connection (your machine turns off etc etc)

# Part 4: The Good, the bad and the scammy

Congratualtions you have now successfully set up your lightning node on either testnet or mainnet, no the next big part of lightning is the security of it.

## Security
### KEY PHRASE
As stated in previous steps it is ESSENTIAL that you save your key phrase when creating the wallet, beacuse otherwise you might lose your SAT. The SAT is thankfully sealed on-chain, but that doesn't mean you can access it. 

### Static channel backup 
In case of a hardware failure another important issue that might take place is the permanant liquidity loss in the channels you were connected to (one's that still have your funds, fees that you have collected and collateral, which the network might decide you have to pay, beacause of a unactivity for example). 

The channel backups are located at ```~/.lnd/data/chain/bitcoin/channel.backup```, preferably copied and stored somewhere on another device for security.

After creating a new node and wallet to finally close the channel and recieve your funds back you need to use:
```
# This command is often run during the wallet unlock sequence or separately:
lncli restorechanbackup --multi_chan_backup [PATH_TO_BACKUP]
```

The biggest limitation of this however is that all peers on the network must be cooperative and let you take the funds back when clsoing the channel.

## Scamming 
### Watchtowers

An important topic to look at in this case is also the watchtowers. They let you observe the other nodes connected to the channel for potential channel breaches (publishing previous, expired or revoked commitment that may or may not benefit them) 

To run an LND watch tower all you have to do is put an extra ```watchtower.active=1``` above all tags.
To look at the tower info type:
```
lncli tower info
```

