---
sidebar_position: 16
---

# Rust Enum


**Rust Enum** (short for enumeration) is a powerful feature in the Rust programming language that allows you to define a type by enumerating its possible variants. Enums are used to represent data that can be one of a fixed set of possibilities, each of which may carry different types or amounts of associated data. This makes them ideal for modeling state machines, message types, error handling, and other scenarios where data can take multiple forms.

### Key Characteristics of Rust Enums
1. **Variants**: Each enum defines a set of possible variants, which can be simple (like a constant) or complex (holding associated data).
2. **Type Safety**: Rust ensures that you handle all possible variants of an enum explicitly, often using pattern matching (`match` or `if let`), preventing runtime errors.
3. **Associated Data**: Variants can hold data of different types, such as integers, strings, or even other structs/enums.
4. **Memory Efficiency**: Enums use a tag to indicate the active variant, with the memory layout optimized for the largest variant.

### Syntax of an Enum
```rust
enum EnumName {
    Variant1,              // Simple variant
    Variant2(Type),       // Variant with associated data
    Variant3 { field: Type }, // Variant with named fields
}
```

### Common Use Cases for Rust Enums
1. **Modeling State Machines**: Enums are perfect for representing states in a system, such as a traffic light or a game state.
2. **Error Handling**: Rust’s `Option` and `Result` enums are widely used for handling optional values and errors.
3. **Message Passing**: Enums can represent different types of messages in concurrent or event-driven systems.
4. **Heterogeneous Data**: Enums allow you to store different types of data under a single type, with compile-time guarantees.
5. **Domain Modeling**: Enums can represent domain-specific categories, like shapes or commands.

### Code Examples for Different Use Cases

#### 1. **Modeling a State Machine (Traffic Light)**
Enums can represent states in a state machine, with each variant corresponding to a specific state.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

fn action_for_light(light: TrafficLight) {
    match light {
        TrafficLight::Red => println!("Stop!"),
        TrafficLight::Yellow => println!("Slow down!"),
        TrafficLight::Green => println!("Go!"),
    }
}

fn main() {
    let light = TrafficLight::Yellow;
    action_for_light(light); // Output: Slow down!
}
```

Here, the `TrafficLight` enum has three variants, and the `match` expression ensures all cases are handled.

#### 2. **Error Handling with `Result` and Custom Enums**
Rust’s standard library uses `Result` for error handling, but you can create custom enums for specific error types.

```rust
enum MathError {
    DivisionByZero,
    NegativeLogarithm,
}

fn divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

fn main() {
    match divide(10.0, 0.0) {
        Ok(result) => println!("Result: {}", result),
        Err(MathError::DivisionByZero) => println!("Error: Cannot divide by zero"),
        Err(MathError::NegativeLogarithm) => println!("Error: Negative logarithm"),
    }
}
```

This shows how a custom `MathError` enum can represent different error conditions, used with `Result`.

#### 3. **Message Passing in a System**
Enums are great for defining messages in a system, such as a command-line interface or a server.

```rust
enum Command {
    Move { x: i32, y: i32 }, // Variant with named fields
    Say(String),             // Variant with a single value
    Quit,                   // Simple variant
}

fn process_command(cmd: Command) {
    match cmd {
        Command::Move { x, y } => println!("Moving to ({}, {})", x, y),
        Command::Say(message) => println!("Saying: {}", message),
        Command::Quit => println!("Quitting..."),
    }
}

fn main() {
    let cmd1 = Command::Move { x: 10, y: 20 };
    let cmd2 = Command::Say(String::from("Hello, Rust!"));
    process_command(cmd1); // Output: Moving to (10, 20)
    process_command(cmd2); // Output: Saying: Hello, Rust!
}
```

Here, the `Command` enum represents different actions with associated data, and `match` processes them.

#### 4. **Heterogeneous Data (Shapes Example)**
Enums can store different types of data, useful for modeling complex systems like geometric shapes.

```rust
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
}

fn area(shape: Shape) -> f64 {
    match shape {
        Shape::Circle { radius } => 3.14159 * radius * radius,
        Shape::Rectangle { width, height } => width * height,
    }
}

fn main() {
    let circle = Shape::Circle { radius: 5.0 };
    let rectangle = Shape::Rectangle { width: 4.0, height: 6.0 };
    println!("Circle area: {}", area(circle)); // Output: Circle area: 78.53975
    println!("Rectangle area: {}", area(rectangle)); // Output: Rectangle area: 24
}
```

This demonstrates how enums can hold different data structures and compute based on the variant.

#### 5. **Using `Option` Enum for Optional Values**
Rust’s `Option` enum is used to handle cases where a value may or may not be present.

```rust
fn find_first_even(numbers: &[i32]) -> Option<i32> {
    for &num in numbers {
        if num % 2 == 0 {
            return Some(num);
        }
    }
    None
}

fn main() {
    let numbers = vec![1, 3, 4, 5, 6];
    match find_first_even(&numbers) {
        Some(num) => println!("First even number: {}", num), // Output: First even number: 4
        None => println!("No even number found"),
    }
}
```

`Option` is a built-in enum with variants `Some(T)` and `None`, used here to return the first even number or indicate none was found.

### Advanced Features
- **Pattern Matching with Guards**: You can add conditions to `match` arms.
  ```rust
  enum Temperature {
      Celsius(i32),
      Fahrenheit(i32),
  }

  fn describe_temp(temp: Temperature) {
      match temp {
          Temperature::Celsius(t) if t > 30 => println!("Hot!"),
          Temperature::Celsius(t) => println!("Celsius: {}°C", t),
          Temperature::Fahrenheit(t) => println!("Fahrenheit: {}°F", t),
      }
  }
  ```
- **Enums with Methods**: Enums can have `impl` blocks to define methods.
  ```rust
  enum Color {
      Red,
      Blue,
  }

  impl Color {
      fn describe(&self) -> &'static str {
          match self {
              Color::Red => "Red is vibrant",
              Color::Blue => "Blue is calm",
          }
      }
  }

  fn main() {
      let color = Color::Red;
      println!("{}", color.describe()); // Output: Red is vibrant
  }
  ```

### Why Use Enums?
- **Safety**: The `match` expression ensures all variants are handled, preventing bugs.
- **Expressiveness**: Enums allow clear modeling of domain logic.
- **Flexibility**: Variants can carry different data, making enums versatile.

### Real-World Usage
- **Rust Standard Library**: `Option` and `Result` are used extensively for null safety and error handling.
- **Web Frameworks**: Libraries like `actix-web` use enums to represent HTTP methods or response types.
- **Game Development**: Enums model game states, events, or entity types in engines like Bevy.
- **Compilers**: Rust’s own compiler uses enums to represent AST nodes or error types.

For more details, check the Rust Book: [Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html).

If you have a specific use case or want a deeper dive into any aspect, let me know!