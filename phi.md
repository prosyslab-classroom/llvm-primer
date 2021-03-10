# Working with LLVM PHI Nodes
For optimization purposes, compilers often implement their intermediate representation in
static single assignment (SSA) form and LLVM IR is no different. In SSA form, a variable is assigned and
updated at exactly one code point. If a variable in the source code has multiple assignments,
these assignments are split into separate variables in the LLVM IR and then merged back together.
We call this merge point a **phi node**.

To illustrate phi nodes, consider the following code:
```c
int f() {
  int y = input();
  int x = 0;
  if (y < 1) {
    x++;
  } else {
    x--;
  }
  return x;
}
```
```llvm
entry:
  %call = call i32 (...) @input()
  %cmp = icmp slt i32 %call, 1
  br i1 %cmp, label %then, label %else

then:                             ; preds = %entry
  %inc = add nsw i32 0, 1
  br label %if.end

else:                             ; preds = %entry
  %dec = add nsw i32 0, -1
  br label %end

end:                        ; preds = %else, %then
  %x = phi i32 [ %inc, %then ], [ %dec, %else ]
  ret i32 %x
}
```

Depending on the value of `y`, we either take the left branch and execute `x++`, or the right branch and execute `x--`.
In the corresponding LLVM IR, this update on `x` is split into two variables `%inc` and `%dec`. `%x` is assigned
after the branch executes with the `phi` instruction; abstractly, `phi i32 [ %inc, %then ], [ %dec, %else ]` says
assign `%inc` to `%x` if the then branch is taken, or `%dec` to `%x` if the else branch was taken.

Notice that all phi nodes must be at the top of a block. Therefore, you should be very careful when you change LLVM IR code
not to violate this invariant.
