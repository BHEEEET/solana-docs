source: https://www.anza.xyz/blog/anzas-new-svm-api

---

# Anza SVM API
## Defining SVM
Common  definitions:
- Transaction pipeline, from validator runtime to program execution
- lower-level eBPF virtual machine for executing programs

Bank:
- Tracks client accounts and the progress of on-chain programs (state)
- A single bank relates to a block produced by a single leader 
	- each bank except for the genesis bank points back to a parent bank
- Bank then attempts to process the transactions into a block

Leaders:
- Leaders are know ahead of time = leader schedule
- Solana Protocol forwards transactions to the next leader in the leader schedule
- If a leader can't include a transaction in its current slot, it is forwarded to the next leader
- Leaders use a bank to pack blocks
- While other nodes will replay a block
	- Attempting to rebuild a matching instance of a Bank and thus validate a block

Transactions:
- Transactions are processed in batches
- Each transaction contains one or more instructions, each of which targets specific programs
- The concept of read-only and "writable" accounts allows parallel account data access across Bank instances
- Before transactions are processed, they undergo a "sanitization"
	- various checks
	- extracts processing information
		- deduplication of account keys
- Ance all the accounts are loaded, the SVM loads the executable program for each instruction and provisions and eBPF virtual machine to execute the programs, returning the result
- To avoid redundant translation from byte code to machine code, a cache mechanism is used, which stores the translated program until it absoletely needs to be recomputed (or the cache is full)
- The SVM interface used by the Bank has been decoupled from Bank significantly

## Opportunities for SVM
- Off-Chain services:
- Diet Clients:
- State Channels:
- Rollups:
- Avalanche Subnet:
- Extended SVM