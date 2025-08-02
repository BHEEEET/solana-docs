source: https://github.com/anza-xyz/agave/blob/master/svm/doc/spec.md

---
# Solana Virtual  Machine specification
## System Context
![](Pasted%20image%2020250802012027.png)
Bank is external to  the SVM.
- It creates an SVM
- Submits transactions for execution
- receives results of transaction execution from SVM

### Interfaces
The interface to SVM is represented by the `transaction_processor::TransactionBatchProcessors`struct. 
To create a `TransactionBatchProcessor`object the client needs to specify the `slot`, `epoch`, and `program_cache`
- `slot: Slot`is a u64 value representing the ordinal number of a particular blockchain state in context of which transactions are executed. This value is used to locate the on-chain program versions used in the transaction execution. "block number"
- `epoch: Epoch`is a u64 value representing the ordinal number of a Solana epoch, in which the slot was created. This is another index used to locate the on-chain programs used in the execution of transactions in batch
In addition, `TransactionBatchProcessor`needs an instance of `SysvarCache` and a set of pubkeys of builting program IDs

The main entry point to the SVM is the method `load_and_execute_sanitized_transactions

idem [svm-api]

## Functional Model
Steps of `load_and_execute_sanitized_transactions`:
1. Steps of preparation for execution
	- Filter executable program accounts and build program accounts map
	- Add builtin programs to program accounts map
	- Replenish program cache using the program accounts map
		- Gather all required programs to load from the cache
		- Lock the global program cache and initialize the local program cache
		- Perform loading tasks to load all required programs from the cache: loading, verifying, and compiling (where necessary)
		- A helper module - `program_loader`- provides utilities for loading programs from on-chain, namely `load_program_with_pubkey`
2. Load accounts (call to `load_accounts`function)
	- For each `SVMTransaction` and `TransactionCheckResult`:
		- Calculate the number of signatures in transaction and its cost
		- Call `load_transaction_accounts`
			- The function is interwined with the struct `SVMInstruction`
			- Load accounts from accounts DB
			- Extract data from accounts
			- Verify if we've reached the maximum  account  data size
			- Validate the fee payer and loaded accounts
			- Validate the programs accounts that have been loaded and checks if they are builtin programs
			- 
3. 

