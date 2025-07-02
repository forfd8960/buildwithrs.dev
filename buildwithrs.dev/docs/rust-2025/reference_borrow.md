---
sidebar_position: 10
---

# Reference and Borrowing

In Rust, references and borrowing are key concepts for managing memory safely and efficiently. Instead of directly transferring ownership of data, you can create references that "borrow" access to the data [2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html)[4](https://doc.rust-lang.org/rust-by-example/scope/borrow.html)[5](https://rust-book.cs.brown.edu/ch04-02-references-and-borrowing.html). This allows multiple parts of your code to access the same data without the risk of data races or memory corruption [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html).

## References

A reference is like a pointer; it provides a way to access data stored at a certain memory address [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html). Unlike pointers, references in Rust are guaranteed to point to a valid value of a specific type for the lifetime of that reference [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html). References are created using the `&` symbol [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)[5](https://rust-book.cs.brown.edu/ch04-02-references-and-borrowing.html).

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

In this example, `&s1` creates a reference to the `String` `s1`. The function `calculate_length` takes a reference to a `String` (`&String`) as its argument. This allows the function to access the `String` without taking ownership of it [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html).

## Borrowing

Creating a reference is also known as borrowing [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)[2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html). When you borrow, you're allowed to use the data, but you don't own it [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html). When the borrowing ends (i.e., the reference goes out of scope), ownership reverts to the original owner [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html).

## Mutable References

By default, references are immutable; you can't modify the borrowed value [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)[2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html). To modify a borrowed value, you need to use a mutable reference, created with `&mut` [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)[2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html).

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

In this example, `&mut s` creates a mutable reference to `s`. The function `change` takes a mutable reference `&mut String`, allowing it to modify the original `String`.

## Borrowing Rules

Rust has strict rules to prevent data races [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)[2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html):

1.  **One or more immutable references OR one mutable reference:** You can have multiple immutable references (`&T`) to a resource, OR exactly one mutable reference (`&mut T`) [2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html).
2.  **Reference lifetime:** A borrow must not outlive the owner of the data [2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html).

### Avoiding Conflicts

It's important to avoid conflicts between mutable and immutable references. The following code will result in a compile-time error:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);

    let r3 = &mut s; // PROBLEM
    println!("{}", r3);
}
```

This code attempts to create an immutable reference (`r1` and `r2`) and then a mutable reference (`r3`) to the same `String`. Rust's borrowing rules prevent this because you can't have a mutable reference while you have any immutable references to the same data [1](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)[2](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html).

To fix this, the mutable reference must be created in a separate scope where the immutable references are no longer used:

```rust
fn main() {
    let mut s = String::from("hello");

    {
        let r1 = &s;
        let r2 = &s;
        println!("{} and {}", r1, r2);
    }

    let r3 = &mut s;
    println!("{}", r3);
}
```

In this corrected example, the scope created by `{}` ensures that the immutable references `r1` and `r2` are no longer active when the mutable reference `r3` is created. This satisfies Rust's borrowing rules, and the code compiles successfully.
