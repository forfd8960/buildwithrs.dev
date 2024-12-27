---
slug: rust-build-simple-key-value-server-with-tokio
title: Rust Build Simple Key Value Server
authors: forfd8960
tags: [rust, tokio, simple-kv, server]
---


In this blog post we will check how to **build a simple key value server with tokio**.


<!-- truncate -->

## Add Dependency

```sh
cargo new --bin simple-kv

cargo add prost

cargo add prost-types bytes anyhow thiserror

cargo add tokio-util --features codec

cargo add dashmap

cargo add tokio --features rt --features rt-multi-thread --features macros  --features io-util --features net

cargo add tokio-stream features
```

## Add Build Dependency

```sh
cargo add --build anyhow prost-build
```

## Cargo Toml

```toml
[package]
name = "simple-kv"
version = "0.1.0"
edition = "2021"

[dependencies]
anyhow = "1.0.95"
bytes = "1.9.0"
dashmap = "6.1.0"
features = "0.10.0"
prost = "0.13.4"
prost-types = "0.13.4"
thiserror = "2.0.9"
tokio = { version = "1.42.0", features = ["rt", "rt-multi-thread", "macros", "io-util", "net"] }
tokio-stream = "0.1.17"
tokio-util = { version = "0.7.13", features = ["codec"] }

[build-dependencies]
anyhow = "1.0.95"
prost-build = "0.13.4"

```

## Build

`build.rs`:

```rust
use anyhow::Result;

fn main() -> Result<()> {
    prost_build::Config::new()
        .bytes(&["."])
        .out_dir("src/pb")
        .compile_protos(&["chat.proto"], &["."])?;

    Ok(())
}
```

## Proto

```proto
syntax = "proto3";

package kv;

message KeyVal {
    string key = 1;
    bytes val = 2;
}

message CmdGet {
    string key = 1;
}

message CmdGetResp {
    string status = 1;
    KeyVal key_val = 2;
}

message CmdSet {
    KeyVal key_val = 1;
}

message CmdSetResp {
    string status = 1;
    KeyVal key_val = 2;
}

message CmdDel {
    string key = 1;
}

message CmdDelResp {
    string status = 1;
}

message KvCommand {
    oneof Command {
        CmdGet get = 1;
        CmdSet set = 2;
        CmdDel del = 3;
    }
}

message KvCommandResp {
    oneof CommandResp {
        CmdGetResp get_resp = 1;
        CmdSetResp set_resp = 2;
        CmdDelResp del_resp = 3;
    }
}

```
