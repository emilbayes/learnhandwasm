## 07. Loops

One important construct that we are missing to process a large data set is a way
to iterate over memory. We already know that we need to read 8 bytes at a
time but have yet to see how to do loops.

## Branching

One of the security features of WASM, in contrast to real assembler, is that you
cannot jump arbitrarily around the code (eg `goto` and `longjmp`). However there
is a model for branching/looping:

```webassembly
(func $looping
  (param $input.ptr i32)
  (param $input.len i32)

  (local $i i32)

  (set_local $i (i32.const 0))

  (loop $continue
    ;; do something at iteration with a f64. Here we drop the value, but may
    ;; sum it or similar
    (drop (i64.load (i64.add (get_local $input.ptr)
                             (get_local $i))))

    (br_if $continue
           ;; tee_local lets sets the value and also returns it in a single
           ;; instruction, so we can do `length < i++`
           (i32.lt_u (tee_local $i (i32.add (get_local $i) (i32.const 8)))
                     (get_local $input.end))))
```

So the `loop` instruction is a label that you can move to with the `(br $label`)
unconditionally, and move to based on a condition with `(br_if $label i32)` as
in the above example. Note how the loop above increments the counter by 8 every
iteration, since that's the byte width of our type.

## Exercise

The exercise now is to set a list of temperatures into memory, convert them
from Fahrenheit to Celsius in-place with load/store from the previous exercises
and read out the contents.
