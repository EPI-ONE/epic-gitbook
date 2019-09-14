# Architecture

This article provides a brief explanation of our coding project, how we implement the consensus mechanism in `C++`. This project builds on many existing tools which have been developed, perfected and test by the community in the past decade. 

-   The [bitcoin](https://github.com/bitcoin/bitcoin) project has provided all the essential utilities such as the definition and operations of big unsigned integers, hash functions, Merkle tree, serialization, etc. We take most of these codes, some of which with minor modifications.
-   [libsecp256k1](https://github.com/bitcoin-core/secp256k1) is a great C library for EC operations on curve secp256k1.
-   [libevent](https://github.com/libevent/libevent) is an event notification library, on which we build out net communication. 
-   John Tromp wrote a nice proof-of-work algorithm [cukoo](https://github.com/tromp/cuckoo), which we decide to use. His code are included in our project with minor modification. 
-   [gRPC](https://github.com/grpc/grpc) provide a modern, open source, high-performance remote procedure call (RPC) framework. 
-   Facebook Database Engineering Team develops and maintains a persistent key-value store library [RocksDB](https://github.com/facebook/rocksdb) for fast storage. 
-   We use [googletest](https://github.com/google/googletest) as the testing framework to test our project. 
-   We use the fast, header-only `C++` logging library [spdlog](https://github.com/gabime/spdlog).
-   We use the lightweight `C++` option parser library [cxxopts](https://github.com/jarro2783/cxxopts).
-   We use the header-only library [cpptoml](https://github.com/skystrife/cpptoml) for parsing [TOML](https://github.com/toml-lang/toml) configuration files.

Part of the Bitcoin codes (some with modification), cukoo (with modification), (spdlog , cxxopts, cpptoml) headers are included in our source. Other dependencies, e.g, secp256k1, libevent, googletest, gRPC, RocksDB need to be installed as instructed [here](getting-started/install). 

Building on all these excellent existing tools, there is still substantial work required to implement the consensus mechanism. What we have done in this coding project is to design and implement how to send and receive transactions/blocks over a peer-to-peer network, how to process the received information and reach consensus. The source codes are organized in folders to reflect the architecture of this coding project. 

## Peer-to-peer network

Source codes are organized in two subfolders `net` and `peer` for this purpose. In `net`, we implement `address`, `address_manager`, `connection` and `connection_manager`, with the latter two utilizing the library libevent. This part is highly independent of the project. In fact, it is only used by `peer`  for sending and receiving messages. 

We implement a version of peer-to-peer network by defining `peer`, `peer_manager` and `task`, with all the codes organized in the subfoler `net`. To reduce the snowball effect, we have modified the peer-to-peer protocol. The main idea is to add a counter to a message to count how many times it's been relayed. The large this number, the more likely many peers may have already got it. Thus we decrease its probability to be relayed further. We provide some detailed numerical experiment for setting the parameters to strike a balance between fast propagation and network bandwidth waste. 

## Messages

Basic elements of information being sent/receive between peers are: `transaction`, `block`, `bundle` (collection of blocks). For this purpose, we define the base class called `NetMessage`, from which `transaction` and `block` are inherited. There are other types of messages, e.g. `PING`,`PONG`, `VERSION_ACK` all defined in this module. 

## Storage

Information (`transaction`, `blocks`) arrive through the peer-to-peer network. Codes in the subfolder storage provide unified management for how to store such information in memory and DB/file. When a block arrives (or being generated locally by the miner himself), it is stored in memory. All the manipulation are done via smart pointers until it is written into DB/file and deleted from the memory.  

A block is **solid** if all the three blocks it points to are present locally either in memory or in DB/file (likely with a small probability). New blocks need to go through basic syntactic check and **solidity** check. If a block is solid, its pointer will be placed in the container `blockCashe_` (a hashmap), otherwise in `obc` (orphan block container). `obc` records all the non-solid blocks, and continuously check the solidity of all its block whenever new blocks arrive. It is implemented in `obc.h(cpp)`. Pointers to solid blocks will also be passed to DAG, and once consensus is reached, blocks will be archived in DB/file in the unit of `level set` and deleted from memory. 

In this folder, we have `db_wrapper.h(cpp)`, `db.h(cpp)` and `file_util.h(cpp)` for DB and file handling. 

## Consensus

As the core of the whole project, this module handles consensus by performing the following tasks. 

- Organize all solid blocks in our sDAG
- Sort out all the level sets along each milestone chain. 
- Identify the longest milestone chain.
- Very each transaction, ordered by depth-first-search preordering according to blocks and Merkel tree within each block, and maintain UTXOs. 

Each block is referred to as a "vertex" in the sDAG, for that we wrote `vertex.h(cpp)`. To be precise, all the milestone blocks form a tree just like the Bitcoins, though only those on the longest chain (deepest branch of the tree) are deemed to be the "true" milestone. Note that each other branch on the tree also forms a chain. So we wrote a `chains.h(cpp)` to manage all the `chain.h(cpp)`. The ledger, essentially `utxo.h(cpp)` is computed by the instant of `chain.h(cpp)` which is the longest. Since this is a cryptocurrency application, the `coin.h` is for basic currency calculation and is primarily used by `transaction.h(cpp)`.