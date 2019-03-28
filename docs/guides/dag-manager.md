# DAG Manager

### Peer Chain



<Insert Image>



- The figure illustrates a typical scenario with 5 different miners. The red boxes indicates the milestones in the main chain while the blue box is a forked milestone. In this structure, blocks mined by the same miner are organized into a *peer chain*. 

### Tip Set

- Tips are blocks which do not have other blocks connected to it. These are simply the latest mined blocks.
- Miners will locate these tips to connect to their own blocks

### Level set

<Insert Image>

- A level set is analogous to the structure of blocks in Bitcoin
- A level set consists of a milestone block and the blocks directly and indirectly pointed by it, until the previous level set.
- Level sets are used for batch synchronization and storage in DB
- The grey shaded area represents the different level sets
- Pending Set
  - The orange rectangle represents blocks that have not been confirmed by any milestone on the longest milestone chain

### Mempool

- In a high TPS environment, Bitcoin's mempool design is not efficient because the same transaction will be mined by multiple miners. This may cause miners to waste their computing power as some of their blocks might have been mined by other earlier miners.

- Attackers might be able to exploit this by initiating transactions with high fees and flood the system with mined transactions

- We use transaction partition to ensure some transactions can only mined by certain miners

- Allowed Distance

  - Let’s say we estimate hash power of previous n blocks to determine the transactions that next block can mine.
    - Add the difficulty value of n blocks according to time order (the same as reference order)
    - Compute the time span of n blocks;
    - Allowed Distance = Coefficient * MaxTarget * Difficulty sum / Previous MS hash rate / time span

- Distance Calculation

  - Calculate the last 4 bytes of each of each hash and convert it to int
  - Add these 2 int, allowing overflow
  - Convert to unsigned int

  $$
  d(a,b) := \mbox{unsigned} (\mbox{int}(a) + \mbox{int}(b))
  $$

  

### Synchronization: Workflow and API

The synchronization process happens between the Peer and DAG modules. Their rolls in short, Peer processes messages received from the network, and requests data from DAG accordingly. Detailed workflow is lists below.

*Note:* 

- _The split "\~\~\~" means changing to the perspective of another peer on the network_
- _API functions are marked in the form:_ 

```css
ModuleName#function_name(arguments: [arg1, arg2, ...], return: if any)
```

------

**1**)  In `Peer#process_version_message(args: [VersionMessage])` , if our height is less than the peer’s, proceed to 2).

**2**)  `DAG#request_inv`, adds the GetBlocksMessage to Peer’s message sending queue according to the BlockLocator constructed in 3), and waits for callback (List\<Hash> from InventoryMessage):

- If the list is empty, stop. 
- If the list contains the only the genesis hash, repeat 3). with a different _fromHash_ and larger _length_.
- Else, proceed to 7).

**3**)  `Peer#construct_locator(args: [Hash fromHash, long length], return: BlockLocator)` assembles the BlockLocator required by 2). 

\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~

**4**)  When Peer receives the message in 3), Peer#process_get_blocks_message and sends the request to DAG, waits for the callback (List\<Hash>) and adds the InventoryMessage according to the callback value to Peer’s message sending queue.

**5)** `DAG#assemble_inv(args:[List<Hash> locator], return: List<Hash>)` returns the result as the callback to 4).

\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~

**6**)  When Peer receives the inv in 5., `Peer#process_inv` and returns the list of hashes in inv as the callback to 2). 

**7**)  `DAG#request_data(args: [List<Hash>] inv)` adds the GetDataMessage according to the inv received in 6. to the peer’s message sending queue. Record the order of the inv in `get_data_queue<Futures>`, in which each waits for a callback (Bundle):

- If the callback is successful, add the blocks in Bundle to Cat., and proceed with the hash references to the section “Verification”.
- Else, remove the Future in `get_data_queue`.

\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~

**8**)  When Peer receives the GetDataMsg in 7), Peer#process_get_data and sends the request to DAG, waits for the callback (Bundle) and adds the callback value to its message sending queue.

