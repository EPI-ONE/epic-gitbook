# Mining and Consensus

Here we will present what a 'good' or valid block or transaction look like in epic network. First the common structures of blocks and transactions will be mentioned, while it is just the first step towards their validity. Furthermore, we will discuss how to construct valid blocks for miner and how to verify transactions according to our consensus to update the ledger.

## Transactions

Our transaction contains two list of inputs and outputs respectively, following the bitcoin structure that every input of a transaction comes from an output of a transaction that is valid in the ledger as known as unspent transaction output (UTXO). Epic network introduces a script-like mechanism called transaction assembly (TASM) to achieve transaction validity checking.

### Basic structure

#### TASM

The building element in TASM system is called listing, which contains a vector of operations and a binary data array. Listings can be concatenated simply by appending operations and data respectively. For all listings we want to execute, what we do called preprocessing is to append two operations FAILURE and SUCCESS. These operations are indicators to the result of execution.

Running operations of a listing in TASM system follows a non-sequential order from head to tail in the operation vector. Each operation gives an index of where the next operation locates. Therefore, consecutive operations are executed until we get the index of FAILURE or SUCCESS, leading to the final output of the listing.

| Field of listings      | Description                      | Comments                                                     |
| :--------------------- | -------------------------------- | ------------------------------------------------------------ |
| a vector of operations | indeed a vector of uint8_t       | A byte represents an operation. <br>It represents one lambda in the instruction set. |
| binary data            | a vector of bytes to be executed | This field usually contains important data lik user signature and encoded wallet address. |

#### Input and Output

Despite that TASM is different from script in bitcoin, transaction inputs and outputs are very much similar. Both inputs and outputs hold an incomplete listing that needs to be concatenated before execution. This feature is almost the same as bitcoin to help users authorize their spending. 

To make everyone understand inputs and outputs, there is a lot more to explain. But readers who are familiar with bitcoin may skip this paragraph. An outputs consists of a listing and the value of the coins it gives, while an input is made up of a listing and an outpoint, specifying which output it comes from. In other words, the outpoint is just the collection of the index of the target output, the index of the transaction and the hash of the block containing the output. It is straight forward to accurately locate the target output using information above. 

| Field of outpoints | Description | Comments                                               |
| :----------------- | ----------- | ------------------------------------------------------ |
| block hash         | uint256     | target to the block where the output is                |
| transaction index  | uint32_t    | the index of transaction in the block where the output |
| output index       | uint32_t    | the index of output in the transaction                 |

| Field of inputs | Description                          | Comments                             |
| :-------------- | ------------------------------------ | ------------------------------------ |
| an outpoint     | block hash + tx index + output index | location of the corresponding output |
| a listing       | a tasm listing                       | containing signature                 |

| Field of outputs | Description            | Comments                                        |
| :--------------- | ---------------------- | ----------------------------------------------- |
| value of coins   | coin (indeed uint64_t) |                                                 |
| a listing        | a tasm listing         | containing the address where receives the coins |

### Transactions 

A transaction is made of a list of inputs and outputs. It is noted that the proof of ownership (user signature) of money is embedded in the input listing. We can view a transaction as its inputs bring money in and outputs spend money out. Then we say the output value to be the sum of total money outputs giving away and the input value to be the sum of total money from inputs, more explicitly, the sum of total money giving away from corresponding outputs which is pointed by the outpoints of intputs. Hence output value should not exceed input value. Conversely, their difference is served as the transaction fee giving to the miner who packs the transaction.

In epic network, we have two types of transaction namely normal transaction and registration transaction. The former one is just trivial representation of money transferring, while the later one serves as redeeming money for rewarding miners for their consistent hard working to solve hashes.

#### Registrations

