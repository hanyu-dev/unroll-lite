# unroll-lite

[![Crates.io Version](https://img.shields.io/crates/v/unroll-lite)](https://crates.io/crates/unroll-lite)
[![Documentation](https://img.shields.io/docsrs/unroll-lite)](https://docs.rs/unroll-lite)
[![License](https://img.shields.io/crates/l/unroll-lite)](https://github.com/hanyu-dev/unroll-lite/blob/main/LICENSE)

Unroll loops in const contexts, declarative macro version.

## Examples

The main restriction is that the macro only supports standard exclusive ranges, eg. 0..10 and -5..5, but not ..5 or 0..=10.

```rust
use unroll_lite::unroll;

let mut a = 0;

unroll!(i in 0..5 => {
    a += i
});

assert!(a == 10)
```

A custom step size can be set:

```rust
use unroll_lite::unroll;

let mut v = Vec::new();

unroll!(i in (0..5).step_by(2) => {
    v.push(i)
});

assert!(v == vec![0, 2, 4])
```

Iteration can be reversed:

```rust
use unroll_lite::unroll;

let mut v = Vec::new();

unroll!(i in (0..5).rev() => {
    v.push(i)
});

assert!(v == vec![4, 3, 2, 1, 0])
```

Combinations of step size and reverse iteration are also supported:

```rust
use unroll_lite::unroll;

let mut v = Vec::new();

unroll!(i in (0..10).rev().step_by(4) => {
    v.push(i)
});

assert!(v == vec![9, 5, 1]);
```

```rust
use unroll_lite::unroll;

let mut v = Vec::new();

unroll!(i in (0..10).step_by(4).rev() => {
    v.push(i)
});

assert!(v == vec![8, 4, 0])
```

You can use mutable and wildcard variables as the loop variable, and they act as expected.

```rust
use unroll_lite::unroll;

let mut v = Vec::new();

unroll!(mut i in (0..4) => {
    i *= 2;
    v.push(i)
});

assert!(v == vec![0, 2, 4, 6]);
```

```rust
use unroll_lite::unroll;

let mut a = 0;

unroll!(_ in 0..5 =>
   a += 1
);

assert!(a == 5)
```

The body of the loop can be any statement. This means that the following is legal, even though it is not in a regular for loop.

```rust
use unroll_lite::unroll;

let mut a = 0;

unroll!(_ in 0..5 => a += 1);

assert!(a == 5)
```

```rust
use unroll_lite::unroll;

unsafe fn unsafe_function() {}

unroll!(_ in 0..5 => unsafe {
   unsafe_function()
});
```

If the beginning of the range plus the step overflows the integer behaviour is undefined.

A real world example:

```rust
const fn gen_white_pawn_attacks() -> [u64; 64] {
    let mut masks = [0; 64];

    let mut rank: u8 = 0;
    while rank < 8 {
        let mut file: u8 = 0;
        while file < 8 {
            let index = (rank*8+file) as usize;
            if file != 7 { masks[index] |= (1 << index) >> 7 as u64 }
            if file != 0 { masks[index] |= (1 << index) >> 9 as u64 }

            file += 1;
        }
        rank += 1;
    }

    masks
}
```

After:

```rust
use unroll_lite::unroll;

const fn gen_white_pawn_attacks() -> [u64; 64] {
    let mut masks = [0; 64];

    unroll!(rank in 0..8 => {
        unroll!(file in 0..8 => {
            let index = (rank*8+file) as usize;
            if file != 7 { masks[index] |= (1 << index) >> 7 as u64 }
            if file != 0 { masks[index] |= (1 << index) >> 9 as u64 }
        })
    });

    masks
}
```

## Credits & License

This crates is originally a fork of [`const_for`](https://crates.io/crates/const_for) by Joachim Enggård Nebel, licensed under the MIT License.
