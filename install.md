# Installation
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
brew install llvm@10.0.0
```

We assumes that users set up the OCaml environment with [OPam](https://opam.ocaml.org).
The [LLVM OCaml Binding](https://llvm.moe/ocaml/index.html) will be installed with the following command:
```
opam install llvm.10.0.0
`
