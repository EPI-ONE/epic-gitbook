# Transaction

The structure of a transaction in EPI is very similar to Bitcoin's transaction. The difference lies in the ScriptSig in which EPI has simpler script. There are 3 types of transaction: normal, registration and redemption.

## Types of Transaction

### Normal

#### General Structure

<table>
  <thead>
    <tr>
      <th style="text-align:left">Field</th>
      <th style="text-align:center">
        <p>Type</p>
        <p>(size)</p>
      </th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Version</td>
      <td style="text-align:center">
        <p>int</p>
        <p>(4 bytes)</p>
      </td>
      <td style="text-align:left">Version of transaction data.</td>
    </tr>
    <tr>
      <td style="text-align:left">Input Count</td>
      <td style="text-align:center">
        <p>VarInt</p>
        <p>(1-9 bytes)</p>
      </td>
      <td style="text-align:left">Indicates the upcoming number of inputs</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="transaction.md#transaction-input">Input(s) </a>
      </td>
      <td style="text-align:center">Refer to Transaction Input</td>
      <td style="text-align:left">Details of a transaction input.</td>
    </tr>
    <tr>
      <td style="text-align:left">Output Count</td>
      <td style="text-align:center">
        <p>VarInt</p>
        <p>(1-9 bytes)</p>
      </td>
      <td style="text-align:left">Upcoming number of outputs.</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="transaction.md#transaction-output">Output(s)</a>
      </td>
      <td style="text-align:center">Refer to Transaction Output</td>
      <td style="text-align:left">Details of a transaction output.</td>
    </tr>
  </tbody>
</table>#### Transaction Input

<table>
  <thead>
    <tr>
      <th style="text-align:left">Field</th>
      <th style="text-align:center">
        <p>Type</p>
        <p>(size)</p>
      </th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">TXID</td>
      <td style="text-align:center">
        <p>uint256</p>
        <p>(32 bytes)</p>
      </td>
      <td style="text-align:left">Refer to an existing transaction</td>
    </tr>
    <tr>
      <td style="text-align:left">VOUT</td>
      <td style="text-align:center">
        <p>int</p>
        <p>(4 bytes)</p>
      </td>
      <td style="text-align:left">Select one of the output from that transaction.</td>
    </tr>
    <tr>
      <td style="text-align:left">ScriptSig Size</td>
      <td style="text-align:center">
        <p>VarInt</p>
        <p>(1-9 bytes)</p>
      </td>
      <td style="text-align:left">Upcoming size of the unlocking code.</td>
    </tr>
    <tr>
      <td style="text-align:left">ScriptSig</td>
      <td style="text-align:center">
        <p>Script</p>
        <p>(Variable)</p>
      </td>
      <td style="text-align:left">A script that unlocks the input.</td>
    </tr>
  </tbody>
</table>#### Transaction Output

<table>
  <thead>
    <tr>
      <th style="text-align:left">Field</th>
      <th style="text-align:center">Type (size)</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Value</td>
      <td style="text-align:center">
        <p>long</p>
        <p>(8 bytes)</p>
      </td>
      <td style="text-align:left">The value of the output in Xunit.</td>
    </tr>
    <tr>
      <td style="text-align:left">ScriptPubKey Size</td>
      <td style="text-align:center">
        <p>VarInt</p>
        <p>(1-9 bytes)</p>
      </td>
      <td style="text-align:left">Upcoming size of the locking code.</td>
    </tr>
    <tr>
      <td style="text-align:left">ScriptPubKey</td>
      <td style="text-align:center">
        <p>Script</p>
        <p>(Variable)</p>
      </td>
      <td style="text-align:left">A script that locks the output.</td>
    </tr>
  </tbody>
</table>### Registration and Redemption

Registration and Redemption are transactions that are automatically generated when peers decide to create a peer chain and redeem the block rewards. The purpose of Registration and Redemption transaction is to replace coinbase transaction used by Bitcoin. With huge mining traffic, the use of coinbase transaction generated for every new transaction is not efficient for data storage.

#### Registration

* The first block each miner creates will be a Registration transaction
* Distinguished by transaction input
* Does not contain transaction signature

#### Redemption

* Redemption transaction is created when miners want to claim the accumulated rewards
* Contain transaction input
* `Input = Total number of Block Reward + Transaction fee`  between the Redemption block and the latest Registration block \(Refer to Reward Scheme\)
* This will sum up the total reward that the miner will receive
* It requires the miner's private key to create this transaction

## Status

* Conditions for an invalid transaction:
  * Missing input or output for redemption of reward
  * Redeem value greater than reward
  * Double redemption
  * Number of sigOps is greater than `MAX_BLOCK_SIGOPS`
  * Signature failed
  * Transaction fees to large \(greater than `MAX_MONEY`\)
* Invalid transaction will catch exception and terminate
* Only valid transaction will be allowed to accumulate rewardThis is an enum that describes the underlying reason the transaction was created. It's useful for rendering wallet GUIs more appropriately.

