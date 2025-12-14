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
# ↓ORIGINAL README↓

## Lightning Network Daemon

[![Release build](https://github.com/lightningnetwork/lnd/actions/workflows/release.yaml/badge.svg)](https://github.com/lightningnetwork/lnd/actions/workflows/release.yaml)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/lightningnetwork/lnd/blob/master/LICENSE)
[![Irc](https://img.shields.io/badge/chat-on%20libera-brightgreen.svg)](https://web.libera.chat/#lnd)
[![Godoc](https://godoc.org/github.com/lightningnetwork/lnd?status.svg)](https://godoc.org/github.com/lightningnetwork/lnd)
[![Go Report Card](https://goreportcard.com/badge/github.com/lightningnetwork/lnd)](https://goreportcard.com/report/github.com/lightningnetwork/lnd)

<img src="logo.png">

The Lightning Network Daemon (`lnd`) - is a complete implementation of a
[Lightning Network](https://lightning.network) node.  `lnd` has several pluggable back-end
chain services including [`btcd`](https://github.com/btcsuite/btcd) (a
full-node), [`bitcoind`](https://github.com/bitcoin/bitcoin), and
[`neutrino`](https://github.com/lightninglabs/neutrino) (a new experimental light client). The project's codebase uses the
[btcsuite](https://github.com/btcsuite/) set of Bitcoin libraries, and also
exports a large set of isolated re-usable Lightning Network related libraries
within it.  In the current state `lnd` is capable of:
* Creating channels.
* Closing channels.
* Completely managing all channel states (including the exceptional ones!).
* Maintaining a fully authenticated+validated channel graph.
* Performing path finding within the network, passively forwarding incoming payments.
* Sending outgoing [onion-encrypted payments](https://github.com/lightningnetwork/lightning-onion)
through the network.
* Updating advertised fee schedules.
* Automatic channel management ([`autopilot`](https://github.com/lightningnetwork/lnd/tree/master/autopilot)).

## Lightning Network Specification Compliance
`lnd` _fully_ conforms to the [Lightning Network specification
(BOLTs)](https://github.com/lightningnetwork/lightning-rfc). BOLT stands for:
Basis of Lightning Technology. The specifications are currently being drafted
by several groups of implementers based around the world including the
developers of `lnd`. The set of specification documents as well as our
implementation of the specification are still a work-in-progress. With that
said, the current status of `lnd`'s BOLT compliance is:

  - [X] BOLT 1: Base Protocol
  - [X] BOLT 2: Peer Protocol for Channel Management
  - [X] BOLT 3: Bitcoin Transaction and Script Formats
  - [X] BOLT 4: Onion Routing Protocol
  - [X] BOLT 5: Recommendations for On-chain Transaction Handling
  - [X] BOLT 7: P2P Node and Channel Discovery
  - [X] BOLT 8: Encrypted and Authenticated Transport
  - [X] BOLT 9: Assigned Feature Flags
  - [X] BOLT 10: DNS Bootstrap and Assisted Node Location
  - [X] BOLT 11: Invoice Protocol for Lightning Payments

## Developer Resources

The daemon has been designed to be as developer friendly as possible in order
to facilitate application development on top of `lnd`. Two primary RPC
interfaces are exported: an HTTP REST API, and a [gRPC](https://grpc.io/)
service. The exported APIs are not yet stable, so be warned: they may change
drastically in the near future.

An automatically generated set of documentation for the RPC APIs can be found
at [api.lightning.community](https://api.lightning.community). A set of developer
resources including guides, articles, example applications and community resources can be found at:
[docs.lightning.engineering](https://docs.lightning.engineering).

Finally, we also have an active
[Slack](https://lightning.engineering/slack.html) where protocol developers, application developers, testers and users gather to
discuss various aspects of `lnd` and also Lightning in general.

First-time contributors are [highly encouraged to start with code review
first](docs/review.md), before creating their own Pull Requests.

## Installation
  In order to build from source, please see [the installation
  instructions](docs/INSTALL.md).

## Docker
  To run lnd from Docker, please see the main [Docker instructions](docs/DOCKER.md)

## IRC
  * irc.libera.chat
  * channel #lnd
  * [webchat](https://web.libera.chat/#lnd)

## Safety

When operating a mainnet `lnd` node, please refer to our [operational safety
guidelines](docs/safety.md). It is important to note that `lnd` is still
**beta** software and that ignoring these operational guidelines can lead to
loss of funds.

## Security

The developers of `lnd` take security _very_ seriously. The disclosure of
security vulnerabilities helps us secure the health of `lnd`, privacy of our
users, and also the health of the Lightning Network as a whole.  If you find
any issues regarding security or privacy, please disclose the information
responsibly by sending an email to security at lightning dot engineering,
preferably encrypted using our designated PGP key
(`91FE464CD75101DA6B6BAB60555C6465E5BCB3AF`) which can be found
[here](https://gist.githubusercontent.com/Roasbeef/6fb5b52886183239e4aa558f83d085d3/raw/1ecb328bbcf36f76ead67f08008f8db1da07e60e/security@lightning.engineering).

## Further reading
* [Step-by-step send payment guide with docker](https://github.com/lightningnetwork/lnd/tree/master/docker)
* [Contribution guide](https://github.com/lightningnetwork/lnd/blob/master/docs/code_contribution_guidelines.md)


