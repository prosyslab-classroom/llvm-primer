# LLVM Primer
This article provides basic information about LLVM especially,
- Students who take courses of [ProSys Lab](prosys.kaist.ac.kr) and to homework using LLVM
- OCaml programmers who want to use [LLVM OCaml Binding](https://llvm.moe/ocaml/index.html)

## 1. Installation
Make sure you have the correct version of Clang/LLVM installed in the system.
```
clang --version
opt --version
llvm-config --version
```
For the details, see the installation document for [Linux](https://apt.llvm.org).
If you use MacOS and Homebrew, run the following commands:
```
brew update
brew install llvm@9.0.0
```

We assumes that users set up the OCaml environment with [OPam](https://opam.ocaml.org). 
The [LLVM OCaml Binding](https://llvm.moe/ocaml/index.html) will be installed with the following command:
```
opam install llvm.9.0.0
```

## 2. Generating LLVM IR with Debug Information
LLVM IR can be generated from C source code using [clang](https://clang.llvm.org), a C language family front-end for LLVM.
For example, you can generate LLVM IR files for the example C programs under `test` with the following commands:
```
cd test
clang -c -emit-llvm -S -fno-discard-value-names -Xclang -disable-O0-optnone -o exampl1.tmp.ll -g example1.c
```
Each commandline option of `clang` has the following meaning:
- `-c`: only run preprocess, compile, and assemble steps
- `-emit-llvm`: generate LLVM IR rather than machine code
- `-S`: generate LLVM IR in a human-readable format
- `-fno-discard-value-names`: prevent `clang` from discarding names in the generated LLVM IR
- `-O0`: disable `clang`'s optimizations
- `-Xclang -disable-O0-optnone`: allow `opt` to perform further optimizations (see below)
- `-o`: specify output file name
- `-g`: generate LLVM IR with debug information that will provide source code information (e.g., line and column numbers) 

Then, you will get the final representation of the target prgram in LLVM IR with the following command:
```
opt -mem2reg -S -o example1.ll example1.tmp.ll
```
[Opt](http://llvm.org/docs/CommandGuide/opt.html) is a tool from LLVM that performs
optimizations on LLVM IR. We use `opt` to perform an optimization, called `mem2reg`.
The `mem2reg` optimization pass promotes the variables from memory to registers, thereby
generating manageable and simpler LLVM programs for the homework. You can compare the two LLVM IR files
before (`example1.tmp.ll`) and after (`example1.ll`) the optimization and see how the code looks like.
Note that if you do not specify `-Xclang -disable-O0-optnone` in the first step, the `-mem2reg`
optimization pass is not performed.

## 3. LLVM Data Structure and APIs
You will mainly use API functions in the [Llvm module](https://llvm.moe/ocaml/Llvm.html).
Here we provide instructions about using LLVM APIs which can be needed for basic usage.

#### (Almost) Everything is llvalue
In LLVM IR, type `llvalue` is the most important data type.
Functions, instructions, global variables, constants, and much more are all `llvalue`s.
You can use API function `classify_value` to classify a given value with a type of `llvalue`.
The output of the API is a value with a type of [ValueKind.t](https://llvm.moe/ocaml/Llvm.ValueKind.html).
Because of this structure, some API functions may return `None` when an operation is invalid for a certain  `llvalue`.
For example, API function `int64_of_const` returns the corresponding 64bit integer if the `llvalue` actually
represents an integer constant in the input program, otherwise `None` (e.g., applying `int64_of_const` to an assignment).
Notice that `llvalue` is the OCaml counterpart of the [Value](https://llvm.org/doxygen/classllvm_1_1Value.html)
class of the LLVM source code in C++.

#### Instruction as Variable
In the LLVM framework, the object for an assignment instruction (e.g., call, binary operator, icmp, etc) also
represents the variable it defines (i.e. its left-hand side). Because of SSA representation,
each instruction uniquely determines a variable. For example,
```llvm
%x = add nsw i32 10, 20
%inc = add nsw i32 %x, 1
```
You will use the objects for the assignment instructions themselves to refer to variables `%x` and `%inc`,
in your implementation. Also the objects will be used as keys of the memory data structure in the homework.
The `operand` API will be useful when you want to access subexpressions of each instruction.

Suppose `inst_x` refers to the first instruction,
`Llvm.operand inst_x 0` and `Llvm.operand inst_x 1` will return the first (constant `10`) and second operand (constant `20`)
of the instruction, respectively. Each of them is a `llvalue`-type value.
Notice that the first operand of `%inc = add nsw i32 %x, 1` is variable `%x` that also represents the first instruction.

#### Instructions within a Block
A block is a sequence of non-jump instructions.
API function `instr_begin` returns the first position of a given basic block
and `instr_succ` returns the next position of a given instruction.
A position is either "before an instruction" or "at the end of block".

