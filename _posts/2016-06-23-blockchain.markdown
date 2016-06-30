---
layout: post
title:  "Blockchain & Bitcoin introduction" 
date:   2016-06-23 23:59:00
Tags: [blockchain, Bitcoin]
Categories: [security, database, distributed, peer2peer]
---

# The Blockchain (block-chain)
This article is intended to be an introduction to the blockchain technology, that was introduced with the Bitcoin currency. I will try to keep a technological approach rather than a financial one. You will not find any discussion about how to buy or manage Bitcoins. You will not find either my personal opinion on the Bitcoin and wether you should invest in/use that currency. 

## Is a database
A `blockchain` is a way to store data. It can be compared to a new type of database. It is composed of `blocks` that are `chained` togethers thus forming a `block-chain`. Each `block` is linked to the previous block of the chain. A block contains data or even programs in some recent implementations. 

<img class="center" src="http://ronanquillevere.github.io/img/blockchain1.png" alt="blockchain" width="600px">

The blockchain was the technological innovation behind the first decentralized digital currency, the `Bitcoin`. Nevertheless other types of blockchains have been created since which are not targeted to be digital currencies. The real revolution of the blockchain is the promise of reducing the cost of establishing and maintaining trust for both individuals and organizations. This trust is enforced by various mechanisms that we will detail below. 

In the following paragraph we will use the Bitcoin as an example of blockchain. For the Bitcoin, each block contains a list of transactions like this

    Account A sent 10 coins to Account B
    Account C sent 5 coins to Accound D


## Is distributed
We need to remember that the blockchain, that was introduced by the Bitcoin currency, had one objective : 

>A purely peer-to-peer version of electronic cash [which] would allow online payments to be sent directly from one party to another without going through a financial institution.

Basically the goal is to remove the trusted third party needed in electronic transactions. One of the problem with trusted third parties is that they respresent a `single point of failure`. Thus the Bitcoin blockchain was designed to be distributed. In fact, each node composing the Bitcoin network contains a copy of the entire blockchain. Because each block contains a list of transactions, the Bitcoin blockchain is also refered as a `public ledger`. No central party has ownership of the ledger, therefore no one can individually amend the entries already on the blockchain.

When new transactions arrives they need to be verified, meaning if an account A has a balance of +10 coins, the owner of account A should not be able to spend more than those 10 coins. This problem is also known as the `double-spending` problem, where one could use one digital token/coin twice. This problem cannot happen in real life, with real coins, unless you are a good magician. This verification is also distributed. It is not the responsibility of someone in particular to verify the transactions. Instead, all nodes validate the transaction independently. This is possible because the blockchain is known by everyone and because transactions are not encrypted, so anyone can go back in the past and recompute the balance of an account.

<img class="center" src="http://ronanquillevere.github.io/img/blockchain-bitcoin.png" alt="blockchain-bitcoin" width="800px">

## Uses adress pseudonyms and digital signatures

Distributing the blockchain would not be enough to make the system safe. Users making transactions should stay anonymous. A user may not want anyone to know that he has concluded a transaction with some other user. This is also why users (= account) are referenced as an `adress`. An adress is an identifier of 26-35 alphanumeric characters. Unlike e-mail addresses, people have many different Bitcoin addresses and a unique address should be used for each transaction. To simplify you can see an adress as a pseudonym. If your pseudonym is being linked to your real identity and you always use the same pseudonym for each transaction then all your transactions will be known, because the entire blockchain is accessible to anyone.

Here is an example of an adress.

    1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2

Also someone should not be able to steal your identity and use your Bitcoins to buy something. That is why each transaction must be digitaly signed. The owner of a Bitcoin address has the private key associated with the address. To spend Bitcoins, he must sign his transaction with his private key, which proves he is the owner. The public key associated with each Bitcoin address is public, so anyone can verify the digital signature of the transaction. 


