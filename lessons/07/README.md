# 07. A detour on debugging

So far we have trusted our ability to write correct code, but soon we will go
into more looping, which is notorious for causing bugs, so I want to do a detour
on how I do debugging in WASM. It's a bit lengthy, so skip ahead if you want to
get back to writing code.

## Disassembly

Current Firefox and Chrome will disassemble WASM into the linear format in their
DevTools, but in slightly different ways. They will also allow you to inspect
the stack of values when hitting a breakpoint and step though the instructions.

Chrome 69 Debugger:
```webassembly
func $i32.add (param i32 i32) (result i32)
  get_local 0
  get_local 1
  i32.add
end
```

Firefox 63 Debugger:
```webassembly
(module
  (type $type0 (func (param i32 i32) (result i32)))
  (export "i32.add" (func $i32.add))
  (func $i32.add (;0;) (param $var0 i32) (param $var1 i32) (result i32)
    get_local $var0
    get_local $var1
    i32.add
  )
)
```

## Logging

However sometimes you just want to log a value, whether as a separate
instruction or inline in a call to see a value at a particular point in time.

For this I always define `$t.log: [t] -> []` and `$t.log_tee: [t] -> [t]`.
The former will log any one value eg.

```webassembly
(call $i32.log (get_local $counter))
```

or log inline by inspecting the value and putting it back on the stack:

```webassembly
(set_local $hash
           (call $i64.log_tee
                 (i64.mul (i64.const 0xffff)
                          (call $i64.log_tee
                                (get_local $hash))
```

Notice how this will first log the initial value of `$hash` before
multiplication and the product after multiplication, before updating `$hash`,
by simply injecting `(call $i64.log expr)`. This will always work since WASM
functions (as of current) can only push a single value on the stack at a time.

### How it's done

```webassembly
(module $debug_helpers
  (func $i32.log (import "debug" "log") (param i32))
  (func $i32.log_tee (import "debug" "log_tee") (param i32) (result i32))
  ;; No i64 interop with JS yet - but maybe coming with WebAssembly BigInt
  ;; So we can instead fake this by splitting the i64 into two i32 limbs,
  ;; however these are WASM functions using i32x2.log:
  (func $i32x2.log (import "debug" "log") (param i32) (param i32))
  (func $f32.log (import "debug" "log") (param f32))
  (func $f32.log_tee (import "debug" "log_tee") (param f32) (result f32))
  (func $f64.log (import "debug" "log") (param f64))
  (func $f64.log_tee (import "debug" "log_tee") (param f64) (result f64))

  ;; i64 logging by splitting into two i32 limbs
  (func $i64.log
    (param $0 i64)
    (call $i32x2.log
      ;; Upper limb
      (i32.wrap/i64
        (i64.shl (get_local $0)
                 (i64.const 32)))
      ;; Lower limb
      (i32.wrap/i64 (get_local $0))))

  (func $i64.log_tee
    (param $0 i64)
    (result i64)
    (call $i64.log (get_local $0))
    (return (get_local $0))))
```

```js
var wasmModule = require('./debug-module')

wasmModule({
  import: {
    debug: {
      log (...args) {
        console.stack(...args)
      },
      log_tee (arg) {
        console.stack(arg)
        return arg
      }
    }
  }
})
```

## Asserting

Another good technique to catch bugs is being very defensive when coding, which
can be done by making assertions of certain facts in a program. In Node.js we
have the `assert` module providing a suite of utilities for asserting various
facts. In WASM there's a `unreachable` instructions which will cause an
unconditional trap, meaning the execution of the program stops, just like throw
in Javascript. The trap will be propagated to the Javascript engine and
converted to an `RuntimeError`.

For assertions I use the following snippet:

```
(func $assert (param i32) (if (i32.eqz (get_local 0)) (unreachable)))
```

When the trap is converted to a Javascript error, the error message will have
`assert` in the stack trace, letting me know that the error was caused by an
assertions.

Important to note, just as in Javascript, is that any mutations done
