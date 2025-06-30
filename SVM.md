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

