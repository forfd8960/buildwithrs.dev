---
sidebar_position: 5
---

# Rust Tuples

A tuple is a compound data type in Rust that allows you to group together a number of values with potentially different types into one compound type. Tuples have a fixed length, meaning once they're declared, they cannot grow or shrink in size.

## Tuple Basics

### Creating Tuples

Tuples are created by writing a comma-separated list of values inside parentheses:

```rust
fn main() {
    // A tuple with different types
    let person: (String, i32, f64) = (String::from("Alice"), 30, 5.8);
    
    // Type inference works too
    let coordinates = (10, 20);  // (i32, i32)
    
    // Empty tuple (unit)
    let unit = ();
}
```

### Accessing Tuple Elements

You can access tuple elements in two ways:

#### 1. Using dot notation with index:

```rust
fn main() {
    let person = ("Alice", 30, 5.8);
    
    let name = person.0;
    let age = person.1;
    let height = person.2;
    
    println!("Name: {}, Age: {}, Height: {} ft", name, age, height);
}
```

#### 2. Using destructuring (pattern matching):

```rust
fn main() {
    let person = ("Alice", 30, 5.8);
    
    // Destructuring the tuple
    let (name, age, height) = person;
    
    println!("Name: {}, Age: {}, Height: {} ft", name, age, height);
}
```

## Practical Examples of Tuples

### Example 1: Returning multiple values from a function

Tuples are perfect for returning multiple values from a function:

```rust
fn calculate_statistics(numbers: &[i32]) -> (i32, i32, i32) {
    let sum = numbers.iter().sum();
    let max = *numbers.iter().max().unwrap_or(&0);
    let min = *numbers.iter().min().unwrap_or(&0);
    
    (sum, max, min)
}

fn main() {
    let numbers = [1, 5, 10, 2, 8];
    
    // Get multiple return values using a tuple
    let (sum, max, min) = calculate_statistics(&numbers);
    
    println!("Sum: {}, Max: {}, Min: {}", sum, max, min);
}
```

### Example 2: Representing a point in 2D/3D space

Tuples are useful for representing coordinates:

```rust
fn main() {
    // 2D point
    let point_2d = (4.0, 3.0);
    
    // Calculate distance from origin
    let distance = (point_2d.0.powi(2) + point_2d.1.powi(2)).sqrt();
    println!("Distance from origin: {}", distance);
    
    // 3D point
    let point_3d = (4.0, 3.0, 7.0);
    println!("3D point: ({}, {}, {})", point_3d.0, point_3d.1, point_3d.2);
}
```

### Example 3: Key-value pairs

Simple key-value pairs can be represented with tuples:

```rust
fn main() {
    let records = [
        ("Alice", "Engineering", 3),
        ("Bob", "Marketing", 5),
        ("Charlie", "Sales", 2),
    ];
    
    for (name, department, years) in records {
        println!("{} works in {} for {} years", name, department, years);
    }
}
```

### Example 4: Using tuples with vectors

Tuples can be stored in collections like vectors:

```rust
fn main() {
    // Vector of tuples
    let mut employee_data = Vec::new();
    
    // Add some data
    employee_data.push(("Alice", 30, 75000.0));
    employee_data.push(("Bob", 25, 65000.0));
    employee_data.push(("Charlie", 35, 80000.0));
    
    // Process the data
    let mut total_salary = 0.0;
    let mut total_age = 0;
    
    for (name, age, salary) in &employee_data {
        println!("{} is {} years old and earns ${:.2}", name, age, salary);
        total_salary += salary;
        total_age += age;
    }
    
    let avg_salary = total_salary / employee_data.len() as f64;
    let avg_age = total_age as f64 / employee_data.len() as f64;
    
    println!("Average salary: ${:.2}", avg_salary);
    println!("Average age: {:.1}", avg_age);
}
```

### Example 5: Partial destructuring

You can partially destructure tuples when you're only interested in certain elements:

```rust
fn main() {
    let user = ("Alice", "alice@example.com", "password123", 30);
    
    // Destructure only the first and third elements
    // Use _ to ignore elements we don't need
    let (name, _, password, _) = user;
    
    println!("Name: {}, Password: {}", name, password);
}
```

### Example 6: Nested tuples

Tuples can be nested to create more complex structures:

```rust
fn main() {
    // Nested tuple representing a person with an address
    let person = (
        "Alice", 
        30, 
        (
            "123 Main St", 
            "Anytown", 
            "CA", 
            "12345"
        )
    );
    
    // Access elements in nested tuples
    let street = person.3.0;
    let city = person.3.1;
    let state = person.3.2;
    
    println!("{} lives in {}, {}", person.0, city, state);
    
    // Destructuring nested tuples
    let (name, age, address) = person;
    let (street_address, city_name, state_code, zip) = address;
    
    println!("{} is {} years old and lives at {}", name, age, street_address);
}
```

## Limitations of Tuples

1. **Fixed size**: Once declared, you cannot add or remove elements.
2. **Limited indexing**: You can only access elements using dot notation (.0, .1, etc.).
3. **No named fields**: Unlike structs, tuple elements don't have names (only positions).
4. **Practical limit on size**: While technically you can have many elements in a tuple, it becomes unwieldy with too many elements. The standard library only implements traits for tuples up to 12 elements.

## When to Use Tuples

Tuples are best used when:
- You need to return multiple values from a function
- You need to group a small number of related values
- The grouping is simple and doesn't warrant creating a custom struct
- The data structure is temporary or internal to a function

For more complex data structures with named fields or when you need more flexibility, Rust's structs are generally a better choice.