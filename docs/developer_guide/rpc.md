# RPC

We specify commands with their description and parameters.

## Already implemented

### Wallet command line

* status: show the peer status
  * whether the miner is working
  * lastest milestone hash
* start-miner: let the miner to run
* stop-miner: let to miner to stop
* generate-new-key: generate a new ECkey
* create-first-reg: create the first registration for peer chain
  * necessary condition for miner to mine 
* redeem: let miner to create a redemption transaction 
  * parameter: coin value to redeem (0 or "all" to redeem maximum amount) + address for next redemption ("new" to generate new key and use its address)
* set-passphrase: set your wallet passphrase
  * commands related to wallet can only be called after passphrase is successfully set and wallet is logged in
  * parameter: a string of passphrase. Confirmation needed
* change-passphrase: change wallet passphrase
  * parameter: old passphrase and new passphrase
* login: log in to wallet by passphrase
  * parameter: passphrase
* get-balance: get total balance of wallet
* connect: connect to a peer given ip and port
  * parameter: a vector of (ip + port)
* disconnect: disconnect to some or all peers
  * parameter: a vector of (ip + port) or "all"
* create-tx: create a transaction from wallet UTXO
  * parameter: transaction fee + a vector of (coin value + address where the value comes from)
* show-peer: show information of some or all peers
  * parameter: (ip + port) or "all"
  * information: peer id + socket + is valid + is inbound + is fully connected + is available to sync + connected time + block version + local service + app version

### Block explorer

* get-block: get block information by block hash
  * parameter: block hash
* get-new-milestone-since: get milestone blocks since certain milestone block
  * parameter: hash of milestone block
* get-latest-milestone: get the most recent milestone block
* get-levelset: get all the blocks in the querying level set
  * parameter: hash of the milestone block in the level set
* get-levelset-size: get the number of blocks in the querying level set
  * parameter: hash of the milestone block in the level set
* get-vertex: get vertex information by block hash
  * parameter:  block hash
  * a vertex contains the block as well as (height in the DAG + is milestone + redemption status (first redemption or normal redemption or normal block) + a vector a transaction status (valid or invalid) + coin value of miner reward)
* get-milestone: get milestone information by block hash
  * parameter: block hash
  * a milestone contains height in the DAG  + its chain work + block difficulty target + milestone difficulty target + estimated hash rate
* get-peer-chains: get all the peer chains in the DAG
  * a peer chain contains the hash of chain head + peer chain height + the time when the most recent block is mined
* get-forks: get all the milestone chains (forks included) in current DAG
  * a milestone chain contains the most recent milestone + the peer chain which the milestone block belongs to
* get-recent-stat: get statistic information about recent DAG
  * the result contains time from + time to + numbers of blocks, transactions and tps in the time range 
* statistic: get statistic information about DAG from genesis block
  * the result contains current height of DAG + total numbers of blocks and transactions + tps + current mempool size

### Block solver

* send-pow-task: send a pow task to remote server
  * a pow task describes its task id + cycle length + initial nonce and time + search step + raw bytes of the block header + difficulty target
* stop-task: let remote solver stop a pow task by id
  * parameter: task id

## To be implemented

* get-wallet-addr: get all the addresses that holds coins in the wallet
* get-txout: get an available transaction output in the wallet
  * parameter: block hash + transaction index + output index
* get-txout-set: get all tx output information

* validate-addr: check whether the input address is valid

  * parameter: address

* verify-message: verify a single signed message

  * parameter: address + signature + message
* get-mempool-size: get the number of transaction in mempool

* create-multisig: create a n-required-in-m multi-signature address
  * parameter: n required signature + a vector of m addresses
  * a multi-signature address will be returned

