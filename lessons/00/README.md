# 00. Welcome!

Welcome to the handcrafting WebAssembly workshop. WebAssembly is a byte-code for
the web, making it possible for compilers to target at common instruction format.

Despite the name it is note quite assembly and not quite web, since it does not
natively allow you to access any Web APIs (yet!) and is more high-level than
assembly, as you don't directly interact with the CPUs instruction set, but
an idealised instruction set, that the runtime can very simply and efficiently
translate to machine code.

As we will see when we look at the instruction format, WebAssembly is an
extremely secure sandbox, where access to anything outside the VM is granted
explicitly.

To get started you need a recent version of Node (8+), which you can check with:

```sh
node -v
```

You also need a simple compiler that can translate the text-format to the binary
format. For the beginning of this workshop we will also use an additional tool
that will wrap you WASM binary module in a Javascript module for easy usage.

Install the following tools:

```sh
npm install --global wat2wasm wat2js
```

You can verify the tools with:

```sh
wat2wasm --help
wat2js --help
```

`wat2wasm` is the compiler and `wat2js` depends on this compiler, and in
addition will produce a very simple Javascript module from your WAT code.

**Remark**: The above tools are userland tools, the first made by me and the
second made by @mafintosh. There is however an official toolchain called WABT,
WebAssembly Binary Toolkit, but it can be tricky to compile this C++ based
toolchain, so for this workshop we will work with the simplified versions.
In practise the `wat2wasm` from npm is the native WABT `wat2wasm` compiled to
WebAssembly. So you will be compiling WebAssembly with WebAssembly. Meta!
[Official WABT repository](https://github.com/WebAssembly/wabt)

[**Exercise 01**](../01)