Registration transactions are typically for miner usage. It is the signal that epic network gives to miner to claim that they want to redeem the mining reward. The mechanism of registration is part of reward scheme and will be thoroughly discussed in the section [Registration blocks](#reg-block). For now we would like to briefly introduce the requirement of its structure. 

Like the coinbase in bitcoin, the money of a registration transaction comes from nowhere. However, the protocol of epic network asks that it must have one and only one input, the outpoint of which has *unconnected* transaction index and output index. Here we say an index is unconnected if there is no target to refer to. Furthermore, an unconnected index is represented by the maximum number in the index range. At the same time, the hash in the outpoint must be the hash of the block which contains the last registration transaction the miner sent. For the first registration of a typical miner, the hash should be the hash of zero as required since there is no last registration.

One more thing to note for each registration is that it redeems rewards from last registration through its input and specify the address that next registration awards to and amount of money to redeem through its output. For example, suppose a miner has already sent a registration with address A. After some mining, he wants to redeem rewards and claim next redemption address to be B. So considering registration in this case, the input should have the listing with corresponding signature of address A and the output should have the listing with address B. 

## Blocks

A block is the combination of a block header and a merkle tree of transaction, following bitcoin structure. However, the chain of blocks now becomes the directed acyclic graph (DAG) of blocks in the design of epic network. As a result, blocks have in total three reference hashes in the header instead of only one. A detailed specification is presenting as following.

| Field                | Description | Comments                                     |
| -------------------- | ----------- | -------------------------------------------- |
| version              | uint16_t    | specifying epic network version number       |
| milestone block hash | uint256     | points to milestone block                    |
| previous block hash  | uint256     | points to the previous block the miner mined |
| tip block hash       | uint256     | points to a block not mined by the miner     |
| merkle root hash     | uint256     | the root of merkle tree of transactions      |
| time stamp           | uint32_t    | time stamp in Unix format                    |
| difficult target     | uint32_t    | compact number of arithmetic uint256         |
| nonce                | uint32_t    | certificate of POW of the block              |

### Registration blocks  {#reg-block}

A block is classified by the first transaction it contains, to be more precise, whether the transaction is a registration. If so, we call the block registration block. Recall that in designing of epic network, every miner has a chain of blocks they mined called *peer chain*. It is noted in the structured DAG that for each block created by the same miner, its previous block hash points to the last block created by the miner. Moreover, the first block in its peer chain must be registration block as a declaration of its coming. And the reward for miners accumulates from zero after first registration block. Miners are packing normal transactions constantly while making registration transaction sporadically to get rewards. 

### Making a syntactically valid block

 In spite of fields requirement, it is the syntactically valid blocks that can only update ledger, similar to the integrity check. We say that a block is syntactically valid if it agrees with the following rules:

1. It does not exist *locally* [^locally].
2. It has the correct syntax.
3. It is solid, i.e., all of its ancestors are already added to our *local DAG* [^localDAG].
4. It reaches the minimum required difficulty target.
5. It does not link to a milestone that is too old, i.e., less than a certain height determined by `Param`.

[^locally]: When we say a block exists locally, it means that it is either in our local DAG (explained below) or in the OBC.
[^localDAG]: The local DAG refers to the union of the blocks stored in DB and in the following containers: the level sets and pending sets in all the chains in `DAGManager`.

For any miner who is engaged in the epic network, there are a lot more details to follow. A full specification list of syntactical validity is listed below:

- version number matches with network version
- milestone block hash points to a milestone block whose height is not less than the height of current best milestone chain minus a certain number called punctuality threshold
- previous block hash points to the last block the miner mined
- tip block hash points to blocks not mined by the miner, which is recommended to be as new as possible
- merkle tree of transaction has no duplicated transactions and its hash is computed correctly
- time stamp should not be too advanced in the future. For miners' simplicity, just set it be the time when a block is solved 
- difficult target matches with target required by the milestone block it points to.
- nonce must be the correct proof of work
- total size of the block does not exceed the maximum size
- it is required that each transaction
  - has no empty inputs or outputs
  - has positive output values
  - does not conflict with each other, i.e. double spending on the same output

## Reaching Consensus

### Verification Detail

There are two types of verification processes: the verification of the basic syntax, PoW, solidity, and punctuality of the block (we call it *online verification*), and the verification of the transactions contained in the block (namely *offline verification*). The former one is performed on every `Block` received from the network.

For any non-solid block, we add it to the OBC and send a synchronization request to the peer who sends it to us if any of its ancestors does not exist locally. If a block passes the syntax check, we say it is syntactically valid. All syntactically valid blocks are added to the pending set of each milestone chain managed by DAG for the subsequent verification of transactions. For the ones that are not syntactically valid, we simply discard them. 

The offline verification is triggered whenever we find a `Block` to be a milestone on some chain at the end of the online verification. We use the post-order depth-first search to assign a unique order to the blocks in the pending set that precede the milestone block, which form the level set of this milestone. For each block in the level set, the offline verification determines whether the transactions it contains are valid, and thus we can update the UTXO ledger accordingly. The offline verification on each milestone chain in DAG are independent, and the level set being verified is stored as a vector of `Vertex` on its corresponding chain.

### Making a valid transaction

When a syntactically valid block is going through the verification procedure, its transactions update ledger one by one. At this stage, the integrity of transactions is achieved but not the validity, for example, some inputs may come from spent outputs, or some transactions have been processed already. In this section we will focus on steps of transaction verification. There are a lot more rules to follow. Rules for sortition and registration transactions can be well-followed in mining procedure. But those for normal transactions are out of miners' control.

#### Sortition

Sortition is an implementation of transaction assignments mechanism. The aim is to reduce the probability that  one transaction is packed by different miners so that epic network can be more efficient to process as many transactions as possible. And the idea from sortition is restrict the range of transactions which is available to miners. Imagine that total transaction pool is a paper with many dots randomly drawn. The dots are just transactions. Sortition is like tearing up the paper into pieces and assigning each miner a piece. The dots on each pieces represent transactions the miner can mine.

However, our sortition is not as perfect as the above analogy. In epic network, the estimated hashing power (numbers of hashing per unit time) of the miner is the size of the piece of paper. And the latest block hash is where the piece locates. They are not that accurate but a rough estimation. To put in a mathematical way, we will define a distance measure between transactions and blocks and find a threshold for the distance, which is proportional to the hashing power of miner. 

Moreover, a miner must mine a certain number of blocks without normal transactions, say one hundred, after his first registration block. This gives the first estimation of his hashing power. And it updates iteratively according to recent one hundred blocks. Then the miner should calculate the distance, given by an xor operation on the hashes of target transaction and latest block in epic network. A transaction can be mined only if the corresponding distance is within the threshold. Note that two hashes both appear randomly so that every transaction can be packed by some miner and collision is reduced. 

Related validation rules are:

- the block contains normal transactions before reaching height where first estimation of hashing power is
- the distance exceed corresponding threshold

#### Offline verification

Besides sortition, we say a transaction is valid in the offline verification if it meets the following requirements:

- If it is the first registration, then it is valid automatically.
- If it is a redemption:
  1. the last redemption/registration block is not redeemed;
  2. its input contains a valid signature; and
  3. its redemption value in the output is not greater than the peer chain’s accumulative reward.
- If it is a normal transaction:
  1. the total value of inputs is not less than the total value of outputs;
  2. the total value of inputs is not greater than the maximum allowed money;
  3. hash in an outpoint of inputs points to an unspent output; and
  4. every input contains a valid signature.

### Workflow graph

The following graph is an abstract work flow chart for both the online and offline (in the yellow shade) verification.

![verification workflow](/home/oersted/epic-gitbook/docs/.gitbook/assets/veri_workflow.png)