# Solana Virtual Machine (SVM)
source: https://www.anza.xyz/blog/the-solana-ebpf-virtual-machine 
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
- Registries are small memory locations (storage slots in RAM) that store data currently being operated on.
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

### Instruction layout
- Instruction layout covers how the instructions are encoded in the VM and what each bit means
- 64-bit slots, instructions can occupy one or two slots, indicated by the op code of the first slot

```
+-------+--------+---------+---------+--------+-----------+
| class | opcode | dst reg | src reg | offset | immediate |
|  0..3 |   3..8 |   8..12 |  12..16 | 16..32 |    32..64 | Bits
+-------+--------+---------+---------+--------+-----------+
low byte                                          high byte
```

```
| bit index | meaning
| --------- | -------
| 0..=2     | instruction class
| 3..=7     | operation code
| 8..=11    | destination register
| 12..=15   | source register
| 16..=31   | offset
| 32..=63   | immediate
```
- `Instruction class`: Identifies the type of instruction (arithmetic, memory access, etc.)
- `Op code `: The specific operation
- `Destination register`: The register where the result of the operation will be stored
- `Source register`: The register where the operation input data is sourced
- `Offset`: Used for memory access or jump offsets
- `Immediate`: Constant values

#### Op codes
- Row labels = upper four bits
- Column labels = lower four bits
```

|     | ·0   |  ·1  | ·2  |  ·3  |   ·4   |  ·5  |   ·6    |   ·7   |  ·8  |  ·9   |  ·A  |  ·B   |   ·C   |  ·D   |   ·E    |   ·F   |
|----:|:----:|:----:|:---:|:----:|:------:|:----:|:-------:|:------:|:----:|:-----:|:----:|:-----:|:------:|:-----:|:-------:|:------:|
|  0· | lddw |  -   |  -  |  -   | add32  |  ja  |    -    | add64  |  -   |   -   |  -   |   -   | add32  |   -   |    -    | add64  |
|  1· |  -   |  -   |  -  |  -   | sub32  | jeq  |    -    | sub64  | lddw |   -   |  -   |   -   | sub32  |  jeq  |    -    | sub64  |
|  2· |  -   |  -   |  -  |  -   | mul32  | jgt  |    -    | mul64  |  -   |   -   |  -   |   -   | mul32  |  jgt  |    -    | mul64  |
|  3· |  -   |  -   |  -  |  -   | div32  | jge  | uhmul64 | div64  |  -   |   -   |  -   |   -   | div32  |  jge  | uhmul64 | div64  |
|  4· |  -   |  -   |  -  |  -   |  or32  | jset | udiv32  |  or64  |  -   |   -   |  -   |   -   |  or32  | jset  | udiv32  |  or64  |
|  5· |  -   |  -   |  -  |  -   | and32  | jne  | udiv64  | and64  |  -   |   -   |  -   |   -   | and32  |  jne  | udiv64  | and64  |
|  6· |  -   | ldxw | stw | stxw | lsh32  | jsgt | urem32  | lsh64  |  -   | ldxh  | sth  | stxh  | lsh32  | jsgt  | urem32  | lsh64  |
|  7· |  -   | ldxb | stb | stxb | rsh32  | jsge | urem64  | rsh64  |  -   | ldxdw | stdw | stxdw | rsh32  | jsge  | urem64  | rsh64  |
|  8· |  -   |  -   |  -  |  -   | neg32  | call | lmul32  | neg64  |  -   |   -   |  -   |   -   |   -    | callx | lmul32  |   -    |
|  9· |  -   |  -   |  -  |  -   | mod32  | exit | lmul64  | mod64  |  -   |   -   |  -   |   -   | mod32  |   -   | lmul64  | mod64  |
|  A· |  -   |  -   |  -  |  -   | xor32  | jlt  |    -    | xor64  |  -   |   -   |  -   |   -   | xor32  |  jlt  |    -    | xor64  |
|  B· |  -   |  -   |  -  |  -   | mov32  | jle  | shmul64 | mov64  |  -   |   -   |  -   |   -   | mov32  |  jle  | shmul64 | mov64  |
|  C· |  -   |  -   |  -  |  -   | arsh32 | jslt | sdiv32  | arsh64 |  -   |   -   |  -   |   -   | arsh32 | jslt  | sdiv32  | arsh64 |
|  D· |  -   |  -   |  -  |  -   |   le   | jsle | sdiv64  |   -    |  -   |   -   |  -   |   -   |   be   | jsle  | sdiv64  |   -    |
|  E· |  -   |  -   |  -  |  -   |   -    |  -   | srem32  |   -    |  -   |   -   |  -   |   -   |   -    |   -   | srem32  |   -    |
|  F· |  -   |  -   |  -  |  -   |   -    |  -   | srem64  | hor64  |  -   |   -   |  -   |   -   |   -    |   -   | srem64  |   -    |

```

Instruction HEX formatting
```
CALL = 0x85
0x85 → binary 1000 0101

Upper bits: 0x8 → row 8·

Lower bits: 0x5 → column ·5
``` 


#### Instructions by class

## GET BACK TO THIS!!!!!

#### Verification
- Defines rules by verifying an eBPF program

## Solana VM Builtin Programs (Loaders)
- The functions within a compiled eBPF program are read into a *function registry* when the binary is loaded by the eBPF VM
- But the rBPF VM supports *builtin program*, which also have their own function registries
- Solana native programs = provide access to  functions built into the execution environment
    - When a instruction for a builtin program is called, it does not load dand execute some compiled BPF program, it calss a function that is built into the runtime
    - These programs do not exist on-chain, but have an on-chain placeholder at their corresponding address
    - It's code ships with the Solana runtime
    - Built into the VM

- Loader = VM-level builtin program, which gives executable access to the set of builtin functions
- Builtin functions = There are many tupes of VM builtin functions, but the primary provided by the VM loader are system calls (syscalls)
- Syscalls = allow executing eBPF programs to call fucntions outsidde their compiled bytecode, built into the virtual machine, to do things as:
    - Print log messages `sol_log`
    - Invoke other Solana programs (CPI) `sol_invoke`
    - Perform cryptographic arithmetic operations 
- Solana protocol syscall interfaces repo: https://github.com/anza-xyz/agave/blob/a5b3c2b7e0aa761d8d8107c19b8fb8c2a9bdaa78/sdk/program/src/syscalls/definitions.rs
- Changes to the syscalls are governed by the Solana Improvement Documents (SIMD) process
- The Agave validator implements all Solana syscalls on the BPF loader
    - Validators can be lagging if they don't upgrade after a SIMD
- BPF Loader = runtime builtin programs

## Program Execution
- the rBPF VM library can execute eBPF programs in 2 ways
    - Interpreter
    - JIT compilation to x86_64 machine code
