---
slug: rust-collections
title: Rust Collections
authors: forfd8960
tags: [rust, collections]
---

Rust, known for its focus on memory safety, performance, and concurrency, provides a rich set of data structures in its standard library, collectively known as "collections." These collections are essential for organizing and manipulating data efficiently in your applications. While `Vec` and `HashMap` are often the go-to choices for most general-purpose data storage, Rust offers a variety of other specialized collections, each with its own strengths and optimal use cases [2](https://doc.rust-lang.org/std/collections/index.html).

This blog post will guide you through Rust's most common collections, explaining their purpose, typical use cases, and providing practical code examples. We'll also touch upon how to use custom hashing algorithms for `HashMap` and `HashSet`.

<!-- truncate -->

## Rust's Core Collections: An Overview

Rust's collections can be broadly categorized into sequences, maps, sets, and miscellaneous structures [2](https://doc.rust-lang.org/std/collections/index.html). Understanding their underlying implementations and performance characteristics is key to choosing the right tool for the job.

### 1. `Vec<T>` (Vector)

A `Vec<T>` is a dynamic, resizable array that can grow or shrink in size as needed [1](https://doc.rust-lang.org/book/ch08-00-common-collections.html)[3](https://dev.to/alexmercedcoder/working-with-collections-in-rust-a-comprehensive-guide-3c9f). It stores elements contiguously in memory, offering excellent cache performance and O(1) access to elements by index [2](https://doc.rust-lang.org/std/collections/index.html).

*   **Common Use Cases:**
    *   General-purpose lists where the number of elements changes at runtime.
    *   Implementing a stack (using `push` and `pop`).
    *   Collecting items to be processed later [2](https://doc.rust-lang.org/std/collections/index.html).
    *   Heap-allocated arrays [2](https://doc.rust-lang.org/std/collections/index.html).

*   **Code Example:**

    ```rust
    fn main() {
        // Create a new, empty vector
        let mut numbers: Vec<i32> = Vec::new();

        // Add elements
        numbers.push(1);
        numbers.push(2);
        numbers.push(3);
        println!("Vector after pushes: {:?}", numbers); // Output: [1, 2, 3]

        // Use the vec! macro for initial values
        let mut more_numbers = vec![4, 5, 6];
        println!("Vector created with macro: {:?}", more_numbers); // Output: [4, 5, 6]

        // Access elements by index
        println!("First element: {}", numbers[0]); // Output: 1

        // Remove the last element (pop)
        let last_num = numbers.pop();
        println!("Popped element: {:?}", last_num); // Output: Some(3)
        println!("Vector after pop: {:?}", numbers); // Output: [1, 2]

        // Iterate over elements
        for num in &more_numbers {
            print!("{} ", num); // Output: 4 5 6
        }
        println!();
    }
    ```

### 2. `VecDeque<T>` (Vector Deque)

A `VecDeque<T>` is a double-ended queue implemented as a ring buffer. It provides efficient insertion and removal at both the front and back of the sequence, unlike `Vec` which is efficient only at the back [2](https://doc.rust-lang.org/std/collections/index.html).

*   **Common Use Cases:**
    *   Implementing a queue (FIFO - First-In, First-Out).
    *   Implementing a double-ended queue (deque).
    *   When you need `Vec`-like behavior but also require efficient insertions/deletions at the front [2](https://doc.rust-lang.org/std/collections/index.html).

*   **Code Example:**

    ```rust
    use std::collections::VecDeque;

    fn main() {
        let mut deque: VecDeque<char> = VecDeque::new();

        // Add elements to the back
        deque.push_back('a');
        deque.push_back('b');
        println!("Deque after push_back: {:?}", deque); // Output: ['a', 'b']

        // Add elements to the front
        deque.push_front('x');
        deque.push_front('y');
        println!("Deque after push_front: {:?}", deque); // Output: ['y', 'x', 'a', 'b']

        // Remove elements from the front
        let front_char = deque.pop_front();
        println!("Popped from front: {:?}", front_char); // Output: Some('y')
        println!("Deque after pop_front: {:?}", deque); // Output: ['x', 'a', 'b']

        // Remove elements from the back
        let back_char = deque.pop_back();
        println!("Popped from back: {:?}", back_char); // Output: Some('b')
        println!("Deque after pop_back: {:?}", deque); // Output: ['x', 'a']
    }
    ```

### 3. `LinkedList<T>`

A `LinkedList<T>` is a doubly linked list. While it offers O(1) time complexity for insertions and deletions at either end, and efficient splitting/appending of lists, it's generally less common in Rust than `Vec` or `VecDeque` due to higher memory overhead and poorer cache performance for sequential access [2](https://doc.rust-lang.org/std/collections/index.html)[3](https://dev.to/alexmercedcoder/working-with-collections-in-rust-a-comprehensive-guide-3c9f).

*   **Common Use Cases:**
    *   When you need to frequently split and append lists efficiently [2](https://doc.rust-lang.org/std/collections/index.html).
    *   When you require efficient insertions or deletions in the middle of a list, and you have an iterator pointing to that position.
    *   When `Vec` or `VecDeque`'s amortized reallocations are unacceptable, and you need consistent O(1) insertion/deletion at ends [2](https://doc.rust-lang.org/std/collections/index.html).

*   **Code Example:**

    ```rust
    use std::collections::LinkedList;

    fn main() {
        let mut list: LinkedList<i32> = LinkedList::new();

        // Add elements to the back
        list.push_back(10);
        list.push_back(20);
        println!("List after push_back: {:?}", list); // Output: [10, 20]

        // Add elements to the front
        list.push_front(5);
        println!("List after push_front: {:?}", list); // Output: [5, 10, 20]

        // Remove elements from the front
        let front_val = list.pop_front();
        println!("Popped from front: {:?}", front_val); // Output: Some(5)
        println!("List after pop_front: {:?}", list); // Output: [10, 20]

        // Remove elements from the back
        let back_val = list.pop_back();
        println!("Popped from back: {:?}", back_val); // Output: Some(20)
        println!("List after pop_back: {:?}", list); // Output: [10]

        // Iterate over elements
        for item in &list {
            print!("{} ", item); // Output: 10
        }
        println!();
    }
    ```

### 4. `HashMap<K, V>`

A `HashMap<K, V>` is a key-value store that uses a hash table for efficient storage and retrieval. It offers average O(1) time complexity for insertions, deletions, and lookups, making it incredibly fast for many operations [2](https://doc.rust-lang.org/std/collections/index.html)[3](https://dev.to/alexmercedcoder/working-with-collections-in-rust-a-comprehensive-guide-3c9f). The order of elements is not guaranteed.

*   **Common Use Cases:**
    *   Associating arbitrary keys with arbitrary values [2](https://doc.rust-lang.org/std/collections/index.html).
    *   Implementing caches.
    *   Counting occurrences of items.
    *   Storing configuration settings.

*   **Code Example:**

    ```rust
    use std::collections::HashMap;

    fn main() {
        let mut scores: HashMap<String, i32> = HashMap::new();

        // Insert key-value pairs
        scores.insert(String::from("Alice"), 10);
        scores.insert(String::from("Bob"), 20);
        scores.insert(String::from("Charlie"), 15);
        println!("Scores map: {:?}", scores); // Order may vary

        // Get a value by key
        let alice_score = scores.get("Alice");
        println!("Alice's score: {:?}", alice_score); // Output: Some(10)

        // Update a value
        scores.insert(String::from("Alice"), 12);
        println!("Alice's new score: {:?}", scores.get("Alice")); // Output: Some(12)

        // Check if a key exists
        if scores.contains_key("Bob") {
            println!("Bob is in the map.");
        }

        // Iterate over key-value pairs
        for (name, score) in &scores {
            println!("{}: {}", name, score);
        }
    }
    ```

### 5. `HashSet<T>`

A `HashSet<T>` is a collection of unique elements, also implemented using a hash table. It's ideal for scenarios where you need to store distinct items and quickly check for their presence, without caring about the order of elements [2](https://doc.rust-lang.org/std/collections/index.html)[3](https://dev.to/alexmercedcoder/working-with-collections-in-rust-a-comprehensive-guide-3c9f). Like `HashMap`, it offers average O(1) performance for insertions, deletions, and membership tests.

*   **Common Use Cases:**
    *   Storing distinct items (e.g., unique words in a document).
    *   Efficiently checking if an item has been seen before.
    *   Performing set operations like union, intersection, and difference.

*   **Code Example:**

    ```rust
    use std::collections::HashSet;

    fn main() {
        let mut unique_numbers: HashSet<i32> = HashSet::new();

        // Insert elements
        unique_numbers.insert(1);
        unique_numbers.insert(2);
        unique_numbers.insert(3);
        unique_numbers.insert(2); // Duplicate, will be ignored
        println!("Unique numbers set: {:?}", unique_numbers); // Order may vary, contains {1, 2, 3}

        // Check for presence
        if unique_numbers.contains(&1) {
            println!("1 is in the set.");
        }
        if !unique_numbers.contains(&4) {
            println!("4 is not in the set.");
        }

        // Remove an element
        unique_numbers.remove(&3);
        println!("Set after removing 3: {:?}", unique_numbers); // Contains {1, 2}

        // Iterate over elements
        for num in &unique_numbers {
            print!("{} ", num);
        }
        println!();
    }
    ```

### 6. `BTreeMap<K, V>`

A `BTreeMap<K, V>` is a key-value store implemented as a B-tree. Unlike `HashMap`, `BTreeMap` keeps its elements sorted by key. This provides O(log N) time complexity for insertions, deletions, and lookups, which is slower than `HashMap` for individual operations but allows for ordered iteration and efficient range queries [2](https://doc.rust-lang.org/std/collections/index.html).

*   **Common Use Cases:**
    *   When you need a map where keys are always sorted.
    *   Performing range queries (e.g., finding all entries between two keys) [2](https://doc.rust-lang.org/std/collections/index.html).
    *   Finding the smallest or largest key-value pair [2](https://doc.rust-lang.org/std/collections/index.html).
    *   When cryptographic security against hash collision attacks is a concern (as `HashMap` can be vulnerable).

*   **Code Example:**

    ```rust
    use std::collections::BTreeMap;

    fn main() {
        let mut student_grades: BTreeMap<String, char> = BTreeMap::new();

        // Insert key-value pairs
        student_grades.insert(String::from("Alice"), 'A');
        student_grades.insert(String::from("Charlie"), 'C');
        student_grades.insert(String::from("Bob"), 'B');
        println!("Student grades map: {:?}", student_grades); // Output: {"Alice": 'A', "Bob": 'B', "Charlie": 'C'} (sorted by key)

        // Get a value by key
        let bob_grade = student_grades.get("Bob");
        println!("Bob's grade: {:?}", bob_grade); // Output: Some('B')

        // Iterate over key-value pairs (always sorted by key)
        for (name, grade) in &student_grades {
            println!("{}: {}", name, grade);
        }

        // Get a range of entries
        println!("Grades from B to C:");
        for (name, grade) in student_grades.range("Bob".to_string()..="Charlie".to_string()) {
            println!("{}: {}", name, grade);
        }
    }
    ```

### 7. `BTreeSet<T>`

A `BTreeSet<T>` is a collection of unique elements, implemented as a B-tree. Similar to `BTreeMap`, it keeps its elements sorted, providing O(log N) performance for insertions, deletions, and membership tests. It's the set equivalent of `BTreeMap` [2](https://doc.rust-lang.org/std/collections/index.html).

*   **Common Use Cases:**
    *   Storing distinct items that need to be kept in sorted order.
    *   Performing set operations on sorted data.
    *   Efficient range queries on sets.

*   **Code Example:**

    ```rust
    use std::collections::BTreeSet;

    fn main() {
        let mut sorted_words: BTreeSet<String> = BTreeSet::new();

        // Insert elements
        sorted_words.insert(String::from("apple"));
        sorted_words.insert(String::from("zebra"));
        sorted_words.insert(String::from("banana"));
        sorted_words.insert(String::from("apple")); // Duplicate, ignored
        println!("Sorted words set: {:?}", sorted_words); // Output: {"apple", "banana", "zebra"} (sorted)

        // Check for presence
        if sorted_words.contains("banana") {
            println!("'banana' is in the set.");
        }

        // Iterate over elements (always sorted)
        for word in &sorted_words {
            print!("{} ", word); // Output: apple banana zebra
        }
        println!();
    }
    ```

### 8. `BinaryHeap<T>`

A `BinaryHeap<T>` is a priority queue implemented as a max-heap. This means that the largest element is always at the top (root) and can be retrieved in O(1) time. Pushing and popping elements take O(log N) time [2](https://doc.rust-lang.org/std/collections/index.html)[3](https://dev.to/alexmercedcoder/working-with-collections-in-rust-a-comprehensive-guide-3c9f).

*   **Common Use Cases:**
    *   Implementing a priority queue, where you always want to process the "most important" or "largest" element first [2](https://doc.rust-lang.org/std/collections/index.html).
    *   Algorithms like Dijkstra's shortest path or Prim's minimum spanning tree.
    *   Finding the k-largest elements in a stream of data.

*   **Code Example:**

    ```rust
    use std::collections::BinaryHeap;

    fn main() {
        let mut heap: BinaryHeap<i32> = BinaryHeap::new();

        // Push elements
        heap.push(10);
        heap.push(5);
        heap.push(20);
        heap.push(1);
        println!("Heap after pushes: {:?}", heap); // Output: [20, 10, 5, 1] (internal representation, not necessarily sorted)

        // Pop elements (always returns the largest)
        let max1 = heap.pop();
        println!("Popped max: {:?}", max1); // Output: Some(20)
        println!("Heap after first pop: {:?}", heap); // Output: [10, 5, 1]

        let max2 = heap.pop();
        println!("Popped max: {:?}", max2); // Output: Some(10)
        println!("Heap after second pop: {:?}", heap); // Output: [5, 1]
    }
    ```

---

## Using a Custom Hashing Algorithm

By default, `HashMap` and `HashSet` in Rust use a cryptographically secure, but potentially slower, hashing algorithm (`SipHash`). For certain applications, especially those not exposed to untrusted input where hash collision attacks are not a concern, you might want to use a faster, non-cryptographic hasher.

To use a custom hashing algorithm, you need to provide an implementation of the `BuildHasher` trait. A common choice for a faster non-cryptographic hash is the `fnv` crate.

### Steps to use a custom hasher:

1.  **Add the custom hasher crate to your `Cargo.toml`:**

    ```toml
    [dependencies]
    fnv = "1.0"
    ```

2.  **Import and use the custom hasher:**

    ```rust
    use std::collections::HashMap;
    use fnv::FnvBuildHasher;

    fn main() {
        // Create a HashMap using FnvBuildHasher
        let mut map: HashMap<String, i32, FnvBuildHasher> = HashMap::with_hasher(FnvBuildHasher::default());

        map.insert("apple".to_string(), 1);
        map.insert("banana".to_string(), 2);

        println!("Map with FNV hasher: {:?}", map);

        // You can also create a HashSet with a custom hasher
        use std::collections::HashSet;
        use fnv::FnvHashSet; // FnvHashSet is a type alias for HashSet<T, FnvBuildHasher>

        let mut set: FnvHashSet<i32> = FnvHashSet::default();
        set.insert(10);
        set.insert(20);

        println!("Set with FNV hasher: {:?}", set);
    }
    ```

**When to consider a custom hasher:**
*   **Performance-critical applications:** If profiling shows that hashing is a bottleneck and your keys are simple types (like integers or short strings).
*   **Known data:** When you are absolutely certain that the input data to your `HashMap`/`HashSet` is not malicious and will not attempt to cause hash collisions.
*   **Custom types:** If you have a custom type that implements `Hash` and `Eq`, and you want to optimize its hashing behavior beyond the default.

**Caution:** Using a non-cryptographic hasher like FNV can make your `HashMap` or `HashSet` vulnerable to denial-of-service attacks if an attacker can control the keys inserted into the map. They could craft keys that cause many hash collisions, degrading performance from O(1) to O(N). Always stick with the default hasher unless you fully understand the security implications.

---

## Conclusion

Rust's collection types provide powerful and efficient ways to manage data. While `Vec` and `HashMap` are excellent starting points for most tasks, understanding the unique characteristics of `VecDeque`, `LinkedList`, `BTreeMap`, `BTreeSet`, and `BinaryHeap` allows you to select the most appropriate data structure for optimal performance and correctness in specific scenarios. Remember to consider the trade-offs between memory usage, access patterns, and performance characteristics when making your choice.