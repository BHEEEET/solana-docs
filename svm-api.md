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
- **Off-Chain services:**
	- Emulate Solana transaction processing protocol off-chain
	- Pro:
		- Useful for simulating transactions
		- fuzzing/ testing
	- Examples:
		- RPC v2 
- **Diet Clients:**
	- Creation of fraud proofs to demonstrate invalid state transitions by supermajority
	- Pro:
		- Lightweight clients can more efficiently without needing to process every transaction or maintain the full blockchain state
- **State Channels:**
	- Allowing parties to exchange singed messages or transactions privately and instantly, settling the final state on-chain only when necessary
	- Examples:
		- token payment channel, and the final  balances of the channel participants are posted on-chain
- **Rollups:**
	- Networks that need to execute transaction or block without the full validator or consensus protocol, the main chain is used for settling "proofs" of the state transitions within the rollup
- **Avalanche Subnet:**
	- Running smart contract on Avalanche subnet 
- **Extended SVM**:
	- Custom extensions that can be plugged in 

Anza's primary application of the SVM is transaction processing within the Agave validator

## Anza's SVM API
`solana-svm`: Rust crate that equips developers with the tools to build SVM projects using components that operate live on Solana mainnet-beta

- The central interface to the SVM resolves around the  `TransactionBatchProcessor` struct: 
```rust
pub struct TransactionBatchProcessor<FG: ForkGraph>{
	// Bank slot (i.e. block)
	slot: Slot,
	// Bank epoch
	epoch: Epoch,
	// SysvarCache is a collection of system variables that are
	// accessible from on-chain programs. It is passed to SVM from
	// client code (e.g. Bank) and forwarded to the MessageProcessor
	pub sysvar_cache: RwLock<SysvarCache>,
	// Programs required for transaction batch processing
	pub program_cache: Arc<RwLock<ProgramCaceh<FG>>>,
	// Builtin programs idsd
	pub builtin_program_ids: RwLock<HashSet<Pubkey>>,
} 
```
![](Pasted%20image%2020250801231913.png)
The main method for processing transaction batches is `load_and_execute_sanitized_transactions`, it requires these arguments:
- `callbacks`: A `TransactionProcessingCallback`trait instance which allows the transaction processor to summon information about accounts, most importantly loading them for transaction execution
- `sanitized_txs`: A list of sanitized Solana transactions
- `check_results`: A list of transaction check results
- `environment`: The runtime environment for transaction batch  processing
- `config`: Configurations for customizing transaction processing behavior
The method returns a `LoadAndExecuteSanitizedTransactionOutput`, which is defined below in more detail

## Transaction Processing Callback
Anyone using the SVM in their own implementation must implement the `TransactionProcessingCallback`trait, so the SVM knows how to access accounts and other needed data during transaction processing
```rust
pub trait TransactionProcessingCallback {
fn get_account_shared_data(&self, pubkey: &Pubkey) -> Option<AccountSharedData>;
fn account_matches_owners(&self,  account: &Pubkey, owners: &[Pubkey]) -> Option<usize>;
fn add_builtin_account(&self, _name: &str, _program_id: &Pubkey) {}
}
```
Because the API accepts a trait implementation, consumers can provide their own custom implementation of loading accounts, caching, and more.

## Transaction Processing Environment
The transaction processor requires consumers to provide values describing the runtime environment to use for processing transaction, by way of the `TransactionProcessingEnvironment`object:
- `blockhash`: The blockhash to use for the transaction batch
- `epoch_total_stake`: The total stake for the current epoch
- `epoch_vote_accounts`: The vote accounts for the current epoch
- `feature_set`: Runtime feature set to use for the transaction batch
- `lamports_per_signature`: Lamports per signature to charge per transaction
- `rent_collector`: Rent collector to use for transaction batch
These environment settings enable a precise andd controlled execution context for transactions, ensuring that they adhere to specific operational standards.

## Transaction Processing Configuration
Consumers can provide various of configurations to adjust the default behavior of the transaction processor through the `TransactionProcessingConfig` argument:
- `account_overriders`: Encapsulates overridden accounts, typically used for transaction simulation
- `compute_budget`: The compute budget to use for transaction execution
- `log_messages_bytes_limit`: The maximum number of bytes that log messages can consume
- `limit_to_load_programs`: Whether to limit the number of programs loaded for the transaction batch
- `recording_config`: Recording capabilities for transaction execution
- `transaction_accounts_lock_limit`: The max number of accounts that a transaction may lock

## Transaction Processing Output
The transaction processor's main API method - `load_and_execute_sanitized_transactions`- returns a `LoadAndExecuteSanitizedTransactionsOutput`object, encapsulating the result of the processed transaction batch
- `error_metrics`: Error metrics for transactions that were processed
- `execute_timings`: Timings for transaction batch execution
- `execution_results`: List of results indicating whether or not a transaction was executed
- `loaded_transactions`: List of loaded transactions that were processed

## Additional Helpers and API Methods
- `[account_loader::validate_fee_payer](https://docs.rs/solana-svm/latest/solana_svm/account_loader/fn.validate_fee_payer.html)`: Check whether the payer account is capable of paying the transaction fee.
- - `[account_rent_state::RentState](https://docs.rs/solana-svm/latest/solana_svm/account_rent_state/enum.RentState.html)`: API for working with an account’s rent state (rent-paying, rent-exempt, etc.)
- - `[program_loader::load_program_with_pubkey](https://docs.rs/solana-svm/latest/solana_svm/program_loader/fn.load_program_with_pubkey.html)`: Load a program by its public key