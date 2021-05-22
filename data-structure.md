# LLVM Data Structure and APIs
You will mainly use API functions in the [Llvm module](https://llvm.moe/ocaml/Llvm.html).
Here we provide instructions about using LLVM APIs which can be needed for basic usage.

#### (Almost) Everything is llvalue
In LLVM IR, type `llvalue` is the most important data type.
Functions, instructions, global variables, constants, and many more are all `llvalue`s.
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

#### Handling `icmp`
Given an `icmp` instruction, API `icmp_predicate` returns the predicate with a type of
[Icmp.t](https://llvm.moe/ocaml/Llvm.Icmp.html). 
