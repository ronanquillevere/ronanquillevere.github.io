---
layout: post
title:  "Blockchain & bitcoin introduction" 
date:   2016-06-23 23:59:00
Tags: [blockchain, bitcoin]
Categories: [security, database, distributed, peer2peer]
---

# The Blockchain (block-chain)
This article is intended to be an introduction to the blockchain technology, using the bitcoin currency as an example. I will try to keep a technological approach rather than a financial one. My point is to explain the technology, not how to manage bitcoins or to discuss if investing in bitcoins is a good idea or even regulatory issues linked to that new digital currency.

## Is a database
A `blockchain` is a way to store data. It can be compared to a new type of database. It is composed of `blocks` that are `chained` togethers thus forming a `block-chain`. Each `block` is linked to the previous block of the chain. A block contains data or even programs in some recent implementations. 

<img class="center" src="http://ronanquillevere.github.io/img/blockchain1.png" alt="blockchain" width="600px">

The blockchain was the technological innovation behind the first decentralized digital currency, the `bitcoin`. Nevertheless other types of blockchains have been created since which are not targeted to be digital currencies. The real revolution of the blockchain is the promise of reducing the cost of establishing and maintaining trust for both individuals and organizations. This trust is enforced by various mechanisms that we will detail below. 

In the following paragraph we will use the bitcoin as an example of blockchain. For the bitcoin, each block contains a list of transactions like this

    Account A sent 10 coins to Account B
    Account C sent 5 coins to Accound D


## Is distributed
We need to remember that the blockchain, that was introduced by the bitcoin currency, had one objective : 

>A purely peer-to-peer version of electronic cash [which] would allow online payments to be sent directly from one party to another without going through a financial institution.

Basically the goal is to remove the trusted third party needed in electronic transactions. One of the problem with trusted third parties is that they respresent a `single point of failure`. Thus the bitcoin blockchain was designed to be distributed. In fact, each node composing the bitcoin network contains a copy of the entire blockchain. Because each block contains a list of transactions, the bitcoin blockchain is also refered as a `public ledger`. No central party has ownership of the ledger, therefore no one can individually amend the entries already on the blockchain.

When new transactions arrives they need to be verified, meaning if an account A has a balance of +10 coins, the owner of account A should not be able to spend more than those 10 coins. This problem is also known as the `double-spending` problem, where one could use one digital token/coin twice. This problem cannot happen in real life, with real coins, unless you are a good magician. This verification is also distributed. It is not the responsibility of someone in particular to verify the transactions. Instead, all nodes validate the transaction independently. This is possible because the blockchain is known by everyone, so anyone can go back in the past and recompute the balance of an account.

<img class="center" src="http://ronanquillevere.github.io/img/blockchain-bitcoin.png" alt="blockchain" width="800px">

## Uses cryptography
Distributing the blockchain would not be enough to make the system safe. Users making transactions should stay anonymous. A user may not want anyone to know that he has concluded a transaction with some other user. Also someone should not be able to steal your identity and use your bitcoins to buy something. That is why each transaction must be digitaly signed.

This is also why users (= account) are referenced as an `adress`.

In addition cryptography is also used for generating blocks.

## Relies on miners
Transactions are stored inside blocks. But blocks are not free to create and require a `proof-of-work`. A proof-of-work (POW) system (or protocol, or function) is an economic measure originaly designed to deter denial of service attacks and other service abuses such as spam on a network by requiring some work from the service requester, usually meaning processing time by a computer. Bitcoin uses the [Hashcash](https://en.bitcoin.it/wiki/Hashcash) proof of work system.

The good analogy here is to compare a block to a gold nugget. It is not easy to find one, thus one should be rewarded for finding it. The new gold nugget is added to the total amount of gold. People "finding" blocks are called `miners`. The Bitcoin block mining reward halves every 210,000 blocks, the coin reward will decrease from 25 to 12.5 coins. 

For a block to be valid it must hash to a value less than the current target. The target is a 256-bit number (extremely large) that all Bitcoin clients share. The SHA-256 hash of a block's header must be lower than or equal to the current target for the block to be accepted by the network. The lower the target, the more difficult it is to generate a block. Each block contains the hash of the preceding block, thus each block has a chain of blocks that together contain a large amount of work. Changing a block (which can only be done by making a new block containing the same predecessor) requires regenerating all successors and redoing the work they contain. This protects the block chain from tampering. 

It's important to realize that block generation is not a long, set problem (like doing a million hashes), but more like a lottery. Each hash basically gives you a random number between 0 and the maximum value of a 256-bit number (which is huge). If your hash is below the target, then you win. If not, you increment the nonce (completely changing the hash) and try again.

For reasons of stability and low latency in transactions, the network tries to produce one block every 10 minutes. Every 2016 blocks (which should take two weeks if this goal is kept perfectly), every Bitcoin client compares the actual time it took to generate these blocks with the two week goal and modifies the target by the percentage difference. This makes the proof-of-work problem more or less difficult. A single retarget never changes the target by more than a factor of 4 either way to prevent large changes in difficulty.

## Summary

The steps to run the network are as follows:
1) New transactions are broadcast to all nodes.
2) Each node collects new transactions into a block.
3) Each node works on finding a difficult proof-of-work for its block.
4) When a node finds a proof-of-work, it broadcasts the block to all nodes.
5) Nodes accept the block only if all transactions in it are valid and not already spent.
6) Nodes express their acceptance of the block by working on creating the next block in the
chain, using the hash of the accepted block as the previous hash.

## Security

# Bibliography
* [Bitcoin: A Peer-to-Peer Electronic Cash System](https://bitcoin.org/bitcoin.pdf)
* [https://en.bitcoin.it/wiki/](https://en.bitcoin.it/wiki/)
* [http://bitcoinfees.com/](http://bitcoinfees.com/)
* [https://bitcoin.org/bitcoin.pdf](https://bitcoin.org/bitcoin.pdf)
* [https://en.wikipedia.org/wiki](https://en.wikipedia.org/wiki/)
* [https://bitsonblocks.net/2015/09/01/a-gentle-introduction-to-bitcoin/](https://bitsonblocks.net/2015/09/01/a-gentle-introduction-to-bitcoin/)