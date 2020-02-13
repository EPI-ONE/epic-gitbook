# Network

## Introduction

All the full nodes in the P2P network should share peers’ network information and consensus data to help other peers connect to more active and honest peers and synchronize blocks in the DAG quickly and efficiently. Light nodes can also obtain consensus data via connecting to full nodes.

P2P network is not included in the consensus rules, so users can use different network protocols alternatively such as some dedicated network for relaying transactions and blocks used by mining pools.

Here we provide a simple network protocol as the reference.

## Join the P2P network

### Peer Discovery

A new full node discover peers mainly in 3 ways:

1. P2P Seeds  

   P2P seeds don’ t participate in the synchronization of consensus data and just share network information of peers. A peer can get other peers’ information via DNS seeds or IP seeds which are saved in the config file. DNS seeds like `pascal.ieda.ust.hk:7877` are currently maintained by EPIC developers and the peer will only connect to the DNS seeds if DNS seeds exist in the config file since they are safer and more stable. The peer will connect to all the seeds when he starts the network, and will disconnect these seeds after exchanging the address information.

2. Network address book  

   Once having an address book, the peer can directly connect to the proper peers via the address book.

3. Specified by the user

   Users can specify the network addresses in the command line like `—connect 127.0.1.1:7877` as the parameters when starting the program or by sending a RPC command to connect to extra peers. 

### Connecting To Peers

In order to finish P2P connection, peers need to complete the version handshake to confirm basic version information of each other like version number, service, time, etc. If the version information matches, the peer will send an acknowledgement message to confirm it. The version handshake finishes if both peers confirm the version messages. Here’s the data structure of the `version_msg` :

* `version_msg`

| Field | Type | Description |
| :--- | :--- | :--- |
| client\_version | int | client version number |
| local\_service | uint64\_t | every bit indicates if the corresponding service is offered |
| nTime | uint64\_t | the time when the version message is created |
| address\_you | NetAddress | the network address of the remote peer in the IP layer, which is used to help the remote peer to identify the actual network address that he uses for sending messages |
| address\_me | NetAddress | the network address of the local peer himself, which is used to help the outbound remote peer to identify if he has connected to the local peer |
| current\_height | uint64\_t | the current height in the DAG of the local peer, which is used to trigger the data synchronization |
| id | uint64\_t | a random number generated for every peer which is used to judge if a peer has connected to himself |
| version\_info | string | some git information like git commit hash, etc. Mainly for debugging |

### Periodical tasks

#### Share address messages

Peers will exchange address message that contains many network addresses of other peers during the version handshake. Moreover, peers will broadcast their own addresses to neighbors periodically and neighbors will relay these address messages.

#### Ping and Pong

A usual way to implement the heart detect.

## Synchronize data

### Initial synchronization

Every time when a peer starts the network, he needs to finish the initial synchronization to obtain the latest blocks.  
The peer starts the initial synchronization by choosing one neightbor that has larger height than himself to synchronize block data \(usually in the form of batches of blocks, which is called `Bundle`\). If the neighbor is not active or the peer reaches the same height a the neighbor, the peer will choose another neighbor to start another round synchronization util his block data is up-to-date enough, which means `current time - current block time < threshold`.

We suggest that a peer should start mining after finishing the initial synchronization in order to decrease the probability of forking.

### Synchronization between two peers

The synchronization process happens between the Peer and DAG modules. Their roles in short, Peer processes messages received from the network, and requests data from DAG accordingly. Detailed workflow is listed below.

_Note:_

* _API functions are marked in the form:_ 

```css
ModuleName#function_name(arguments: [arg1, arg2, ...], return: if any)
```

| Sync Peer \(request blocks\) | Data Peer \(provide blocks\) |
| :--- | :--- |
| **1**\)  In `Peer#process_version_message(args: [VersionMessage])`:   If the sync peer's height is less than the data peer’s, proceed to 2\). |  |
| **2**\)  `DAG#request_inv`, adds the GetINV message to Peer’s message sending queue according to the BlockLocator constructed in 3\), and waits for callback \(List\ from InventoryMessage\):     -- If the list is empty, stop.  -- If the list contains the only the genesis hash, repeat 3\) with a different _fromHash_ and larger _length_. Else, proceed to 7\). |  |
| **3**\)  `DAG#construct_locator(args: [Hash fromHash, long length, Peer peer], return: BlockLocator)` assembles the BlockLocator required by 2\). |  |
|  | **4**\)  When Peer receives the message in 3\), `Peer#process_get_inv_message` and sends the request to DAG, waits for the callback \(List\\) and adds the InventoryMessage according to the callback value to Peer’s message sending queue. |
|  | **5\)** `DAG#assemble_inv(args:[List<Hash> locator], return: List<Hash>)` returns the result as the callback to 4\). |
| **6**\)  When Peer receives the inv in 5., `Peer#process_inv` and returns the list of hashes in inv as the callback to 2\). |  |
| **7**\)  `DAG#request_data(args: [List<Hash>] inv)` adds the GetDataMessage according to the inventory message received in 6\) to the Peer’s message sending queue. Record the order of the inv in `get_data_queue<Futures>`, in which each waits for a callback \(Bundle\): -- If the callback is successful, add the blocks in Bundle to DAG, and proceed with the hash references to the section “Verification”. -- Else, remove the Future in `get_data_queue`. |  |
|  | **8**\)  When Peer receives the GetDataMsg in 7\), `Peer#process_get_data` and sends the request to DAG, waits for the callback \(Bundle\) and adds the callback value to its message sending queue. |
|  | **9\)** `DAG#get_bundle(args: [Hash hash], return: Bundle)` returns the result as the callback to 8\). |
| **10**\)  When Peer receives the Bundle in 9\), `Peer#process_bundle(args: [Bundle])` checks that it corresponds to the head of `getDataQueue`: -- If this is the case, add the Bundle as a callback to 7\). -- Else, put the Bundle in the `lvs_pool` and check if any Bundle in the `lvs_pool` matches the head of `getDataQueue`. If this is the case, repeat `Peer#process_bundle` with this Bundle. |  |

### synchronization messages

Here’s the data structures of the sync messages used above.

`GetInv`

| Field | Type | Description |
| :--- | :--- | :--- |
| locator | vector&lt;Hash&gt; | a sequence of latest milestone hashes, used to help the remote peer to locate the last common milestone of both peers |
| nonce | uint32\_t | a random number used as the ID of a particular Inv message |

`Inv`

| Field | Type | Description |
| :--- | :--- | :--- |
| hashes | vector&lt;Hash&gt; | a sequence of latest milestone hashes, which means the local peer lacks these milestones. If there is only the genesis block hash, then the local peer should send older milestone hashes since they didn’t find common milestone in the `GetInv` message. If the hash list is empty, then the two peers reach the same height |
| nonce | uint32\_t | a random number, the same as the corresponding one in the GetInv message |

`GetData`

| Field | Type | Description |
| :--- | :--- | :--- |
| type | uint8\_t | indicates the type of data , now we supports `level set` \(data that has been confirmed by some milestones\) and `pending set` \(data that hasn’t been confirmed by any milestones\) |
| bundleNonces | vector&lt;Hash&gt; | a list of random numbers, used as the ID of each data set |
| hashes | vector | a list of milestone hashes corresponding to some data sets |

`Bundle`

| Field | Type | Description |
| :--- | :--- | :--- |
| blocks | vector&lt;Block&gt; | block data sets |
| nonce | uint32\_t | the random number corresponding to the GetData message |

