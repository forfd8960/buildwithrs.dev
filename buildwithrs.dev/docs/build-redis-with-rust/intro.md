---
sidebar_position: 1
---

# What is Redis

So, Redis is an open-source, in-memory data store commonly used for various purposes such as a cache, document database, vector database, streaming engine, and message broker. It supports complex data types like strings, hashes, lists, sets, sorted sets, and JSON, with atomic operations defined on these data types. Redis is known for its speed, simplicity, and performance.

## What will be implemented in this Project

So, In this build redis project with rust project, I will implement the following things for it.

- Redis client protocol: RESP parsing and serialization.
- Network: Listen TCP connection, and handle client request from client.
- Commands: Command Register and Command handler will be implemented to operate the data from client.
- Storage: In Memory Database and Data Structure.
- Persistence: RDB/AOF Saving and Loading.

## The Modules / Code Structure

```sh
ds-cache (main)> tree src/
src/
├── commands
│   └── mod.rs
├── config
│   └── mod.rs
├── main.rs
├── network
│   └── mod.rs
├── persistence
│   └── mod.rs
├── protocol
│   └── mod.rs
├── storage
│   └── mod.rs
└── utils.rs
```

- commands module will handle command define and handlers.
- config module will reponsible for redis config define and parsing.
- network module: will handle TCP connection handling.
- persistence module: will do the RDB/AOF, save data into disk.
- protocol module: will define and parsing / serialize of the request and response.
- storage module: will handle the data store and retrieve.
