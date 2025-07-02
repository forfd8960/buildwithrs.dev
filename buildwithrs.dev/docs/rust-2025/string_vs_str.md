---
sidebar_position: 11
---

# `String` VS `&str`

**String vs. &str in Rust**

In Rust, both `String` and `&str` are used to represent strings, but they have different characteristics and use cases.

*   **`String`**:
    *   A growable, mutable, owned string type.
    *   Allocated on the heap.
    *   You own the `String` data. When the `String` goes out of scope, the memory is automatically freed.
    *   Use when you need to modify or own the string data [4].
*   **`&str`**:
    *   A "string slice", which is an immutable view into a string.
    *   Can be a view into a `String` or a string literal stored in the program's binary.
    *   Does not own the string data; it borrows it.
    *   Use when you need a temporary, read-only view into string data [4].

**Key Differences**

| Feature         | `String`                                  | `&str`                                      |
|-----------------|-------------------------------------------|---------------------------------------------|
| Ownership       | Owns the string data                      | Borrows the string data                     |
| Mutability      | Mutable                                   | Immutable                                     |
| Allocation      | Heap-allocated                            | Can be stack-allocated (string literals) or a borrow from the heap |
| Use Cases       | Modifying, owning string data             | Read-only access, function arguments        |

**When to Use Which**

*   Use `String` when:
    *   You need to modify the string [4].
    *   You need to own the string data.
    *   You need to create a string at runtime.
*   Use `&str` when:
    *   You only need to read the string data [4].
    *   You don't need to own the string data.
    *   You are passing a string as an argument to a function [5].
    *   You are working with string literals.

**Code Examples**

1.  **Creating a `String`**:

```rust
fn main() {
    let mut my_string = String::from("Hello"); // Create a String from a string literal
    my_string.push_str(", world!"); // Modify the String
    println!("{}", my_string);
}
```

2.  **Using `&str` as a function argument**:

```rust
fn print_string_slice(s: &str) {
    println!("{}", s);
}

fn main() {
    let my_string = String::from("Hello, String!");
    print_string_slice(&my_string); // Pass a &str slice of the String

    let my_str = "Hello, str!"; // String literal, which is a &str
    print_string_slice(my_str); // Pass the &str literal
}
```

3.  **Returning a `String` from a function**:

```rust
fn create_string() -> String {
    let s = String::from("A new string");
    s
}

fn main() {
    let my_string = create_string();
    println!("{}", my_string);
}
```

4.  **String vs &str in Structs**
    As a general rule, use `String` in structs and use `&str` for parameters in functions [5].

```rust
struct MyStruct {
    name: String,
}

impl MyStruct {
    fn new(name: String) -> MyStruct {
        MyStruct { name }
    }

    fn print_name(&self, prefix: &str) {
        println!("{}: {}", prefix, self.name);
    }
}

fn main() {
    let my_struct = MyStruct::new(String::from("Devv"));
    my_struct.print_name("Name");
}
```

In summary, use `String` when you need to own and modify the string data, and use `&str` when you need a read-only view of the string data or when passing string literals [4]. Using `&str` for function arguments is flexible because it can accept both `String` (by borrowing) and string literals.
