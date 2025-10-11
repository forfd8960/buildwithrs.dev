---
sidebar_position: 1
---

# How to use `mpsc::channel` broadcast message between sender and receiver


Rust's standard library provides `std::sync::mpsc` (multi-producer, single-consumer) channels for safe, thread-safe communication between threads. This channel type allows **multiple senders** (producers) to send messages to a **single receiver** (consumer). It's not a true "broadcast" channel (which would send to multiple receivers); instead, all messages from multiple senders are queued and consumed sequentially by one receiver.

Key points:
- **Thread-safe**: Use with `std::thread` for inter-thread communication.
- **Cloning senders**: You can clone the sender handle to create multiple producers.
- **Blocking behavior**: Receivers block until a message arrives (use `try_recv` for non-blocking).
- **Closing the channel**: When all senders are dropped, the receiver gets `Err(RecvError)` on further `recv()` calls.

### Basic Usage

Here's a step-by-step guide with code examples.

#### 1. Creating a Channel
Use `std::sync::mpsc::channel()` to create a `(Sender<T>, Receiver<T>)` tuple, where `T` is the message type (e.g., `String`, `i32`).

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // Create a channel for String messages
    let (tx, rx) = mpsc::channel::<String>();

    // Clone the sender for multiple producers (optional)
    let tx_clone = tx.clone();

    // Spawn a thread for the receiver
    let handle = thread::spawn(move || {
        // Receive and process messages
        while let Ok(msg) = rx.recv() {
            println!("Received: {}", msg);
        }
        println!("Channel closed.");
    });

    // Sender 1: Send from main thread
    tx.send("Hello from sender 1!".to_string()).unwrap();

    // Sender 2: Send from another thread
    let sender2 = thread::spawn(move || {
        tx_clone.send("Hello from sender 2!".to_string()).unwrap();
    });

    // Wait for senders to finish
    sender2.join().unwrap();
    drop(tx);

    handle.join().unwrap();
}
```

**Output** (order may vary due to threading):
```sh
Received: Hello from sender 1!
Received: Hello from sender 2!
Channel closed.
```

#### 2. Broadcasting from Multiple Senders

To "broadcast" (i.e., send from multiple producers), clone the `Sender` and distribute clones to different threads. All messages are serialized in a FIFO queue for the single receiver.

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel::<i32>();

    // Create multiple sender threads
    for i in 0..3 {
        let tx_clone = tx.clone();
        thread::spawn(move || {
            // Simulate work
            thread::sleep(std::time::Duration::from_millis(100 * i as u64));
            tx_clone.send(i * 10).unwrap();
            println!("Sender {} sent {}", i, i * 10);
        });
    }

    // Drop the original tx to signal end (optional, but good practice)
    drop(tx);

    // Receiver in main thread
    for received in rx {
        println!("Received: {}", received);
    }
}
```

**Output** (approximate):

```sh
Sender 0 sent 0
Received: 0
Sender 1 sent 10
Received: 10
Sender 2 sent 20
Received: 20
```

#### 3. Non-Blocking Receive
Use `try_recv()` to avoid blocking:

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel::<String>();

    // Send a message
    tx.send("Non-blocking message".to_string()).unwrap();

    // Try to receive (succeeds immediately)
    match rx.try_recv() {
        Ok(msg) => println!("Got: {}", msg),
        Err(mpsc::TryRecvError::Empty) => println!("No message yet"),
        Err(mpsc::TryRecvError::Disconnected) => println!("Sender disconnected"),
    }

    // Simulate no message
    thread::sleep(Duration::from_millis(10));
    match rx.try_recv() {
        Ok(_) => println!("Unexpected message"),
        Err(mpsc::TryRecvError::Empty) => println!("Empty, as expected"),
        Err(_) => println!("Disconnected"),
    }
}
```
**Output**:
```sh
Got: Non-blocking message
Empty, as expected
```

#### 4. Error Handling
- `send()` returns `Result<(), SendError<T>>` (panics if receiver is dropped first).
- `recv()` returns `Result<T, RecvError>` (signals channel closure).
- Use `unwrap()` for simplicity in examples, but handle errors in production.

#### 5. Common Pitfalls and Tips
- **Deadlock**: Avoid sending to a channel from the same thread as the receiver without care.
- **Performance**: For high-throughput, consider `crossbeam-channel` crate (not std), but stick to `mpsc` for simplicity.
- **Sync vs. Async**: This is synchronous/blocking; for async, use `tokio::sync::mpsc`.
- **Generics**: Channels work with any `Send + 'static` type (most types qualify).

For more details, check the [official Rust docs](https://doc.rust-lang.org/std/sync/mpsc/index.html). If you have a specific use case or error, provide more context!