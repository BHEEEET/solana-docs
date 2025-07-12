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
- Wallets generate **keypairs**
	- Keypairs are the 64-byte combinations of public and private keys
	- Public key: 
		- unique identifier
		- similar to a filename
		- 32-bytes Base58 encoded string
	- Private key: 
		- secret, password, that grants permission to access and modify the account
		- 32-bytes
- **User account** = data structure that holds information and state related to their interactions with the Solana Blockchain

- Private keys can also be derived from **mnemonic seed phrases,** usually consisting of 12 or 24 words
	- easier backup and recovery
	- multiple keys can be derived from a single seed phrase


- Solana uses **Ed25519 elliptic curve**
	- Small key size
	- Small signature size
	- Fast computation
	- Immunity to many common attacks
	- Each Solana wallet address represents a point on the Ed25519 elliptic curve

Transaction creation:
- User signs transactions with their private key
- The signature is included with the transaction data
- Can be verified by other parties with the public key
- Signature also acts as unique identifier for the transaction

Operations:
- Sending a transaction is the only way to mutate state on Solana
- Any write operation is performed through a transaction
- Transaction are **atomic**
	- Meaning everything the transaction attempts to do  happens or the transaction fails 

### A Solana Transaction
![A Solana Transaction](Pasted%20image%2020250701190319.png)
- A transaction = transaction message
- 4 sections:
	- **Header (3 bytes):**
		- References to the account address list, indicating which accounts must sign the transaction
			- U1: `num_required_signatures`
			- U2: `num_readonly_signed_accounts`, signer accounts that are read-only
			- U3: `num_readonly_unsigned_accounts`, non-signer accounts that are read-only
	- **Account addresses (each account is 32 bytes):**
		- A list of the accounts that will be read/written from during the transaction
		- This way the know in advance which accounts the transaction will touch
			- **Sealevel scheduler**: parallel transactions, if there are no conflicts
				- Conflict arrive when you want to do parallel transaction on the same account, you can't read and write at the same time on 1 account
	- **A recent blockhash (32 bytes):**
		- Used to prevent duplicate and stale transactions
		- Leader creates the blockhash, so **can't be predicted**
		- A recent blockhash expires after 151 blocks (about 1 minute)
		- By default RPCs attempt to forward transactions every 2 secodsd until the transaction is either finalized or the recent blockhash expires
	- **Instructions:**
		- Core of the transaction
		- Each instruction represents a specific operation (e.g., transfer, mint, burn, create accounts, close account)
		- Requirements:
			- Each instruction specifies the program to execute
			- The accounts required
			- Data needed for the instruction's execution
		- Number of instructions is limited by:
			1. It's size, which can be up to 1,323 bytes
			2. Limits around the number of accounts that van be referenced
			3. Limited by complexity of the transaction, measured in compute units (CUs)
				- CUs quantify the computational resources expended in processing the transactions

#### Fees
- **Total fee** = prioritization fee + base fee
- **Prioritization fee** = compute unit price (micro-lamports) x compute unit limit
	- Technically optional, but during periods of high demand for blockspace it becomes necessary.
	- Purpose:
		- Making transaction more economically compelling for validator nodes to includde in their blocks
- **Base fee** = fixed **5000 lamports** per signature cost
- These fees are prices in micro-lamports(millionth of a lamport) per compute unit
- Currently 100% of the prioritization fees are going to the block producer (leader), after the passing of [SIMD-0096](https://github.com/solana-foundation/solana-improvement-documents/blob/main/proposals/0096-reward-collected-priority-fee-in-entirety.md)
- The smallest unit of SOL = lamport = one billionth of a SOL, named after Leslie Lamport, a computer scientist