---
sidebar_position: 18
---

# Rust HashMaps

A HashMap in Rust is a collection that stores key-value pairs, where each value is associated with a unique key. This data structure allows you to retrieve values by their keys rather than by indices (as with vectors) [1](https://doc.rust-lang.org/std/collections/struct.HashMap.html).

## Basic Concepts

- HashMap is defined as `HashMap<K, V>` where `K` is the key type and `V` is the value type
- Keys must implement the `Eq` and `Hash` traits
- All keys must have the same type, and all values must have the same type [2](https://doc.rust-lang.org/book/ch08-03-hash-maps.html)
- HashMaps store data on the heap

## Creating and Using HashMaps

### Creating a New HashMap

```rust
use std::collections::HashMap;

// Create an empty HashMap
let mut scores = HashMap::new();

// Insert key-value pairs
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

println!("{:?}", scores); // {"Blue": 10, "Yellow": 50}
```

You can also create a HashMap from an array of tuples:

```rust
use std::collections::HashMap;

let solar_distance = HashMap::from([
    ("Mercury", 0.4),
    ("Venus", 0.7),
    ("Earth", 1.0),
    ("Mars", 1.5),
]);
```

### Accessing Values

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// Using get() returns an Option<&V>
let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0);
println!("Blue team's score: {}", score); // Blue team's score: 10

// Checking if a key exists
if scores.contains_key("Red") {
    println!("Red team has a score");
} else {
    println!("Red team has no score yet");
}
```

### Iterating Through a HashMap

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// Iterate over key-value pairs
for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

## Updating Values in a HashMap

### Overwriting Values

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25); // Overwrites the previous value

println!("{:?}", scores); // {"Blue": 25}
```

### Inserting Only If the Key Isn't Present

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

// This will only insert if "Yellow" doesn't exist
scores.entry(String::from("Yellow")).or_insert(50);
// This won't change anything since "Blue" already exists
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores); // {"Blue": 10, "Yellow": 50}
```

### Updating Based on the Old Value

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";
let mut word_count = HashMap::new();

for word in text.split_whitespace() {
    // Get current count or insert 0, then increment
    let count = word_count.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", word_count); // {"hello": 1, "world": 2, "wonderful": 1}
```

## Ownership and HashMap

When you insert values into a HashMap:

- For types that implement the `Copy` trait (like integers), values are copied
- For owned types like `String`, the HashMap takes ownership of the values [2](https://doc.rust-lang.org/book/ch08-03-hash-maps.html)

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);

// The following would cause an error as field_name and field_value 
// have been moved into the HashMap
// println!("{}: {}", field_name, field_value);
```

## Common Use Cases with Examples

### Student Grading System

```rust
use std::collections::HashMap;

fn main() {
    // Create a HashMap for student grades
    let mut grades: HashMap<String, Vec<u32>> = HashMap::new();
    
    // Add grades for students
    add_grade(&mut grades, "Alice", 95);
    add_grade(&mut grades, "Bob", 87);
    add_grade(&mut grades, "Alice", 92);
    add_grade(&mut grades, "Charlie", 76);
    add_grade(&mut grades, "Bob", 90);
    
    // Calculate and display average grades
    for (student, scores) in &grades {
        let average: f64 = scores.iter().sum::<u32>() as f64 / scores.len() as f64;
        println!("{}'s average grade: {:.1}", student, average);
    }
}

fn add_grade(grades: &mut HashMap<String, Vec<u32>>, student: &str, grade: u32) {
    grades.entry(student.to_string())
          .or_insert(Vec::new())
          .push(grade);
}
```

### Word Frequency Counter

```rust
use std::collections::HashMap;
use std::io::{self, Read};
use std::fs::File;

fn main() -> io::Result<()> {
    // Read text from a file
    let mut file = File::open("sample.txt")?;
    let mut text = String::new();
    file.read_to_string(&mut text)?;
    
    // Count word frequencies
    let mut word_counts = HashMap::new();
    
    for word in text.split_whitespace() {
        // Clean the word (remove punctuation and convert to lowercase)
        let clean_word = word.trim_matches(|c: char| !c.is_alphanumeric())
                            .to_lowercase();
        
        if !clean_word.is_empty() {
            *word_counts.entry(clean_word).or_insert(0) += 1;
        }
    }
    
    // Find the most common words
    let mut words: Vec<(&String, &u32)> = word_counts.iter().collect();
    words.sort_by(|a, b| b.1.cmp(a.1));
    
    // Print the top 5 most common words
    println!("Most common words:");
    for (word, count) in words.iter().take(5) {
        println!("{}: {}", word, count);
    }
    
    Ok(())
}
```

### Cache Implementation

```rust
use std::collections::HashMap;
use std::time::{Duration, Instant};

struct TimeCache<K, V> {
    storage: HashMap<K, (V, Instant)>,
    ttl: Duration,
}

impl<K: std::cmp::Eq + std::hash::Hash + Clone, V: Clone> TimeCache<K, V> {
    fn new(ttl_secs: u64) -> Self {
        TimeCache {
            storage: HashMap::new(),
            ttl: Duration::from_secs(ttl_secs),
        }
    }
    
    fn insert(&mut self, key: K, value: V) {
        self.storage.insert(key, (value, Instant::now()));
    }
    
    fn get(&mut self, key: &K) -> Option<V> {
        if let Some((value, time)) = self.storage.get(key) {
            if time.elapsed() <= self.ttl {
                return Some(value.clone());
            } else {
                // Value has expired, remove it
                self.storage.remove(key);
            }
        }
        None
    }
    
    fn clean_expired(&mut self) {
        self.storage.retain(|_, (_, time)| time.elapsed() <= self.ttl);
    }
}

fn main() {
    let mut cache = TimeCache::new(5); // 5 second TTL
    
    cache.insert("key1", "value1");
    
    // This will print Some("value1") if accessed within 5 seconds
    println!("{:?}", cache.get(&"key1"));
    
    // Wait for 6 seconds and the value would be None
    // std::thread::sleep(Duration::from_secs(6));
    // println!("{:?}", cache.get(&"key1"));
}
```

HashMaps are a versatile data structure in Rust, providing efficient key-value storage with strong type safety and ownership rules. They're particularly useful when you need to associate data with identifiers or when building lookup tables and caches [6](https://www.programiz.com/rust/hashmap).