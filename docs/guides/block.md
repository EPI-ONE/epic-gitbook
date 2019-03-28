# Block

Each block contains only one transaction. There are 4 types of block:

- Normal

  - Most common type of block
  - Approximately 0.001 seconds to mine each normal block

- Milestone

  - Distinguished by its difficulty

  - Approximately 10 seconds to mine each Milestone

  - A milestone class is a wrapper on a milestone block which contains information about the milestone chain at the current state. It includes the following data structures:

    - Cumulative chain work
    - Milestone difficulty target for the next milestone
    - Normal block difficulty target for future blocks with R1 linked to it
    - Last timestamp when difficulty target was updated
    - The instance/pointer of the block header
    - A set of pointers to all blocks in its level set

    If a milestone is the head of a milestone chain, it also includes

    - A set of pointers to all the blocks pending for verification with regard to the confirmed blocks on the chain

    When a new milestone occurs, it calculates the difficulty target if it is the time for transition. Otherwise, it inherit the difficulty targets from the previous milestone. The set for pending blocks is also moved to it from its previous milestone.

- Registration

  - The first block each miner creates will be a Registration transaction
  - Distinguished by transaction input
  - Does not contain transaction signature

- Redemption

  - Redemption transaction is created when miners want to claim the accumulated rewards
  - Contain transaction input
  - `Input = Total number of Block Reward + Transaction fee`  between the Redemption block and the latest Registration block (Refer to Reward Scheme)
  - This will sum up the total reward that the miner will receive
  - It requires the miner's private key to create this transaction

The use of Registration and Redemption blocks allow EPI to eliminate the use of coinbase transaction used by Bitcoin. With huge mining traffic, the use of coinbase transaction is not sustainable. 



### Structure

**Header**

<Insert Image>

Each block has 3 pointers to 3 different blocks: 

- Latest milestone block
- Previous node block in the peer chain
- Tip block

The pointers will connect to the Genesis block if those blocks cannot be found.

**Content** (Refer to Transaction)



###  



