# Example
Here is a simple example.
Given a LLVM module `llm`, the following function `count` computes the number of all instructions and
call instructions in the module.

```ocaml
(* scan all instructions in a given block and compute the number of instructions *)
let scan_block (num_all, num_call) blk =
  Llvm.fold_left_instrs                                   (* for each instruction in the block *)
    (fun (num_all, num_call) instr ->
      match Llvm.instr_opcode instr with                  (* count depending on the instruction opcode *)
      | Llvm.Opcode.Call -> (num_all + 1, num_call + 1)
      | _ -> (num_all + 1, num_call))
    (num_all, num_call) blk

(* a naive version *)
let count llm =
  Llvm.fold_left_functions                     (* for each function in the module *)
    (fun (num_all, num_call) f ->
      Llvm.fold_left_blocks                    (* for each block in the function *)
        (fun (num_all, num_call) blk ->
          scan_block (num_all, num_call) blk)  (* scan the block *)
        (num_all, num_call) f)
    (0, 0) llm
```
A simpler and better version will be as folows:

```ocaml
let count llm =
  Llvm.fold_left_functions (Llvm.fold_left_blocks scan_block) (0, 0) llm
```
