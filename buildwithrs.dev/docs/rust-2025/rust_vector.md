---
sidebar_position: 19
---

# Rust Vectors: Dynamic Arrays

A Vector in Rust (`Vec<T>`) is a resizable, heap-allocated data structure that allows you to store multiple values of the same type in a single collection. Unlike arrays, vectors can grow or shrink at runtime, making them ideal for situations where the size of your collection is unknown in advance [1](https://doc.rust-lang.org/rust-by-example/std/vec.html).

## Creating Vectors

There are several ways to create vectors in Rust:

```rust
fn main() {
    // Empty vector with type annotation
    let mut v1: Vec<i32> = Vec::new();
    
    // Using vec! macro
    let v2 = vec![1, 2, 3, 4, 5];
    
    // With capacity (pre-allocates memory)
    let mut v3 = Vec::with_capacity(10);
    
    // Creating from iterators
    let v4: Vec<i32> = (0..10).collect();
    
    println!("v2 = {:?}", v2);
    println!("v4 = {:?}", v4);
}
```

## Adding Elements

```rust
fn main() {
    // Creating a mutable vector
    let mut numbers = Vec::new();
    
    // Adding elements with push
    numbers.push(1);
    numbers.push(2);
    numbers.push(3);
    
    println!("Vector: {:?}", numbers);
    
    // Extend with multiple values
    numbers.extend(vec![4, 5, 6]);
    println!("Extended vector: {:?}", numbers);
    
    // Insert at specific position
    numbers.insert(1, 10); // Insert 10 at index 1
    println!("After insertion: {:?}", numbers);
}
```

## Accessing Elements

```rust
fn main() {
    let v = vec![10, 20, 30, 40, 50];
    
    // Method 1: Using indexing (panics if out of bounds)
    println!("First element: {}", v[0]);
    println!("Second element: {}", v[1]);
    
    // Method 2: Using get() (returns Option<&T>)
    match v.get(2) {
        Some(value) => println!("Third element: {}", value),
        None => println!("No element at index 2"),
    }
    
    // Safe access with if let
    if let Some(value) = v.get(100) {
        println!("Element at index 100: {}", value);
    } else {
        println!("No element at index 100");
    }
}
```

## Removing Elements

```rust
fn main() {
    let mut v = vec![10, 20, 30, 40, 50];
    println!("Original vector: {:?}", v);
    
    // Remove and return last element
    let last = v.pop();
    println!("Popped value: {:?}", last);
    println!("Vector after pop: {:?}", v);
    
    // Remove element at specific index
    let removed = v.remove(1); // Removes element at index 1
    println!("Removed element at index 1: {}", removed);
    println!("Vector after remove: {:?}", v);
    
    // Remove and swap with last element (faster than remove)
    let swap_removed = v.swap_remove(0);
    println!("Swap removed from index 0: {}", swap_removed);
    println!("Vector after swap_remove: {:?}", v);
    
    // Clear all elements
    v.clear();
    println!("Vector after clear: {:?}", v);
}
```

## Iterating Over Vectors

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Basic iteration
    println!("Basic iteration:");
    for num in &numbers {
        println!("> {}", num);
    }
    
    // Iteration with indices
    println!("\nIteration with indices:");
    for (i, num) in numbers.iter().enumerate() {
        println!("numbers[{}] = {}", i, num);
    }
    
    // Mutable iteration
    let mut doubled = vec![1, 2, 3, 4, 5];
    println!("\nBefore doubling: {:?}", doubled);
    
    for num in &mut doubled {
        *num *= 2; // Double each value
    }
    
    println!("After doubling: {:?}", doubled);
    
    // Using iterators and functional approaches
    let sum: i32 = numbers.iter().sum();
    println!("\nSum of all elements: {}", sum);
    
    let even_numbers: Vec<&i32> = numbers.iter().filter(|&&n| n % 2 == 0).collect();
    println!("Even numbers: {:?}", even_numbers);
}
```

## Capacity Management

```rust
fn main() {
    // Create with initial capacity
    let mut v = Vec::with_capacity(10);
    println!("Length: {}, Capacity: {}", v.len(), v.capacity());
    
    // Add elements
    for i in 0..5 {
        v.push(i);
    }
    println!("After adding 5 elements - Length: {}, Capacity: {}", v.len(), v.capacity());
    
    // Reserve more space
    v.reserve(10);
    println!("After reserve(10) - Length: {}, Capacity: {}", v.len(), v.capacity());
    
    // Shrink to fit
    v.shrink_to_fit();
    println!("After shrink_to_fit - Length: {}, Capacity: {}", v.len(), v.capacity());
}
```

## Advanced Operations

```rust
fn main() {
    // Sorting
    let mut numbers = vec![5, 2, 8, 1, 9, 3];
    numbers.sort();
    println!("Sorted vector: {:?}", numbers);
    
    // Reverse sorting
    numbers.sort_by(|a, b| b.cmp(a));
    println!("Reverse sorted: {:?}", numbers);
    
    // Deduplication (removes consecutive duplicates)
    let mut with_duplicates = vec![1, 2, 2, 3, 3, 3, 4, 4, 1];
    with_duplicates.dedup();
    println!("After dedup: {:?}", with_duplicates);
    
    // Slicing
    let slice = &numbers[1..4];
    println!("Slice: {:?}", slice);
    
    // Split at index
    let (left, right) = numbers.split_at(3);
    println!("Left part: {:?}", left);
    println!("Right part: {:?}", right);
}
```

## Practical Use Cases

### Example 1: Task Manager

```rust
#[derive(Debug)]
struct Task {
    id: u32,
    description: String,
    completed: bool,
}

