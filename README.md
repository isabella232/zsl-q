# ZSL on Quorum

## Introduction

This proof of concept (POC) is intended to demonstrate how ZSL can complement Quorum, and provide a platform for experimentation and exploration of different use cases. It implements a simplified, stripped-down version of the Zerocash protocol to enable rapid prototyping. There is no formal security proof for the protocol, and it should not be considered “production-ready”. 

One of the omissions is the lack of a digital signature over the transaction to authenticate and prevent malleability. Addressing this shortcoming is straightforward (albeit time consuming) to implement. However, [forthcoming improvements to zk-SNARKs](https://z.cash/blog/cultivating-sapling-faster-zksnarks.html) offer a simpler, more efficient way to address malleability (as well as faster proof generation), and we expect to implement these improvements for the ZSL/Quorum protocol as we progress from a POC to a production-ready implementation.

## License

The contents of this repository are licensed under the Apache License 2.0.

## Contents

This repository contains:

* Solidity contracts to provide access to ZSL functionality added to Quorum.  Currently these have been written with Solidity 0.3.6 to be compatible with Quorum 1.5.  In future, they are likely to be updated to Solidity 0.4.0 for Quorum 1.6.
* Example contracts using ZSL to demonstrate an example use-case of a private trade of two different assets.
* Zsl-golang library which adds ZSL functionality to Quorum.  Currently developed with go version 1.7.3.
* Utility to create custom parameters for use with ZSL.

## ZSL API

Quorum with ZSL provides a JavaScript API accessible under the zsl.* namespace in the Quorum terminal.

* zsl.getRandomness() returns buffer
* zsl.getNewAddrses() returns keypair
* zsl.debugShielding() returns bool
* zsl.debugUnshielding() returns bool
* zsl.debugShieldedTransfer() returns bool
* zsl.loadTracker(filename) returns json
* zsl.saveTracker(filename, json) returns bool
* zsl.getCommitment(rho, pk, value) returns hash
* zsl.getSendNullifier(rho) returns hash
* zsl.getSpendNullifier(rho, sk) returns hash
* zsl.createShielding(rho, pk, value) returns proof
* zsl.createUnshielding(rho, sk, value, treeIndex, authPath) returns proof
* zsl.createShieldedTransfer(rho1, sk1, value1, treeIndex1, authPath1, rho2, sk2, value2, treeIndex2, authPath2, outrho1, outpk1, outvalue1, outrho2, outpk2, outvalue2) returns proof
* zsl.verifyShielding(proof, send_nf, commitment, value) returns bool
* zsl.verifyUnshielding(proof, spend_nf, root, value) returns bool
* zsl.verifyShieldedTransfer(proof, anchor, spend_nf1, spend_nf2, send_nf1, send_nf2, commitment1, commitment2) returns bool

## Zsl-golang library

This library provides a Golang API for Quorum to create and verify shielded transactions.  Internally, the library makes use of the C++ libsnark library.

### Building

System requirements for building are openssl and boost.

* Ubuntu: `sudo apt get install libboost-all-dev libssl-dev`
* Fedora: `sudo dnf install openssl-devel boost-devel`

Build using a Makefile:

    make

### Testing

Testing the library require proving and verification keys, also known as parameters.

Example parameters can be downloaded from [zsl-q-params](http://github.com/jpmorganchase/zsl-q-params).

Custom parameters can be generated by using the `createparams.go` tool found in the `util` folder of the repository.

To create parameters:

    go run createparams.go

Copy the parameter files into the current directory, and test the library with:

    go test

# Using ZSL on Quorum

## Introduction

To use ZSL with 7nodes, one of the [Quorum examples](https://github.com/jpmorganchase/quorum-examples), clone the following repositories into a folder, e.g. `$HOME/projects/`

* Quorum: https://github.com/jpmorganchase/quorum (zsl branch)
* Contracts/Library: https://github.com/jpmorganchase/zsl-q
* Parameters: https://github.com/jpmorganchase/zsl-q-params
* 7Nodes: https://github.com/jpmorganchase/quorum-examples (zsl branch)

## Setting Up

Build `quorum` and copy binaries into a folder in the `quorum-examples` project:

    mkdir ~/projects/quorum-examples/zsl-tmp/
    cd ~/projects/quorum
    make all
    cp build/bin/* ~/projects/quorum-examples/zsl-tmp/

Download the parameters found in the Github `Release` tab of `zsl-q-params` and copy into the `quorum-examples` folder:

    cp shielding.pk ~/projects/quorum-examples/
    cp shielding.vk ~/projects/quorum-examples/
    cp unshielding.pk ~/projects/quorum-examples/
    cp unshielding.vk ~/projects/quorum-examples/
    cp transfer.pk ~/projects/quorum-examples/
    cp transfer.vk ~/projects/quorum-examples/

Next, follow the set of instructions depending on whether you want to run 7nodes inside a VM or locally on your machine.

### Running inside VM

Launch the VM.

    vagrant up
    vagrant ssh

When launching, `vagrant/bootstrap.sh` will automatically run and copy geth and bootnode from the `zsl-tmp` folder into the VM path `/usr/local/bin'.

Symbolic links should already exist to the parameters, but if not, create them manually:

    cd quorum-examples/7nodes
    ln -s /vagrant/shielding.pk shielding.pk
    ln -s /vagrant/shielding.vk shielding.vk
    ln -s /vagrant/unshielding.pk unshielding.pk
    ln -s /vagrant/unshielding.vk unshielding.vk
    ln -s /vagrant/transfer.pk transfer.pk
    ln -s /vagrant/transfer.vk transfer.vk

### Running on local machine

When running the example locally and not in a VM, in the folder `quorum-examples/examples/7nodes`:

* delete symbolic links: `rm *.pk *.vk`
* move the parameters into the folder to replace the deleted symbolic links.
* ensure the quorum binaries can be found in your local `$PATH` i.e. copy the quorum binaries into `/usr/local/bin/` or add `~/projects/quorum/build/bin` to your `$PATH`.

## Launch 7nodes

In folder `quorum-examples/examples/7nodes` enter:

    ./raft-init.sh
    ./raft-start.sh
    geth attach qdata/dd1/geth.ipc

Now that you have the 7nodes example environment running, you can try out some of the examples below.

## Javascript Functions

Some of the examples use the ZSL API, a set of Javascript functions added to Quorum and available under the `zsl.*` namespace.  You can list them in the Quorum terminal by entering `zsl.` and pressing the TAB key.

The examples also  make use of `tracker.js` which is a file containing a collection of helpful functions, some of which are grouped under the `zdemo.*` namespace.

Ztracker:

* ztracker.shield(ztoken, value)
* ztracker.unshield(note_uuid)
* ztracker.list(ztoken, optional filter amount)
* ztracker.balance(ztoken)
* ztracker.load(filename)
* ztracker.save(filename)

Zdemo:

* zdemo.create_ztoken(tokenName, initialSupply)
* zdemo.get_ztoken(address)
* zdemo.create_zprivatecontract(tracker, bid_constellation_address, bid_token, bid_amount, ask_constellation_address, ask_token, ask_amount)
* zdemo_create_zprivatecontract(tracker, bid_constellation_address, bid_token, bid_amount, ask_constellation_address, ask_token, ask_amount)
* zdemo.get_zprivatecontract(address)
* zdemo.accept_bid(tracker, zprivatecontract)
* zdemo.submit_payment: function(tracker, ztoken, zprivatecontract, noteId)
* zdemo.submit_settlement: function(tracker, ztoken, zprivatecontract, noteId)
* zdemo.watch_events: function(zprivatecontract)

## Example 1 - Shield and unshield z-contract funds

Create the ztoken:

    loadScript('tracker.js')
    ztoken = zdemo.create_ztoken('ACME', 100000)

Explore the ztoken:

    ztoken.name()
    ztoken.balance()
    ztoken.size()
    ztoken.depth()

Generate a keypair:

    keypair = zsl.getNewAddress()
    pk = keypair['a_pk']
    sk = keypair['a_sk']

Generate some randomness:

    rho = zsl.getRandomness()

How much do we want to shield from our funds?

    value = 30000

Generate proof for shielding:

    result = zsl.createShielding(rho, pk, value)

    proof = result['proof']
    cm = result['cm']
    send_nf = result['send_nf']

Let’s verify this proof manually (the ztoken will automatically perform this step)

    zsl.verifyShielding(proof, send_nf, cm, value)

Let's ask the ztoken contract to add this proof and update balances.

    ztoken.shield(proof, send_nf, cm, value, {from:eth.accounts[0], gas:54700000})

Once the transaction has been mined, verify the zcontract has been updated.

    ztoken.size()
    ztoken.balance()

Try to add the same proof again, it won't be allowed.

    ztoken.shield(proof, send_nf, cm, value, {from:eth.accounts[0], gas:54700000})
    ztoken.size()
    ztoken.balance()

Let's create a proof to unshield these funds.

    rt = ztoken.root()
    witnesses = ztoken.getWitness(cm)
    treeIndex = parseInt(witnesses[0])
    authPath = witnesses[1]

    result = zsl.createUnshielding(rho, sk, value, treeIndex, authPath)

    proof = result['proof']
    spend_nf = result['spend_nf']

Let's verify this proof manually (the ztoken will automatically perform this step)

    zsl.verifyUnshielding(proof, spend_nf, rt, value)

Now ask the ztoken contract to unshield and update balances:

    ztoken.unshield(proof, spend_nf, cm, rt, value, {from:eth.accounts[0], gas:54700000})

Once the transaction has been mined, verify the zcontract has been updated.

    ztoken.size()
    ztoken.balance()

Try to unshield again, it's not allowed.

    ztoken.unshield(proof, spend_nf, cm, rt, value, {from:eth.accounts[0], gas:54700000})
    ztoken.balance()

Congratulations, you've just completed your first shielding and unshielding!

## Example 2 - Private Contract Trade

Node 1 and Node 2 will execute a private trade of two different assets.  In this example, Node 1 will offer 300 USD for 100 ACME.

On both node 1 and node 2:

Store node account and private constellation address in local variables.

    var node1 = "0xed9d02e382b34818e88b88a309c7fe71e65f419d";
    var node2 = "0xca843569e3427144cead5e4d5999a3d0ccf92b8e";
    var constellation1 = 'BULeR8JyUWhiuuCMU/HLA0Q5pzkYT+cHII3ZKBey3Bo=';
    var constellation2 = 'QfeDAys9MPDs2XHExtc84jKGHxZg/aj52DTh0vtA3Xc=';

Load demo helper functions and a lightweight note tracker which acts like a wallet. 

    loadScript("tracker.js")

On node 1, create two z-contracts:

    ztoken_usd = zdemo.create_ztoken("USD",100000)
    ztoken_acme = zdemo.create_ztoken("ACME",50000)

On node 1, create a note tracker:

    tracker = new ztracker();

On node 1, shield 10000 USD and transfer 5000 ACME to node 2.

    tracker.shield(ztoken_usd, 10000)
    ztoken_acme.transfer(node2, 5000, {from:node1, gas:5470000});

On node 1, create a private contract for a trade, 300 USD for 100 ACME.

    zprivatecontract = zdemo.create_zprivatecontract(tracker, constellation1, ztoken_usd, 300, constellation2, ztoken_acme, 100);

On node 1, watch for events that update the state of the trade.

    zdemo.watch_events(zprivatecontract)

On node 1, obtain the address of the private contract.

    a = zprivatecontract.address

On node 2, use this address to reference the private contract.

    zprivatecontract = zdemo.get_zprivatecontract(a)

On node 2, watch private contract events and reference the z-contracts involved in the trade.

    zdemo.watch_events(zprivatecontract)
    ztoken_usd = zdemo.get_ztoken(zprivatecontract.bidTokenAddress())
    ztoken_acme = zdemo.get_ztoken(zprivatecontract.askTokenAddress())

On node 2, set up a a note tracker and shield some of the ACME funds received from node 1.

    tracker = new ztracker()
    tracker.shield(ztoken_acme, 1000)
    ztoken_acme.balance()
    tracker.balance(ztoken_acme)
    tracker.list(ztoken_acme, 1111)
    tracker.list(ztoken_acme, 500)

On node 2, accept the bid of 300 USD for 100 ACME.

    zdemo.accept_bid(tracker, zprivatecontract)

The trade has now been agreed, so it's time for both nodes to settle the trade.

On node1, select a note to spend by using its identifier (uuid).
    
    tracker.list(ztoken_usd)
    uuid = ...

On node 1, submit the 300 USD payment to node 2.

    zdemo.submit_payment(tracker, ztoken_usd, zprivatecontract, uuid)
    tracker.spent
    tracker.list(ztoken_usd)

on node2, settle the trade by sending 100 ACME to node 1.

    tracker.list(ztoken_acme)
    uuid = ...
    zdemo.submit_settlement(tracker, ztoken_acme, zprivatecontract, uuid)

The trade should now be settled.  Now check balances and unshield funds.

On node 2, unshield the USD received.

    ztoken_usd.balance()
    tracker.balance(ztoken_usd)
    tracker.list(ztoken_usd)
    uuid = ...
    tracker.unshield(uuid)
    ztoken_usd.balance()
    tracker.balance(ztoken_usd)
    tracker.list(ztoken_usd)
    tracker.spent

on node 1, unshield the ACME received.

    ztoken_acme.balance()
    tracker.list(ztoken_acme)
    uuid = ...
    tracker.unshield(uuid)
    ztoken_acme.balance()
    tracker.list(ztoken_acme)

Congratulations, you've completed your first private trade!

## Example 3 - Tracker Persistence

Continuing example 2 above, on node 1, save the tracker to disk.

    tracker.save("node1.tracker")

On node 2, create a new tracker from the data on disk.

    tmp = new ztracker();
    tmp.load("node1.tracker")

Examine the new tracker to confirm that the data matches that of node 1.

    tmp.list(ztoken_usd)
    tmp.list(ztoken_acme)
    tmp.keypair

The loading and saving of tracker data to disk is for demo purposes only.
