---
sidebar_position: 20
---

# Understanding Rust Structs

In Rust, a struct (short for structure) is a composite data type that allows you to group multiple values of different types under a single name [6](https://medium.com/@er.pwndhull07/understanding-structs-in-rust-a-complete-guide-with-examples-621bf9753b88). Structs are fundamental to organizing data in Rust programs.

## Types of Structs in Rust

Rust offers three varieties of structs [5](https://doc.rust-lang.org/std/keyword.struct.html):

1. **Named-field structs**: The most common form with named fields
2. **Tuple structs**: Structs that are similar to tuples
3. **Unit structs**: Field-less structs used primarily with generics

Let's explore each type with examples.

## 1. Named-field Structs

This is the most common form where each field has a name and type.

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    // Create an instance
    let user1 = User {
        username: String::from("rustacean"),
        email: String::from("rust@example.com"),
        sign_in_count: 1,
        active: true,
    };
    
    // Access fields using dot notation
    println!("Username: {}", user1.username);
    
    // Create a mutable instance
    let mut user2 = User {
        username: String::from("coder"),
        email: String::from("coder@example.com"),
        sign_in_count: 3,
        active: true,
    };
    
    // Modify a field
    user2.email = String::from("new_email@example.com");
}
```

## 2. Tuple Structs

Tuple structs are named tuples, useful when you want to give a tuple a name but don't need named fields.

```rust
struct Color(i32, i32, i32); // RGB
struct Point(f64, f64, f64); // 3D coordinate

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0.0, 0.0, 0.0);
    
    // Access elements using index notation
    println!("Red component: {}", black.0);
    
    // Destructure a tuple struct
    let Point(x, y, z) = origin;
    println!("Coordinates: {}, {}, {}", x, y, z);
}
```

## 3. Unit Structs

Unit structs have no fields. They're useful for implementing traits when you don't need to store any data.

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
    // Useful when implementing traits but don't need any data storage
}
```

## Key Features and Patterns

### 1. Field Init Shorthand

When variable names match struct field names, you can use the shorthand syntax:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,      // instead of email: email,
        username,   // instead of username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

### 2. Struct Update Syntax

Rust provides a convenient way to create a new instance from an existing one:

```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1  // copy remaining fields from user1
};
```

This creates a new struct with the same values as `user1` except for the email field [1](https://doc.rust-lang.org/book/ch05-01-defining-structs.html).

### 3. Destructuring

You can destructure structs in pattern matching contexts:

```rust
struct Point {
    x: f32,
    y: f32,
}

fn main() {
    let point = Point { x: 0.0, y: 0.0 };
    let Point { x: a, y: b } = point;
    
    // Or more concisely:
    let Point { x, y } = point;
    
    println!("x: {}, y: {}", x, y);
}
```

### 4. Derived Traits

Rust allows you to automatically implement common traits for your structs:

```rust
#[derive(Debug, Clone, PartialEq)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };
    
    // Debug trait allows printing with {:?}
    println!("Rectangle: {:?}", rect);
    
    // Clone trait allows cloning
    let rect2 = rect.clone();
    
    // PartialEq allows equality comparison
    if rect == rect2 {
        println!("Rectangles are equal!");
    }
}
```

### 5. Associated Functions and Methods

You can define associated functions and methods for structs:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Associated function (doesn't take self)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
    
    // Method (takes self)
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    // Using the associated function
    let rect1 = Rectangle::new(30, 50);
    
    // Using methods
    println!("Area: {}", rect1.area());
    
    let rect2 = Rectangle::new(10, 20);
    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
}
```

## Idiomatic Rust Struct Patterns

### 1. Builder Pattern

For structs with many optional fields:

```rust
#[derive(Default)]
struct ServerConfig {
    hostname: String,
    port: u16,
    max_connections: u32,
    timeout: u64,
}

impl ServerConfig {
    fn new() -> Self {
        Default::default() // Creates with default values
    }
    
    fn hostname(mut self, hostname: String) -> Self {
        self.hostname = hostname;
        self
    }
    
    fn port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }
    
    fn max_connections(mut self, max_connections: u32) -> Self {
        self.max_connections = max_connections;
        self
    }
    
    fn timeout(mut self, timeout: u64) -> Self {
        self.timeout = timeout;
        self
    }
    
    fn build(self) -> Server {
        Server::new(self)
    }
}

struct Server {
    config: ServerConfig,
}

impl Server {
    fn new(config: ServerConfig) -> Self {
        Server { config }
    }
}

fn main() {
    let server = ServerConfig::new()
        .hostname(String::from("localhost"))
        .port(8080)
        .max_connections(1000)
        .build();
}
```

### 2. Newtype Pattern

For type safety and abstraction:

```rust
// Newtype pattern - wrapping a type for added type safety
struct Meters(f64);
struct Kilometers(f64);

impl Meters {
    fn to_kilometers(&self) -> Kilometers {
        Kilometers(self.0 / 1000.0)
    }
}

impl Kilometers {
    fn to_meters(&self) -> Meters {
        Meters(self.0 * 1000.0)
    }
}

fn main() {
    let distance = Meters(1500.0);
    let km_distance = distance.to_kilometers();
    println!("Distance in kilometers: {}", km_distance.0);
}
```

### 3. Type-State Pattern

Using the type system to enforce state transitions:

```rust
struct Uninitialized;
struct Initialized;
struct Running;

struct Service<State> {
    state: std::marker::PhantomData<State>,
    // Common fields
    name: String,
}

impl Service<Uninitialized> {
    fn new(name: String) -> Self {
        Service {
            state: std::marker::PhantomData,
            name,
        }
    }
    
    fn initialize(self) -> Service<Initialized> {
        println!("Initializing service: {}", self.name);
        Service {
            state: std::marker::PhantomData,
            name: self.name,
        }
    }
}

impl Service<Initialized> {
    fn start(self) -> Service<Running> {
        println!("Starting service: {}", self.name);
        Service {
            state: std::marker::PhantomData,
            name: self.name,
        }
    }
}

impl Service<Running> {
    fn stop(self) -> Service<Initialized> {
        println!("Stopping service: {}", self.name);
        Service {
            state: std::marker::PhantomData,
            name: self.name,
        }
    }
}

fn main() {
    let service = Service::new(String::from("database"))
        .initialize()
        .start();
    
    // The following wouldn't compile because service is now in Running state
    // service.initialize();
}
```

### 4. Private Fields with Public Accessors

Encapsulation pattern:

```rust
struct Person {
    name: String,
    age: u8,
    // Private field
    private_id: String,
}

impl Person {
    fn new(name: String, age: u8) -> Self {
        let private_id = format!("{}:{}", name, age); // Generate an internal ID
        Person { name, age, private_id }
    }
    
    // Public accessor for private field
    fn id(&self) -> &str {
        &self.private_id
    }
}

fn main() {
    let person = Person::new(String::from("Alice"), 30);
    println!("Person: {} ({}), ID: {}", person.name, person.age, person.id());
    
    // This would fail to compile as private_id is private
    // println!("{}", person.private_id);
}
```

These patterns showcase Rust's powerful type system and how structs can be used to create safe, expressive, and maintainable code. By leveraging these patterns and features, you can write idiomatic Rust code that takes advantage of the language's safety guarantees.