fn main() {
    // Create a vector to store tasks
    let mut tasks: Vec<Task> = Vec::new();
    
    // Add tasks
    tasks.push(Task {
        id: 1,
        description: String::from("Learn Rust"),
        completed: false,
    });
    
    tasks.push(Task {
        id: 2,
        description: String::from("Create a project"),
        completed: false,
    });
    
    tasks.push(Task {
        id: 3,
        description: String::from("Take a break"),
        completed: true,
    });
    
    // Display all tasks
    println!("All tasks:");
    for task in &tasks {
        println!("{:?}", task);
    }
    
    // Find and update a task
    if let Some(task) = tasks.iter_mut().find(|t| t.id == 1) {
        task.completed = true;
        println!("\nUpdated task 1: {:?}", task);
    }
    
    // Filter for incomplete tasks
    let pending_tasks: Vec<&Task> = tasks.iter()
        .filter(|t| !t.completed)
        .collect();
        
    println!("\nPending tasks: {:?}", pending_tasks);
    
    // Count completed tasks
    let completed_count = tasks.iter()
        .filter(|t| t.completed)
        .count();
        
    println!("\nCompleted tasks: {}", completed_count);
}
```

### Example 2: Working with Data

```rust
fn main() {
    // Sample data
    let data = vec![5, 2, 9, 1, 5, 6, 3, 8, 4];
    
    // Calculate statistics
    let sum: i32 = data.iter().sum();
    let count = data.len();
    let average = sum as f64 / count as f64;
    
    let min = data.iter().min().unwrap_or(&0);
    let max = data.iter().max().unwrap_or(&0);
    
    println!("Data: {:?}", data);
    println!("Count: {}", count);
    println!("Sum: {}", sum);
    println!("Average: {:.2}", average);
    println!("Min: {}", min);
    println!("Max: {}", max);
    
    // Find values above average
    let above_average: Vec<&i32> = data.iter()
        .filter(|&&x| x as f64 > average)
        .collect();
    println!("Values above average: {:?}", above_average);
    
    // Frequency analysis
    let mut frequency = std::collections::HashMap::new();
    for &value in &data {
        *frequency.entry(value).or_insert(0) += 1;
    }
    println!("Frequency distribution: {:?}", frequency);
}
```

### Example 3: Dynamic 2D Grid

```rust
fn main() {
    // Create a 2D grid as a vector of vectors
    let mut grid: Vec<Vec<i32>> = Vec::new();
    
    // Initialize a 3x3 grid with zeros
    for _ in 0..3 {
        grid.push(vec![0; 3]);
    }
    
    // Set some values
    grid[0][0] = 1;
    grid[1][1] = 5;
    grid[2][2] = 9;
    
    // Print the grid
    println!("Grid:");
    for row in &grid {
        for &cell in row {
            print!("{} ", cell);
        }
        println!();
    }
    
    // Add a new row
    grid.push(vec![7, 8, 9]);
    
    // Add a new column to each row
    for row in &mut grid {
        row.push(0);
    }
    
    println!("\nUpdated grid:");
    for row in &grid {
        for &cell in row {
            print!("{} ", cell);
        }
        println!();
    }
    
    // Calculate sum of each row
    println!("\nRow sums:");
    for (i, row) in grid.iter().enumerate() {
        let sum: i32 = row.iter().sum();
        println!("Row {}: {}", i, sum);
    }
}
```

Vectors are one of Rust's most important data structures, offering a perfect balance between flexibility and performance. They're implemented in a way that provides memory safety guarantees while still being efficient [2](https://doc.rust-lang.org/std/vec/struct.Vec.html). Understanding how to work with vectors is essential for effective Rust programming.
