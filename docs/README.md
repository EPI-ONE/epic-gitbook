# Overview

> We want to express appreciation for being able to take advantage of the work that’s been done by others before us. We didn’t invent the language or mathematics. Everything we do depends on other members of our species and the shoulders that we stand on. We try to use the talents we do have to add something to that flow.
>
> — Steve Jobs

## Objective

`epic` is a POW (proof-of-work) chain that achieves an order of magnitude improvement on the throughput over current mainstream cryptocurrencies, e.g, Bitcoin, without compromising security and decentralization. The designed max TPS (transaction per second) is **1024** per second. Miners collectively create **8** `blocks` per second on average, each containing up to **128** `transactions`. You may join the `epic` network by running a node following the [instruction](getting-started/compile-run). `epic` uses the same UTXO system as Bitcoin, consequently no rosy applications like Hello Kitty or Texas Poker for now.

## Consensus Mechanism

The main idea is to expand a chain of blocks, e.g. the Bitcoin blockchain, to a more general graph of blocks organized in a certain structure. Refer to our [paper](https://arxiv.org/abs/1901.02755) for the full description or this video [presentation](https://youtu.be/UEeYkIvl6dA) for a verbal explanation. Here we provide a brief description and some intuition. The blocks are organized in a **structured Directed Acyclic Graph** (sDAG): 

- Each miner forms a `peer chain` by connecting his newly mined block to his most recently created block. Miners, depending on their mining power, has the flexibility of creating several parallel `peer chains` by splitting their mining power to these parallel chains.
- Each block turns out to be either a `milestone` with probability $$p$$, or a `regular` block with probability $$1-p$$. $$p$$ is determined by the current number of transactions getting processed and the hashing power of the entire network. In general, $$p$$ increases as the transactions pile up and as the network hashing power grows. Before a block is successfully mined, the miner has no idea which type it will turn out to be.
- In addition to connecting to its immediate preceding block in the peer chain, each block, regardless of its type, must point to two additional blocks: the most recent `milestone` and another `regular` block on a different `peer chain`.  

In our sDAG, all the `milestones` form a chain, connecting the parallel `peer chains`. The connectivity among the parallel `peer chains` is quite strong since each block needs to point to another `regular` block on a different `peer chain`. 

### Continuous propagation over the network

Organized in the sDAG, we will be able to break a big block, e.g., Bitcoin block containing thousands of transactions, into smaller ones, allowing quick dispersions of information across a peer-to-peer network continuously. The motivation for such a design is that the state-of-art Bitcoin blockchain network propagates a block of 1 Megabyte periodically every 10 minutes on average. During the interval, the capacity of the network is wasted. Many smaller blocks propagating over a peer-to-peer network in a more "continuous" fashion significantly improve the utilization of the network capacity. Fundamentally, to build a high TPS system, the corresponding amount of information has to be synchronized across the network. Increasing the utilization of network, before the current Internet infrastructure upgrades to be faster, is important.

### Reaching consensus 

While benefitting from the increased utilization of the network capacity, many smaller blocks organized in a more general structure than a chain certainly brings challenges on reaching consensus. This is resolved by the design of our sDAG. We say a block is *confirmed* by a `milestone` if it is connected to the `milestone` directly or indirectly. For each `milestone`, it is associated with a *level set*, all the regular blocks confirmed by this `milestone` but not by any preceding `milestone`. Essentially, all the level sets form a chain, same as the Bitcoin blockchain. In this way, a consensus is reached in the same way as the Bitcoin by POW and Nakamoto consensus proposed in the original [Bitcoin white paper](https://bitcoin.org/bitcoin.pdf). As a result, the security of `epic` is guaranteed in the exactly the same way as the Bitcoin.

### Fast confirmation

To ensure a certain security level, Bitcoin needs to wait for a certain number of blocks to *confirm a transaction*. For example, to limit the probability of a successful attack by 0.001 given 10% malicious mining power, we will have to wait for 6 blocks. In order to achieve the same security, we need to wait for the same number of milestones provided the same percentage of malicious mining power. A simple calculation based on the above parameters shows that a `milestone` will be created on average every 16 seconds in `epic`. We can achieve a faster transaction confirmation because the speed of creating milestones is faster (16 seconds v.s. 10 minutes), not because we are using a more superior consensus mechanism. However, we are working on incorporating the idea of multiple verifying chains proposed in the [paper](https://arxiv.org/abs/1810.08092) so that confirmation can be much faster because the number of milestones we have to wait is reduced.

### Transaction Assignment

In a high TPS system, different miners may work on the same `transaction` from the mempool causing waste of capacity, despite that consensus is not affected. To reduce waste, we design the transaction assignment rule. A miner can process a `transaction` only if the hash of this transaction and his most recent block exhibit a certain pattern. Intuitively, transactions in the mempool are "distributed" to miners (more precisely `peer chains`) in a probabilistic fashion. Our [paper](https://arxiv.org/abs/1901.02755) shows a quantitative computation of how such probability affects the waiting time of a transaction in the mempool and the wasted capacity. Under our carefully chosen parameters, the capacity waste is controlled under 1% and the waiting time in mempool is roughly in the order of seconds when the system is underloaded. In fact, the waiting time will largely depend on the speed at which all the users generate transactions if that speed is close to the system capacity, namely the max TPS. 

### POW and reward scheme

Blocks are created by POW, using the [cuckoo](https://github.com/tromp/cuckoo) algorithm. Users may choose to provide transaction fees in the same way as in the Bitcoin system. In addition to this reward, each block brings a block reward. Each `milestone` brings an additional reward depending on the number of regular blocks in its `level set`. Such a reward scheme incentivize miners to connect to recent blocks on another `peer chain` to enhance the connectivity of the sDAG. 

In Bitcoin, each block reward generates a coinbase. If blocks are mined at a much higher speed, the number of UTXO will explode. To avoid this, we let miner redeem his reward by creating a normal transaction every once in a while, with the frequency completely depending on the miner. 
