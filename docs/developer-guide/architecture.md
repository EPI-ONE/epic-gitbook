# Architecture

This article provides a brief explanation of our coding project, how we implement the consensus mechanism in C++.

## Dependencies

The EPIC project is built on many existing tools which have been developed, perfected and tested by the community in the past decade.

* The [bitcoin](https://github.com/bitcoin/bitcoin) project has provided all the essential utilities such as the definition and operations of big unsigned integers, hash functions, Merkle tree, serialization, etc. We take most of these code, some of which with minor modifications.
* [libsecp256k1](https://github.com/bitcoin-core/secp256k1) is a great C library for EC operations on curve secp256k1.
* [libevent](https://github.com/libevent/libevent) is an event notification library. Net communication is built on top of it.
* John Tromp wrote a nice proof-of-work algorithm [cukoo](https://github.com/tromp/cuckoo). The code are included in our project with minor modification.
* [gRPC](https://github.com/grpc/grpc) provides a modern, open source, high-performance remote procedure call \(RPC\) framework.
* Facebook Database Engineering Team develops and maintains a fast persistent key-value storage library [RocksDB](https://github.com/facebook/rocksdb). It is used as an search index to the file storage system.
* [googletest](https://github.com/google/googletest) is the unit-test framework of our project.
* The fast, header-only C++ library [spdlog](https://github.com/gabime/spdlog) is the logging utility we use.
* We use the lightweight C++ option parser library [cxxopts](https://github.com/jarro2783/cxxopts) and header-only library [cpptoml](https://github.com/skystrife/cpptoml) for parsing [TOML](https://github.com/toml-lang/toml) configuration files.

Part of the Bitcoin code \(some with modification\), cukoo \(with modification\), \(spdlog , cxxopts, cpptoml\) headers are included in our source. Other dependencies, e.g, secp256k1, libevent, googletest, gRPC, RocksDB need to be installed as instructed [here](https://github.com/EPI-ONE/epic-gitbook/tree/7e1f9021085a98e11098e7177486cefd63054eba/docs/developer_guide/getting-started/install/README.md).

Building on all these excellent existing tools, there is still substantial work required to implement the consensus mechanism. What we have done in the project is to design and implement how to send and receive transactions/blocks over a peer-to-peer network, how to process the received messages from network then reach consensus. The source code is organized in folders to reflect the architecture of this coding project.

## Peer-to-peer network

Related code is organized in two subfolders `net` and `peer` for this purpose. In `net`, we implement `address`, `address_manager`, `connection` and `connection_manager`, with the latter two utilizing the library libevent. This part is highly independent of the project. In fact, it is only used by `peer` for sending and receiving messages.

We implement a version of peer-to-peer network by defining `peer`, `peer_manager` and `task`, with all the code organized in the subfoler `net`. To reduce the snowball effect, we have modified the peer-to-peer protocol. The main idea is to add a counter to a message to count how many times it's been relayed. The large this number, the more likely many peers may have already got it. Thus we decrease its probability to be relayed further. We provide some detailed numerical experiments for setting the parameters to strike a balance between fast propagation and network bandwidth waste.

## Messages

Most of messages being sent/receive between peers are: `transaction`, `block`, `bundle` \(collection of blocks\). For this purpose, we define the base class called `NetMessage`, which `transaction` and `block` inherite from. Howeverm, there are a lot of other types of messages, e.g. `PING`,`PONG`, `VERSION_ACK` all defined in this module in order to complete synchronization.

## Storage

Information \(`transaction`, `blocks`\) arrives through the peer-to-peer network. Code in the subfolder storage provides unified management for how to store such information in memory and DB/file. When a block arrives \(or being generated locally by the miner himself\), it is stored in memory. All the manipulations are done via smart pointers until it is written into DB/file and deleted from the memory.

A block is _solid_ if all the three blocks it points to are present locally either in memory or in DB/file \(likely with a small probability\). New blocks need to go through basic syntactic check and solidity check. If a block is solid, its pointer will be placed in the container `blockCashe_` \(a hashmap\), otherwise in `obc` \(orphan block container\). `obc` records all the non-solid blocks, and continuously check the solidity of all its block whenever new blocks arrive. It is implemented in `obc.h(cpp)`. Pointers to solid blocks will also be passed to DAG, and once consensus is reached, blocks will be archived in DB/file in the unit of `level set` and deleted from memory.

In this folder, we have `db_wrapper.h(cpp)`, `db.h(cpp)` and `file_util.h(cpp)` for DB and file handling.

## Consensus

As the core of the whole project, this module handles consensus by performing the following tasks:

* Organize all solid blocks in our sDAG.
* Sort out all the level sets along each milestone chain.
* Identify the longest milestone chain.
* Verify each transaction, ordered by post-order depth-first-search according to sDAG and Merkel tree within each block, and maintain UTXOs.

Each block is referred to as a "vertex" in the sDAG, for that we wrote `vertex.h(cpp)`. To be precise, all the milestone blocks form a tree just like the Bitcoin, though only those on the longest chain \(deepest branch of the tree\) are deemed to be the "true" milestone. Note that each other branch on the tree also forms a chain. So we wrote a `chains.h(cpp)` to manage all the `chain.h(cpp)`. The ledger, essentially `utxo.h(cpp)` is computed by the instant of `chain.h(cpp)` which is the longest. Since this is a cryptocurrency application, the `coin.h` is for basic currency calculation and is primarily used by `transaction.h(cpp)`.

