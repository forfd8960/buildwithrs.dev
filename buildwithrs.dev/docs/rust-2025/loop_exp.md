---
sidebar_position: 8
---

# Control Flow: `loop` Expression

**1. `loop`**

The `loop` keyword creates an infinite loop that continues until explicitly stopped with `break` [1](https://doc.rust-lang.org/rust-by-example/flow_control/loop.html)[2](https://doc.rust-lang.org/reference/expressions/loop-expr.html).

```rust
fn main() {
    let mut count = 0;

    loop {
        count += 1;

        println!("Count: {}", count);

        if count == 5 {
            println!("Exiting loop");
            break;
        }
    }
}
```

**2. `while`**

The `while` loop continues as long as a condition is true [2](https://doc.rust-lang.org/reference/expressions/loop-expr.html)[3](https://www.programiz.com/rust/while-loop).

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("Number: {}", number);
        number -= 1;
    }

    println!("Liftoff!");
}
```

**3. `for`**

The `for` loop is used to iterate over a sequence of values, such as a range or a collection [2](https://doc.rust-lang.org/reference/expressions/loop-expr.html)[3](https://www.programiz.com/rust/for-loop).

```rust
fn main() {
    // Iterate over a range
    for i in 1..6 { // 6 is exclusive
        println!("Range: {}", i);
    }

    // Iterate over an array
    let array = ["apple", "banana", "cherry"];
    for element in array.iter() {
        println!("Array element: {}", element);
    }
}
```

**4. `loop` with `break` and a return value**

`loop` can return a value using `break`.

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

**5. `while let`**

`while let` is useful for destructuring values from an `Option` or other enum as long as the pattern matches [2](https://doc.rust-lang.org/reference/expressions/loop-expr.html).

```rust
fn main() {
    let mut optional = Some(0);

    while let Some(i) = optional {
        if i > 9 {
            optional = None;
        } else {
            println!("i is {}", i);
            optional = Some(i + 2);
        }
    }
}
```

**6. Looping Through Array with Index**

You can loop through an array and access its index using `enumerate()`.

```rust
fn main() {
    let array = ["apple", "banana", "cherry"];

    for (index, element) in array.iter().enumerate() {
        println!("Index: {}, Element: {}", index, element);
    }
}
```

These examples demonstrate the basic usage of `loop`, `while`, and `for` loops in Rust. Each type of loop serves different purposes, and understanding their use cases is crucial for writing effective Rust code.
