# Generating LLVM IR with Debug Information
LLVM IR can be generated from C source code using [clang](https://clang.llvm.org), a C language family front-end for LLVM.
For example, you can generate LLVM IR files for the example C programs under `test` with the following commands:
```
cd test
clang -c -emit-llvm -S -fno-discard-value-names -Xclang -disable-O0-optnone -o example1.tmp.ll -g example1.c
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

For the details of LLVM IR, see this [document](https://llvm.org/docs/LangRef.html).
