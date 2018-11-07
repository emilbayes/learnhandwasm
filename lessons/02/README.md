# 02. Arithmetic ... WAT?

Recall the module from the last exercise:

```webassembly
;; arithmetic.wat
(module $arithmetic
  (func $i32.add (export "i32.add")
    (param $a i32)
    (param $b i32)
    (result i32)
    (i32.add
      (get_local $a)
      (get_local $b))))
```

As you may already have guessed this function simply adds two numbers together,
however there are a number of details in the function.

## S-Expressions

As of yet I haven't even explained how the syntax works. This syntax
is called an "S-Expression", and the format is `(operator operands*)`, where
operator is the instruction to be called, and `operands` can be zero or more
"arguments" to be passed. A simple, familiar example would be `2 + 2 * 4`, which
as an S-Expression would be `(+ 2 (* 2 4))`. As you can see precedence is
extremely clear, the syntax is very easy to parse for computers and humans (with
a bit of practise) and there is no ambiguity. As a mental model for executing
this code in your head, you can start at the innermost expression, execute it
and replace the result, while you move outwards. See the following:

```
2 * 2 + 4 * 8 / 2
(+ (* 2 2) (* 4 (/ 8 2))) ;; evaluating the division
(+ (* 2 2) (* 4 4)) ;; evaluating both multiplications
(+ 4 16) ;; evaluating the addition
20
```

Also note that whitespace is ignored, so you can use it to organise your code.
Single line comments are made with `;;` while multiline comments are `(; ... ;)`
making it easy to remove entire instructions.

## The `$i32.add` function

The first instruction you encounter in the module is `(func $i32.add ...)`.
The `$i32.add` is a label just like `$arithmetic` was, and will be replaced with
a counting integer when compiled, for efficiency reasons. The name is not
arbitrarily chosen. In WASM there is the convention of naming instructions
`type.operator`, and I tend to follow the same convention, to make my
modules more readable.

Next we declare `(export "i32.add")`, which is to say that the function should
be publicly accessible outside the module. This means that in a minute, we will
be able to call the function from Javascript or other WASM modules, under that
name.

In WASM, part of the sandboxing model is that modules should be able to be
"validated" at compile time (which happens in Node or the Browser) for soundness.
This implicates that everything in WASM is strongly typed. There are very few
builtin types, namely `i32`, `i64`, `f32` and `f64`. The first two are integers
of different bit lengths. We have 32-bit integers in Javascript (yes it's true!)
which we will see later. We do have 64-bit integers through BigInt, but this
type is not yet accessible from Javascript. Both of these integer types are
sign-agnostic, meaning that whether a number is positive or negative depends on
the operators it is used with. We will have a look at this later.

Lastly we have single-precision and double-precision floating point numbers,
where double-precision is exactly the `Number` you know from Javascript. A new
type that is coming to WASM is the `v128` vectorised type. This will allow
writing even faster algorithms by utilising SIMD (Single Instruction, Multiple
Data), however this has not completely laded yet. You can see all proposals here
https://github.com/WebAssembly/proposals.

You may ask now where booleans, strings, arrays and objects are in all of this,
and the short answer is that they don't exist! Or said in another way, we will
need to write our own primitives as we go along.

Back to typing. As you can see the strict typing means we need to explicitly
define the "signature" of our function, under the restrictions of the provided
types. This will be familiar to anyone who has worked with a strictly typed
language, such as TypeScript, Java or Go. AS you can see we define to parameters
`$a` and `$b`, both being `i32` and a result of `i32` also. Like all other
labels, these will be replaced by a counting integer when compiled. We may as
well have written:

```webassembly
;; arithmetic.wat
(module
  (func (;0;) (export "i32.add")
    (param (;0;) i32)
    (param (;1;) i32)
    (result i32)
    (i32.add
      (get_local 0)
      (get_local 1))))
```

We finally arrive at the body of our function, with the `i32.add` instruction
(which the function is named after). A surprising detail is that variable need
to be explicitly accessed with the `(get_local $label)` instruction. Remember
the mental model from earlier and you should be able to execute the code.

You may question why there is no return statement, despite we having said the
result of the function is `i32`. We could have written `(result (i32.add ...))`
to make it explicit, but this just works. A hand-wavy explanation for now is
that the last value returned, will become the return value of the function.

## Compiling!

Now try compiling the WASM module:

```
wat2wasm arithemtic.wat
```

Also have a look at the dumped module, with and without the debug names.
How many instructions is the actual function body? What is the type section?
What happened to the function signature?

When you have answered these questions, try changing the function.

* Try changing one of the parameters to `f64`
* Try doing two additions in the body next to each other. What does the compiler
  say?

When done, continue to the next exercise:

[**Exercise 03**](../03)
