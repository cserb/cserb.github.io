---
title: "Build your first blockchain from scratch (#1 Probabilistic Finality)"
date: 2019-10-31T11:49:12+01:00
type: post
draft: false
---
### This is the first part in a series that will teach you how to develop a full featured blockchain from scratch.

First you have to understand some concepts behind blockchain technology and how they work together.

In this article we will focus on the concept of "Probabilistic Finality".

**1. Probabilistic Finality**

Finality is the agreement that a valid transaction between *A* and *B* has been executed and can't be reverted. If *A* uses a Visa card to buy coffee from *B*, *B* trusts its bank that the agreement the bank made with Visa will ensure a finalized transaction.

In a decentralized network the trust is given to the majority of the participants (nodes) and we find out what is considered finalized by looking at how the majority sees the state of the transaction.

Learn more about it here: [On Settlement Finality](https://blog.ethereum.org/2016/05/09/on-settlement-finality/), [Finality in Blockchain Consensus [video]](https://www.youtube.com/watch?v=efyiPhZvqOA)

So how do we "implement finality" in our blockchain?

Imagine you are a *node* within our blockchain network. Your node is maintaining a copy of the blockchain data and at the moment it's waiting for the next block to be propagated. Current block is `#3000` and `#3001` should arrive soon. But because there are multiple miners there is a chance you will receive 2 versions of `#3001`. Which one to choose?

![Alt Text](https://thepracticaldev.s3.amazonaws.com/i/0hobbj89lkrwdy7gaeua.png)

The example above shows how the current tip of the blockchain could look like. In the end we don't choose any of the propagated `#3001` blocks, but wait for several more consecutive blocks to make a decision.

To understand why waiting makes sense we need to understand *Proof of Work* (PoW).

**2. What is PoW?**

PoW is the algorithm that makes mining possible, and it works in a way that adds a component of randomness to this process. There are other use cases for PoW, but for finality we are interested in the randomness, which makes it unlikely that a node will receive 2 versions of the next block at the same time.

**3. Why is randomness important?**

First let's find out what would happen if we build a blockchain without PoW. If every node in the network would have the ability to create a block instantly, it would look like in the example below

![Alt Text](https://thepracticaldev.s3.amazonaws.com/i/2f330n5qeruall6jk3d6.png)

Multiple miners will create a block at the same time and broadcast it to the network. It would be impossible to reach consensus, if for each round `T` all blocks of `Tn` would be propagated at the same time.

It can still happen that different miners find the next block at the same time (see example below), but at some point the chances that *only one* miner will find the next block are very high and as a consequence we can consider the chain of `B1.1-> B2.1->B3.1->B4.1->B5.1` as the longest one. Also we can decide to finalize block `B2.1` if we want.

![Alt Text](https://thepracticaldev.s3.amazonaws.com/i/macwc1kyewgfb5cl5n05.png)

Note: Block 4.2 has been abandoned mid-mining because it just wasn't fast enough. So there is no reason to waste more energy on it.

**4. Conclusion**

In order to reach probabilistic finality we need a way to store blocks and pick out and mark a block as finalized. Further we also need PoW to ensure randomness of block propagation.

**5. Next up**

In the next article I will write how to use a Directed Acyclic Graph to find the longest chain.

[Part #2: DAGs](https://dev.to/cerbivore/build-your-first-blockchain-from-scratch-2-dag-4f8k)
