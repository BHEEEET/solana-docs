# SVM
## SBPF Virtual Machine rBPF
repo: https://github.com/anza-xyz/sbpf 

- It's a crate
- SBPF = Rust implementation of a Extended Berkley Packet Filter (eBPF) virtual machine
- BPF is an assembly-like language
    - Helps filter packets in the kernel, to avoid useless copies in user-space
    - eBPF = faster version with more features
- Runs in user-space, not in the kernel
- Virtual machine cointains:
    - Interpreter
    - X86_64 JIT-compiler for eBPF programs
    - assembler
    - disassembler
    - verifier
- Validator runtime also runs in the node's user-space

### SVM ISA
- SBPF VM implements the Solana Virtual Machine Instruction Set Architecture (SVM ISA) and is used by the Agave validator
- Firedancer has a different VM that adheres to the SVM ISA

## Berkley Packet Filter
- Solana programs are compiled to BPF format

- BPF format leverages *qualifiers*
    - qualifiers: `tcpdump 'tcp port 80'` `tcp` -> protocol qualifier, `port 80` -> port qualifier

- Solana also uses discrimintators as qualifiers
    - discriminator: Small piece of data at the start of an accounts or instruction that identifies what type it is, like a tag 
    - Discriminators are not enforced
    - Used for in program use

- BPF programs use these qualifiers to define conditions under which packets are captured or dropped. 

- BPF eventual evolved into Extended Berkley Packer Filter (eBPF) format, which is what Solana's LLVM compiles to.

- eBPF allows programs to configure a restricted instruction set and contraints specifically desinged for safe kernel execution.
=> This is useful for Solana programs, since it prevents the validators from crashing and creates a consistent environment for all programs

- Solana programs are mostly written in Rust => compiled to eBPF by the Solana platforom tools https://github.com/anza-xyz/platform-tools

- However programs can also be written in Zig, C or Assembly.

- They just need to respect the proper eBPF format, which respects the eBPF restructions imposed by the Solana VM

## Solana rBPF ISA

- eBPF allows platforms, such as VM, to impose tight instruction set and constraints for eBPF programs
- ISA = Instruction Set Architecture, is the thing that defines the constraints
- All Solana VMs must adhere to ISA in order to be compliant with the Solana Protocol

###  Registers
- Supported by the rBPF VM
- 64 bits wide
- Registries are small memory locations (storage slots in the CPU) that store data currently being operated on.
- ISA defines 10 General-Purpose Registers (GPRs)

```
|  name | feature set | kind            | Solana ABI 
|-------:|:-----------|:----------------|:----------
|  `r0` | all         | GPR             | Return value
|  `r1` | all         | GPR             | Argument 0 
|  `r2` | all         | GPR             | Argument 1 
|  `r3` | all         | GPR             | Argument 2 
|  `r4` | all         | GPR             | Argument 3 
|  `r5` | all         | GPR             | Argument 4 <br/>or stack spill ptr
|  `r6` | all         | GPR             | Call-preserved 
|  `r7` | all         | GPR             | Call-preserved 
|  `r8` | all         | GPR             | Call-preserved 
|  `r9` | all         | GPR             | Call-preserved 
| `r10` | all         | Frame pointer   | System register 
| `r11` | from v2     | Stack pointer   | System register 
|  `pc` | all         | Program counter | Hidden register
```
- `r0` holds the function's return data/value
- r1` through `r5` store function arguments, and `r5' can also store "spillover" data, which is represented by a pointer to some stack data
- `r6` through `r9` are call-preserved registers, which means that their values are temporarly preserved across functions calls.
- `r10`references to the current stack frame in memory (local variables)
-  `r11` is a stack pointer, that keeps track of where the top of the stack is
- `pc' is the program counter, that holds the address of the current instruction being executed