**9)** `DAG#get_bundle(args: [Hash hash], return: Bundle)` returns the result as the callback to 8).

\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~

**10**)  When Peer receives the Bundle in 9), `Peer#process_bundle(args: [Bundle])` checks that it corresponds to the head of `getDataQueue`:

- If this is the case, add the Bundle as a callback to 7).
- Else, put the Bundle in the `lvs_pool` and check if any Bundle in the `lvs_pool` matches the head of `getDataQueue`. If this is the case, repeat `Peer#process_bundle` with this Bundle.

### Verification: Workflow and API

There are two types of verification processes: `verify_syntax` and `validate_block`:

- The former one happens in Cat for each block received from the network or from RPC applications. When the syntax check passes, blocks are added to a memory database in Cat. For any non-solid block, Cat sends synchronization requests to DAG. 
- The latter one is triggered in DAG whenever `DAG#update_pendings` finds a block to be a milestone on the main chain.

Two functions are used when adding blocks to Cat with respect to the form upon receival (single Block or Bundle): `addBlockToCat` and `addBundleToCat`. Detailed workflow are listed below.

- `Cat#addBlockToCat(args: [Block block, Peer peer_received_from])`:
  - 1. If `Cat#is_exist(block)`, **return**.
    2. If `Cat#is_solid(block)`, `DAG#verify_syntax`. Else: A
       - Add `block` to OBC
       - If `Cat#contains(block.all_links) == true`, **return**.
       - Else, throw VerificationException and request synchronization.
    3. `Cat#verify_syntax(block)`
    4. `Cat#store_to_memory(block)`
    5. `DAG#update_pendings(&block)`:
       - `MilestoneChainHead#add_block_to_pending(&block)` for every branch
       - If `DAG#is_milestone(&block)` (on main chain), `DAG#validate_block(&block)`:
         - If `block.hash_transaction`, `DAG#validate_tx(block, head.height)`:
           - If `block.tx` is a redemption, `DAG#validate_redemption(block)`
             1. Check that the redemption key is indeed the from the last registration block on the peer chain. If this is not the case, mark `block.tx` to be invalid and **return**.
             2. Check that the redemption value is smaller than the peer chain’s accumulative reward. If this is not the case, mark `block.tx` to be invalid and **return**.
             3. Mark `block.tx` to be valid and **return**.
           - Else, verify that the total value of inputs is not less than the total value of outputs. If this is not the case, mark `block.tx` to be invalid and **return**.
           - For each `TransactionInput input` in `block.tx`,
             1. `UTXO prev_out = Cat#get_utxo(&block.tx)`
             2. If `input.get_script_signature.correctly_spends(utxo, verify_flags)` throws ScriptException, mark `block.tx` to be invalid and **return**.
             3. `Cat#update_utxo(input)`
    6. `Cat#release_blocks_from_OBC(block.hash)`
- `Cat#addBundleToCat(args: [List<Block> blocks, boolean check_solidify])`: 
  - 1. `List<Blocks> sorted = DAG#topological_sort(blocks)`.
    2. For each `block` in `sorted`, perform similar operations as `addBlockToDAG`.

### Orphan Block Container (OBC) Workflow

**Remark:** Need to be integrated with CAT or itself can be a part of CAT

#### Elements in OBC

- For blocks that are temporarily not solid in CAT, we have another shared_ptr stored in OBC that points to the block or block entry. And we say the block is added to OBC.
- When a block b is added into OBC, we do:
- 1. Update OBC_Dict: $\{\mbox{hash } h : S_h  \cup b\}$ if h is a missing hash of b
  2. Add non-solid Degree of b: # of missing hashes

#### Update Procedure

- When adding a block b to DM(dag manager), we see if we can update OBC (Achieved by threading).
- Remove the block in OBC_Dict.get(b.getHash) and decrease their solid degree by 1. For those got non-solid degree 0, remove them from OBC.



### Fork Resolve Process



### Reward Scheme



