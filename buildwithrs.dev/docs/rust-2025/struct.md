---
sidebar_position: 13
---

# Rust Struct

In Rust, structs (or structures) are user-defined data types that group multiple related values into a single unit [2]. They are similar to tuples but offer more structure by allowing you to name each field [4].

## Defining Structs

You define a struct using the `struct` keyword, followed by the struct's name and a block containing the fields with their respective types [2].

```rust
struct Person {
    name: String,
    age: u8,
    height: u32,
}

```

In this example, `Person` is a struct with three fields: `name` (a `String`), `age` (an unsigned 8-bit integer), and `height` (an unsigned 32-bit integer).

## Instantiating Structs

To use a struct, you need to create an instance of it.  This is done by specifying values for each of the struct's fields [2].

```rust
let person1 = Person {
    name: String::from("John Doe"),
    age: 30,
    height: 175,
};

```

Here, `person1` is an instance of the `Person` struct, with the `name` field initialized to "John Doe", `age` to 30, and `height` to 175.  The order of initialization must match the order in which the fields are defined in the struct [1].

## Shorthand Initialization

When a variable and a field have the same name, you can use the field init shorthand [1].

```rust
struct User {
    name: String,
    admin: bool,
}

impl User {
    pub fn new(name: String) -> Self {
        Self {
            name,  // Shorthand: equivalent to name: name
            admin: false,
        }
    }
}

```

## Struct Update Syntax

You can create a new struct instance based on an existing one using struct update syntax [1].

```rust
let person2 = Person {
    name: String::from("Jane Doe"),
    ..person1 // Use the remaining fields from person1
};

```

`person2` will have the `name` field set to "Jane Doe", while the `age` and `height` fields will have the same values as `person1`.

## Accessing Fields

You can access the fields of a struct using dot notation (`.`) [2].

```rust
fn main() {
    // define a Person struct
    struct Person {
        name: String,
        age: u8,
        height: u8,
    }
    // instantiate Person struct
    let person = Person {
        name: String::from("John Doe"),
        age: 18,
        height: 178,
    };
    // access value of name field in Person struct
    println!("Person name = {}", person.name);
    // access value of age field in Person struct
    println!("Person age = {}", person.age);
    // access value of height field in Person struct
    println!("Person height = {}", person.height);
}

```

## Use Cases for Structs

1. **Grouping Related Data**: Structs are ideal for bundling related pieces of data. For example, representing a `Rectangle` with `width` and `height`, or a `Point` with `x` and `y` coordinates.
    
    ```
    struct Rectangle {
        width: u32,
        height: u32,
    }
    
    ```
    
2. **Creating Custom Data Types**: Structs allow you to define custom data types that fit your specific needs. This improves code readability and maintainability.
    
    ```rust
    struct Color {
        red: u8,
        green: u8,
        blue: u8,
    }
    
    ```
    
3. **Implementing Methods**: You can implement methods on structs to define behavior associated with the data.

    ```rust
    impl Rectangle {
        fn area(&self) -> u32 {
            self.width * self.height
        }
    }
    
    fn main() {
        let rect = Rectangle { width: 10, height: 20 };
        println!("Area: {}", rect.area()); // Output: Area: 200
    }
    
    ```

4. **Data Structures**: Structs can be used as building blocks for more complex data structures like linked lists, trees, and graphs.
5. **API Responses**: When interacting with APIs, structs can model the structure of the data being sent or received.

    ```rust
    #[derive(Debug)]
    struct ApiResponse {
        status: u16,
        body: String,
    }
    
    fn main() {
        let response = ApiResponse { status: 200, body: String::from("Success") };
        println!("{:?}", response);
    }
    
    ```

## Tuple Structs

Tuple structs are similar to regular structs but their fields don't have names [1]. They are accessed like tuples using indexing [1].

```rust
struct Color(u8, u8, u8);

fn main() {
    let red = Color(255, 0, 0);
    println!("Red: {}", red.0); // Output: Red: 255
}

```

## Unit Structs

Unit structs are field-less structs, often used as markers or to implement traits [1].

```rust
struct Marker;

impl Marker {
    fn mark(&self) {
        println!("Marked");
    }
}

fn main() {
    let m = Marker;
    m.mark(); // Output: Marked
}

```