## Relies on miners
Transactions are stored inside blocks. But blocks are not free to create and require a `proof-of-work`. A proof-of-work (POW) system (or protocol, or function) is an economic measure originaly designed to deter denial of service attacks and other service abuses such as spam on a network by requiring some work from the service requester, usually meaning processing time by a computer. Bitcoin uses the [Hashcash](https://en.Bitcoin.it/wiki/Hashcash) proof of work system.

The good analogy here is to compare a block to a gold nugget. Finding one requires a lot of efforts thus one should be rewarded for finding it. Once found, the new gold nugget is added to the total amount of gold in circulation. That is why people "finding" blocks are called `miners`. Note that the Bitcoin block mining reward halves every 210,000 blocks. The coin reward will decrease from 25 (16.000 us dollar) to 12.5 coins around the 9th of July 2016.

For a block to be valid it must hash to a value less than the current `target`. The target is a 256-bit number (extremely large) that all Bitcoin clients share. The `SHA-256` hash of a block's header must be lower than or equal to the current target for the block to be accepted by the network. The lower the target, the more difficult it is to generate a block. Each block contains the hash of the preceding block, thus each block has a chain of blocks that together contain a large amount of work. Changing a block (which can only be done by making a new block containing the same predecessor) requires regenerating all successors and redoing the work they contain. This protects the block chain from tampering. 

### Who are hard workers (and good at math)

Exadecimal is a numeral system with a base of 16. Each hexadecimal digit represents four binary digits (bits).
    
<img class="center" src="/img/hexa.png" alt="hexa" width="400px">

As mentionned before the target hash is a number of 256 bits (= 64 hexadecimal characters). Example :

    (0x) e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855

Let's define the following variables :

    Content of the block = b
    SHA-256 function = h()
    Target = t

So finding a block means
    
    h(b) < t

Pb is h() (and all hash functions) is `deterministic`, meaning

	b1 = b2 => h(b1) = h(b2)

It means that if the content of the block does not change, the SHA-256 hash of that content will stay the same. That is why each block contains a 'nonce' field (= number used once = random number). So the real challenge is to find this random number so that `h(b) < t`. And this is a real lottery, because even a slight change in the nonce number will completly change the value of the hash.

Because hash functions are `uniforms`, every hash value in the output range should be generated with roughly the same probability. So the probability of finding a valid number equals the number of valid numbers (t) divided by the number of possible output 2^256

    t / 2^256

So in average one will need 2^256 / t attempts to find a solution. The lower the target, the higher the number of attemps. This number can be huge ! 

The bitcoin network, for reasons of stability and low latency in transactions, will adjust dynamically the difficulty so that a block is found in average every 10 minutes. This enables transactions on the network to be validated in an acceptable delay. The adjustement mechanism is based on the time taken to compute the previous 2016 blocks. In the long run, the tendency is to raising the difficulty in order to compensate the increase of computation power of mining pools. 

## Summary

The steps to run the network are as follows:

1. New transactions are broadcast to all nodes.
2. Each node collects new transactions into a block.
3. Each node works on finding a difficult proof-of-work for its block.
4. When a node finds a proof-of-work, it broadcasts the block to all nodes.
5. Nodes accept the block only if all transactions in it are valid and not already spent.
6. Nodes express their acceptance of the block by working on creating the next block in thechain, using the hash of the accepted block as the previous hash.

## Security

# Bibliography
* [Bitcoin: A Peer-to-Peer Electronic Cash System](https://Bitcoin.org/Bitcoin.pdf)
* [https://en.Bitcoin.it/wiki/](https://en.Bitcoin.it/wiki/)
* [http://Bitcoinfees.com/](http://Bitcoinfees.com/)
* [https://Bitcoin.org/Bitcoin.pdf](https://Bitcoin.org/Bitcoin.pdf)
* [https://en.wikipedia.org/wiki](https://en.wikipedia.org/wiki/)
* [https://bitsonblocks.net/2015/09/01/a-gentle-introduction-to-Bitcoin/](https://bitsonblocks.net/2015/09/01/a-gentle-introduction-to-Bitcoin/)
* [https://blockchain.info/fr/](https://blockchain.info/fr/)