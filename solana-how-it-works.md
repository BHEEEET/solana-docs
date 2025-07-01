# Solana, How it works
![How Solana works](images/Pasted%20image%2020250701125216.png)


## Key concepts
- Solana is a high performance, low-latency blockchain
- Block time of 400 miliseconds
- Transactions fees fractions of a cent
- *Software should never get in the way of hardware*
- Applications built on this single blockchain, no bridging, no separate chains or fragmentation of liquidity
- Substantial changes to the core Solana Protocol go through formal, transparent process of submitting a Solana Improvement Document (SIMD). SIMDs are then voted on by the network

**Transaction Lifecycle**
1. Users initiate transactions
2. Transactions are sent to current lead block producer (leader)
3. Leader compiles transactions into a block
4. Leader executes and thereby updates the blockchain state
5. This block of transactions is then propagated throughout the network for other validators to execute and confirm

**Six stages**
- Users
- Gulf Stream
- Block Building
- Turbine
- Block verification
- Consensus
![Stages diagram](images/Pasted%20image%2020250701130701.png)

## Users

