2017-01-09 Lit sortof works on testnet.  There are known bugs / omissions / errors, and also unknown ones!  Reporting bugs / crashes is helpful, and fixing them is even more helpful.

2018-06-12 Updated to use new addresses and command formats.  Lit also works much better than it did in the above note.

## Lit 0.0 Walkthrough

This walkthrough is to set people up who want to send payments over channels on a test network. If you haven't already, make sure to go to the [README](./README.md) for setup instructions.

### Step 1: Files in place

Make sure you have built both the `lit` and `lit-af` (in `cmd/lit-af`) packages with `go build`

If you have a full node, like bitcoind or btcd, start running that as well, either on testnet3 or regtest mode.

For this walkthrough, you will run 2 lit nodes and have them make channels.  It's more fun for the nodes to be on different computers controlled by different people, but you can run them on the same machine for testing.  You can also have the full node running on that machine as well.

In this example, there are 2 computers.  Most things should work the same with 1 computer; just make sure to have 2 different folders for the lit nodes.

Set up two folders (i.e. anode and bnode) and copy both the `lit` and `lit-af` executables you built into both folders.

Here is a sample for Linux users who have not already built the packages

#### Alice's setup

```
alice@pi2:~$ mkdir anode
alice@pi2:~$ cd gofolder/src/github.com/mit-dci/lit
alice@pi2:~/gofolder/src/github.com/mit-dci/lit$ go build
alice@pi2:~/gofolder/src/github.com/mit-dci/lit$ cp lit ~/anode/
alice@pi2:~/gofolder/src/github.com/mit-dci/lit$ cd cmd/lit-af
alice@pi2:~/gofolder/src/github.com/mit-dci/lit/cmd/lit-af$ go build
alice@pi2:~/gofolder/src/github.com/mit-dci/lit/cmd/lit-af$ cp lit-af ~/anode/
alice@pi2:~/gofolder/src/github.com/mit-dci/lit/cmd/lit-af$ cd ~/anode/
alice@pi2:~/anode$
```

### Step 2: Run lit and sync up

Alice starts running lit (with `./lit --tn3 fullnode.net`) and syncs up to the blockchain.  The lit node will print lots of stuff on the screen, but can't be controlled from here.

Alice connects to her full node, fullnode.net.  By default this is on testnet3, using port 18333.

```
alice@pi2:~/anode$ ./lit --tn3=fullnode.net -v
lit node v0.1
-h for list of options.
No file testkey.hex, generating.
passphrase:
repeat passphrase:

WARNING!! Key file not encrypted!!
```

Here Alice can type a passphrase to secure the wallet the newborn lit node is generating.  But she just pressed enter twice, because this is testnet.  Even though it's testnet, lit will show warnings about not using a passphrase.

Now in another window, alice connects to the lit node over RPC using `./lit-af`

```
alice@pi2:~/anode$ ./lit-af
lit-af# ls -a
entered command: ls -a
        Peers:
1 127.0.0.1:49958 (ln1wj3csaf6p6kgvj6zre6zqta7lnjms9zndkwc8d)
        Listening Ports:
Listening for connections on port(s) [[::]:2448] with key ln1akupazfterdadrywclht0s8kgvt3r3jga0lyez
        Addresses:
1 tb1qf77l4ja2kq6na0ffmekph6qv22puufqr5aatg0 (mnnbGE9dxHpNFuzBrTWUk5PZ9272JF2bkK)
        Balances:
Type: 1 Sync Height: 1481792    FeeRate: 80     Utxo: 0 WitConf: 0 Channel: 0

```

Buy default the lit-af utility use ~/.lit directory for litHomeDir and 2448 for autoListenPort.
You can change this running:
alice@pi2:~/gofolder/src/github.com/mit-dci/lit/cmd/lit-af/lit-af --litHomeDir=/path/to_alice/lit1 --autoListenPort=2448



ls -a shows how much money Alice has, which is none.  She can get some from a faucet. One such faucet you can find [here](https://testnet.manu.backend.hamburg/faucet).

Bob can start a wallet in the same way; if Bob's node is running on the same computer, he'll have to run lit placing a lit.conf in to the ~/bnode folder:

tn3=1
rpchost=0.0.0.0
rpcport=8002
tracker=http://hubris.media.mit.edu:46580
autoListenPort=2449
autoReconnect=true
autoReconnectInterval=5

Then run lit as:

alice@pi2:~/gofolder/src/github.com/mit-dci/lit/lit --dir=~/bnode


Once Alice and Bob are both set up, they can connect to each other.

If Bob is on the same computer he should run lit-af with the parameters:
./lit-af --litHomeDir=~/bnode --autoListenPort=2449



### Step 3: Connect

Bob can connect to Alice using her node-ID:

```
lit-af# con ln1akupazfterdadrywclht0s8kgvt3r3jga0lyez@:2448@pi2
entered command: con ln1akupazfterdadrywclht0s8kgvt3r3jga0lyez@:2448@pi2
connected to peer 1
lit-af# say 1 Hi Alice!
entered command: say 1 Hi Alice!
```

Bob puts the pubkey@hostname for Alice and connects.  Then he says hi to Alice.

### Step 4: Sweep Funds

Before Bob can make a channel, he needs to sweep his coins to make sure they are in his segwit address.

```
lit-af# sweep tb1qf77l4ja2kq6na0ffmekph6qv22puufqr5aatg0 50000000
entered command: sweep tb1qf77l4ja2kq6na0ffmekph6qv22puufqr5aatg0 50000000
Swept
0 d297da04c43919e683ffc03539ee38e185425e8fa14d1ccae6577fbb35be575a
```

### Step 5: Open a channel

Bob is connected to Alice and wants to open a payment channel. If he has enough (segwit) money and is connected to Alice, he can open a channel.

```
lit-af# fund 1 1 50000000 0
```

This opens a channel with peer 1 (Alice) with a channel capacity of 50,000,000 satoshis (half a coin), and sends 0 satoshis over in the creation process.  Bob starts out with all 50,000,000 satoshis in the channel, so only he can send to Alice.  The second "1" is to specify the cointype, which is 1 for Bitcoin Testnet3.

### Step 6: Send micro-payments

(Do they count as micro-payments if testnet coins have zero value?)

use ls again to see that the channel is there, it will be labeled channel 1.

```
lit-af# push 1 200000
```

This pushes 200,000 satoshis to the other side of the channel.  You can do this 200 trillion times before the channel needs to be closed.  (Actually, since I don't think you can send that many payments, the software will probably crash if you do manage to exceed 2^48)

### Step 7: Close/Break the channel

You can close the channel cooperatively using:

```
lit-af# close 1
```
or non-cooperatively with:

```
lit-af# break 1
```

to close the channel at the current state. Uncooperative is maybe more fun because the node who breaks the channel has to wait 5 blocks before they can use their money.  The other node can spend immediately.  After a cooperative close, they can both spend immediately.  The `break` command does not need a connection to the other node.
