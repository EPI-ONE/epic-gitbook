---
description: >-
  > I think most creative people want to express appreciation for being able to
  take advantage of the work that’s been done by others before us. I didn’t
  invent the language or mathematics I use.
---

# Overview

> I think most creative people want to express appreciation for being able to take advantage of the work that’s been done by others before us. I didn’t invent the language or mathematics I use. I make little of my own food, none of my own clothes. Everything I do depends on other members of our species and the shoulders that we stand on. And a lot of us want to contribute something back to our species and to add something to the flow. It’s about trying to express something in the only way that most of us know how—because we can’t write Bob Dylan songs or Tom Stoppard plays. We try to use the talents we do have to express our deep feelings, to show our appreciation of all the contributions that came before us, and to add something to that flow.
>
> Steve Jobs

## About this project

Epic is a pow chain that achives an order of magnitude improvement on the throughput without sacraficing the security and decentralization.

## Motivation

Blockchain technology is fundamentally a consensus mechanism design based on a chain structure of storing information in a distributed manner over a peer-to-peer network. Bitcoin has been one of the most successful use case of this technology but its problem with scalability prevents it from being used as sound money, particularly due to poor medium of exchange.

The problem lies within the blockchain structure where transactions in a block would need to wait for 10 minutes before being confirmed and broadcasted to the network. Although this was implemented to reduce the frequency of forks which leads to wasted work, this particular design restricts Bitcoin from scaling. Further research for alternative designs is required to build the underlying technology for sound money.

## What is EPI?

EPI is a consensus mechanism design project based on directed acyclic graph \(DAG\). This project aims to solve Bitcoin's scalability issue by exploring different data structures that could scale without compromising decentralization and security. Our philosophy is to retain Bitcoin's core design that makes it robust and replacing/improving the parts that restrict its scaling capability.

This design fundamentally breaks one Bitcoin blocks into multiple smaller ones to allow continuous flow of data exchange in a decentralized system. Each blocks only contain one transaction which makes the two very similar conceptually but different in structure. A special block called a Milestone is introduced to maintain consensus among peers. The milestone has much higher difficulty than a normal block and requires more time to mine. The connected milestones essentially forms a chain which is very similar to a Nakamoto chain in Bitcoin. Each peers have their own chain connected to the genesis block. Each blocks have three pointers: its previous milestone block, tip block and own block. This structure allows peers to continuously broadcast blocks while having consensus maintained by milestone blocks.

Further reading please refer to our [research paper](https://arxiv.org/abs/1901.02755).

## Video Explanation

{% embed url="https://youtu.be/UEeYkIvl6dA" caption="" %}

