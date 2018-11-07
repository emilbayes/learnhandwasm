# 01. Your first WASM Module

Without further ado, here is the first WASM module we will be looking at:

```webassembly
(module $arithmetic
  (func $i32.add (export "i32.add")
    (param $a i32)
    (param $b i32)
    (result i32)
    (i32.add
      (get_local $a)
      (get_local $b))))
```

The above module is the "hello world" of WebAssembly. Before we start writing
WAT ourselves we need to learn a bit about the syntax and grammar of WAT. You
should paste the module into a file so we can play around with it. I will be
calling mine for `arithmetic.wat`.

The first thing we notice is the `(module $arithmetic)` instruction.
`$arithmetic` is a label that the compiler will be turning into a simple
integer that will count up. In WASM you can (at present) have a single module
per file, and the name is only for debugging, as it will be replaced with said
integer. However `wat2wasm` can extract labels and place them into a special
debugging section, like source maps from Javascript, so they can be consumed
by disassemblers and debuggers.

Try running the following command:

```
echo '(module $arithmetic)' | wat2wasm --dump-module -
```

What it does is that it compiles the module from `stdin` and prints the
annotated byte-code. As you can see it's is not very interesting, simply
printing a `\0` byte, the ASCII string `asm` and then a version identifier.

Now try with the following command:

```
echo '(module $arithmetic)' | wat2wasm --dump-module --debug-names -
```

You can see a lot more information has been added to the binary module.
In particular notice line `0000012` with our name `arithmetic` encoded.
During the workshop it is recommended to always append the `--debug-names`
flag, when compiling your modules. However, for production it is recommended
to not add these symbols, as they can incur a significant increase in module
size, just like you wouldn't ship source maps to production.

A bit of a nitty gritty detour, but head on to the next exercise where we will
look at `$i32.add` function:

[**Exercise 02**](../02)
