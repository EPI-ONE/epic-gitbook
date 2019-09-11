# Block

Each block contains only one transaction. There are 2 types of block: normal and milestone.

## Types of Block

### Normal

* Most common type of block
* Approximately 0.001 seconds to mine each normal block

### Milestone

* Difficulty much higher than normal block
* Approximately 10 seconds to mine each Milestone
* A milestone class is a wrapper on a milestone block which contains information about the milestone chain at the current state. It contains the following information:

  * Cumulative chain work
  * Milestone difficulty target for the next milestone
  * Normal block difficulty target for future blocks with R1 linked to it
  * Last timestamp when difficulty target was updated
  * The instance/pointer of the block
  * A set of pointers to all blocks in its level set

  If a milestone is the head of a milestone chain, it also includes

  * A set of pointers to all the blocks pending for verification with regard to the confirmed blocks on the chain



  When a new milestone occurs, it calculates the difficulty target if it is the time for transition. Otherwise, it inherit the difficulty targets from the previous milestone. The set for pending blocks is also moved to it from its previous milestone.

## Structure

<table>
  <thead>
    <tr>
      <th style="text-align:left">Field</th>
      <th style="text-align:center">
        <p>Type</p>
        <p>(Size)</p>
      </th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Version</td>
      <td style="text-align:center">
        <p>uint</p>
        <p>(4 bytes)</p>
      </td>
      <td style="text-align:left">The version of the block.</td>
    </tr>
    <tr>
      <td style="text-align:left">Block Hash</td>
      <td style="text-align:center">Sha256Hash (32 bytes)</td>
      <td style="text-align:left">Hash of all information in the block.</td>
    </tr>
    <tr>
      <td style="text-align:left">Milestone Block Hash</td>
      <td style="text-align:center">Sha256Hash (32 bytes)</td>
      <td style="text-align:left">The Block Hash referencing the current latest milestone.</td>
    </tr>
    <tr>
      <td style="text-align:left">Previous Node Block Hash</td>
      <td style="text-align:center">Sha256Hash (32 bytes)</td>
      <td style="text-align:left">The Block Hash referencing the peer&apos;s previous block in the peer
        chain.</td>
    </tr>
    <tr>
      <td style="text-align:left">Tip Block Hash</td>
      <td style="text-align:center">Sha256Hash (32 bytes)</td>
      <td style="text-align:left">The Block Hash referencing a random tip.</td>
    </tr>
    <tr>
      <td style="text-align:left">Time</td>
      <td style="text-align:center">
        <p>uint</p>
        <p>(4 bytes)</p>
      </td>
      <td style="text-align:left">The Unix time at which the Block Hash is generated.</td>
    </tr>
    <tr>
      <td style="text-align:left">difficultyTarget</td>
      <td style="text-align:center">
        <p>uint</p>
        <p>(4 bytes)</p>
      </td>
      <td style="text-align:left">A shortened version of the Target.</td>
    </tr>
    <tr>
      <td style="text-align:left">Nonce</td>
      <td style="text-align:center">
        <p>uint</p>
        <p>(4 bytes)</p>
      </td>
      <td style="text-align:left">The field that miners change in order to try and get a hash of the block
        (Block Hash) that is below the Target.</td>
    </tr>
  </tbody>
</table>Each block has 3 pointers to 3 different blocks:

* Latest milestone block
* Previous node block in the peer chain
* Tip block

{% hint style="info" %}
The pointers will connect to the Genesis block if pointers to those blocks cannot be found.
{% endhint %}

