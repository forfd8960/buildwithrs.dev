---
sidebar_position: 9
---

# Rust Ownership basics

Ownership is Rust's most unique feature, enabling memory safety without a garbage collector [1](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html)[3](https://lucacorbucci.medium.com/understanding-ownership-in-rust-5baf5b919246)[5](https://dev.to/francescoxx/ownership-in-rust-57j2). It governs how a Rust program manages memory through a set of rules checked by the compiler [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html). Violating these rules will result in a compilation error [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

## The Stack and the Heap

Many programming languages abstract away the stack and the heap, but in Rust, understanding them is crucial because they affect how the language behaves [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

*   **Stack**: Stores values in the order it gets them and removes the values in the opposite order (Last In, First Out or LIFO) [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html). All data stored on the stack must have a known, fixed size [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html). Adding data is called "pushing," and removing data is called "popping" [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).
*   **Heap**: When you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html). This is called "allocating" [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html). Because the pointer to the heap is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

## Ownership Rules

1.  Each value in Rust has an owner [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).
2.  There can only be one owner at a time [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).
3.  When the owner goes out of scope, the value will be dropped [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

## Variable Scope

A scope is the range within a program for which an item is valid [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

```rust
fn main() {
    {                      // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
        println!("{}", s);
    }                      // this scope is now over, and s is no longer valid
}
```

`s` is valid from the point it is declared until the end of the current scope [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

## The `String` Type

The `String` type is needed to illustrate ownership rules because it is more complex than the data types covered in Chapter 3 of the Rust Book [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html). String literals are hardcoded into the text of a program and are immutable. The `String` type, on the other hand, is allocated on the heap and can grow in size [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

```rust
fn main() {
    let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() appends a literal to a String

    println!("{}", s); // This will print `hello, world!`
}
```

In this case, the string can be mutated. The difference is that the string literal is known at compile time and is stored on the stack, while the `String` type is allocated on the heap and can grow in size.

## Ownership and Dropping

When a variable goes out of scope, Rust automatically calls the `drop` function and frees the memory [2](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html).

```rust
fn main() {
    let s = String::from("hello");
} // Here, s goes out of scope and `drop` is called. The memory is now freed.
```

## Move

To ensure there is only one owner, Rust does the following:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}", s1); // This will result in a compile-time error
}
```

In this case, `s1` is no longer valid after being assigned to `s2`. This is called a "move" [3](https://lucacorbucci.medium.com/understanding-ownership-in-rust-5baf5b919246). `s1`'s data is moved to `s2`, and `s1` is invalidated to prevent a double free [3](https://lucacorbucci.medium.com/understanding-ownership-in-rust-5baf5b919246).

## Clone

If you want to deeply copy the heap data of the `String`, you can use the `clone` method:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}
```

In this case, both `s1` and `s2` are valid, and each owns its own copy of the data on the heap.

## Ownership and Functions

Passing a variable to a function will move or copy, just as assignment does [4](https://www.reddit.com/r/rust/comments/yibdpi/ownership_as_explained_in_the_rust_book/).

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it’s okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

If a function takes ownership, the variable is no longer valid in the scope it was originally defined in.

## Return Values and Scope

Returning values can also transfer ownership:

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and `drop` is called. s2 was moved, so nothing
  // happens. s1 goes out of scope and `drop` is called.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("hello"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// takes_and_gives_back will take a String and return one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

This covers the fundamental concepts of ownership in Rust. Understanding these concepts is crucial for writing safe and efficient Rust code.
