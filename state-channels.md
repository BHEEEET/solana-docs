source: https://ethereum.org/en/developers/docs/scaling/state-channels/ 

---
# State Channels
State channels allow participants to securely transact off-chain while keeping interaction with mainnet at a minimum. Channel peers can conduct an arbitrary number of off-chain transactions while only submitting two on-chain transaction to open and close the channel. This allows for extremely high transaction throughput and results in lower costs for users.

## What are channels?
Public blockchains face, such as Ethereum, face scalability challenges dues to their distributed architecture:
- On-chain transactions must be executed by all nodes

Nodes have to be able to handle the volume of transactions in a block, imposing a limit on transaction throughput to keep the network decentralized

Blockchains channels solve this problem by allowing users to interact off-chain while still relying on the security of the main chain for final settlement.

 **Channels** are simple **peer-to-peer protocols** that allow two parties to make many transactions between themselves and then **only post the final results to the blockchain**
- Uses cryptography to demonstrate that the summary data they generate is truly the result of a valid set of intermediate transactions
- A "multisig" smart contract ensures the transactions are signed by the correct parties
- State changes are executed and validated by interested parties => minimizing computation on Ethereum's execution layer
- Each channel is managed by a multisig contract

To open a channel:
1. Participants deploy channel contract on-chain
2. Deposit funds into it
3. Both parties collectively sign a state update to initialize the channel's state
4. Now they can transact quickly and freely off-chain

To close the channel:
1. Participants submit the last agreed-upon state of the channel onchain
2. Smart contract distributes the locked funds according to each participant's balance in the channel's final state

Examples: payment channels, state channels

## Payment channels
A payment channel is best described as a "two-way ledger" collectively maintained by two users. 

The ledgers initial balance is the sum of deposits locked into the on-chain contract during the channel opening phase
- Can be performed instantaneously and without involvement of the blockchain
- Except for an initial one-time on-chain creation and eventual closing of the channel

Updates to ledger balance:
- requires  approval of all parties in the channel

Payment channel participants can conduct unlimited amount of instant, feeless transactions between each other, as long as the net sum of their transfers does not exceed the deposited tokens.

## State channels

**WORK FURTHER HERE**