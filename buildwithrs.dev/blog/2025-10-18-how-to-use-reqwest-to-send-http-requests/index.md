---
slug: how-to-send-request-with-rust-reqwest
title: How to Send Request With Rust Reqwest
authors: forfd8960
tags: [rust, http, reqwest]
---

# How to Send Request With Rust Reqwest

In this guide, we'll walk through everything from basics to advanced patterns. Whether you're crafting a CLI tool or a full-fledged web service, you'll leave with practical code examples you can copy-paste and adapt. Let's dive in!

<!-- truncate -->

## What is Reqwest and Why Use It?

Reqwest is an ergonomic HTTP client for Rust, built on top of Hyper and Tokio. It handles the nitty-gritty details like redirects, TLS encryption, proxies, and automatic decompression, so you can focus on your app's logic.

### Key Use Cases
- **API Integration**: Fetching JSON from RESTful services (e.g., weather APIs or GitHub endpoints).
- **Form Submissions**: Posting user data or login credentials.
- **File Handling**: Uploading images or documents to servers.
- **Web Scraping**: Downloading and parsing HTML/XML.
- **Microservices**: Efficient communication in async-heavy backends.
- **CLI Tools**: Simple, blocking requests for scripts.

Reqwest is async-first but offers a blocking API for simpler scenarios. It's perfect for any networked Rust app—no more wrestling with raw sockets!

## Installation and Setup

Add Reqwest to your `Cargo.toml`:

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json", "multipart"] }
tokio = { version = "1", features = ["full"] }  # For async
serde_json = "1"  # Optional, for JSON handling
```

Run `cargo build` to fetch dependencies. For JSON support, enable the `json` feature (uses Serde). For file uploads, add `multipart`.

Import basics: `use reqwest;`. Always create a `Client` for reusable connections (enables pooling):

```rust
let client = reqwest::Client::new();
```

## Basic HTTP Requests

Reqwest makes sending GET and POST requests a breeze. Here's a simple async GET:

```rust
use reqwest;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let body = reqwest::get("https://www.rust-lang.org")
        .await?
        .text()
        .await?;
    println!("Body: {}", &body[..300]);
    Ok(())
}
/*
Body: <!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8">
    <title>
            Rust Programming Language
        </title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <meta name="description" content="A language empowering everyone to build reliable 
*/
```

For POST with a raw body:

```rust
use reqwest;

/*
Status: 200 OK
Body: {
  "args": {}, 
  "data": "Hello, server!", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*//*", 
    "Content-Length": "14", 
    "Host": "httpbin.org", 
    "X-Amzn-Trace-Id": "Root=1-68f38a7d-3b3a8bdd28e23c095f4ce675"
  }, 
  "json": null, 
  "origin": "xxx", 
  "url": "http://httpbin.org/post"
}
*/

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::new();
    let res = client
        .post("http://httpbin.org/post")
        .body("Hello, server!")
        .send()
        .await?;

    // Status: 200 OK
    println!("Status: {}", res.status());
    println!("Body: {}", res.text().await?);
    Ok(())
}
```

And with JSON (define structs for typing, or use `serde_json::Value`):

```rust
use reqwest;
use std::collections::HashMap;
use serde_json::Value;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut map = HashMap::new();
    map.insert("lang", "rust");
    map.insert("body", "json");

    let client = reqwest::Client::new();
    let res = client
        .post("http://httpbin.org/post")
        .json(&map)
        .send()
        .await?;

    let json: Value = res.json().await?;
    // Echoed JSON: "rust"
    println!("Echoed JSON: {}", json["json"]["lang"]);
    Ok(())
}
```

These use `httpbin.org` for testing— it echoes your requests back.

## Synchronous vs. Asynchronous API

By default, Reqwest is async (requires Tokio). For sync/blocking code (no `async` boilerplate), import `reqwest::blocking`:

```rust
use reqwest::blocking;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let body = blocking::get("https://www.rust-lang.org")?
        .text()?;
    println!("Body: {}", body);
    Ok(())
}
```

The blocking API is great for threads or simple scripts but blocks the thread during I/O. Use async for concurrency.

## Passing URL Parameters (Query Strings)

Append query params with `.query(&params)`—Reqwest URL-encodes them automatically:

```rust
use reqwest;
use std::collections::HashMap;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut params = HashMap::new();
    params.insert("lang", "rust");
    params.insert("version", "1.75");

    let client = reqwest::Client::new();
    let res = client
        .get("http://httpbin.org/get")
        .query(&params)
        .send()
        .await?;

    // Query params echoed: Object {"lang": String("rust"), "version": String("1.75")}
    let json: serde_json::Value = res.json().await?;
    println!("Query params echoed: {:?}", json["args"]);
    Ok(())
}
```

This turns `/get?lang=rust&version=1.75` into a filtered API call.

## Sending Form Data

For `application/x-www-form-urlencoded` forms, use `.form(&data)`:

```rust
use reqwest;
use std::collections::HashMap;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut form_data = HashMap::new();
    form_data.insert("title", "Hello");
    form_data.insert("content", "World");

    let client = reqwest::Client::new();
    let res = client
        .post("http://httpbin.org/post")
        .form(&form_data)
        .send()
        .await?;

    // Form data echoed: Object {"content": String("World"), "title": String("Hello")}
    let json: serde_json::Value = res.json().await?;
    println!("Form data echoed: {:?}", json["form"]);
    Ok(())
}
```

Perfect for logins or surveys.

## Uploading Files

Enable `multipart` and build a form with `.file()`:

```rust
use reqwest;
use std::fs;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file_path = "example.txt";  // Assume file exists
    let file_data = fs::read(file_path)?;

    let mut form = reqwest::multipart::Form::new();
    form = form.file("file", file_data.as_slice())?;
    form = form.text("description", "A sample file upload");

    let client = reqwest::Client::new();
    let res = client
        .post("http://httpbin.org/post")
        .multipart(form)
        .send()
        .await?;

    let json: serde_json::Value = res.json().await?;
    println!("Uploaded file info: {:?}", json["files"]["file"]);
    Ok(())
}
```

For large files, stream with `tokio::fs` to save memory.

## Custom Headers

Override defaults like User-Agent with `.headers()` or `.header()`:

```rust
use reqwest;
use reqwest::header::{HeaderMap, USER_AGENT};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut headers = HeaderMap::new();
    headers.insert(USER_AGENT, "MyRustBot/1.0".parse()?);
    headers.insert("X-Custom-Header", "secret-value".parse()?);

    let client = reqwest::Client::new();
    let res = client
        .get("http://httpbin.org/headers")
        .headers(headers)
        .send()
        .await?;

    let json: serde_json::Value = res.json().await?;
    println!("Headers echoed: {:?}", json["headers"]["User-Agent"]);
    Ok(())
}
```

Chain with other methods for full control.

## Authentication

Secure requests with Basic, Bearer, or custom headers.

### Basic Auth
```rust
use reqwest;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::new();
    let res = client
        .get("http://httpbin.org/basic-auth/user/pass")
        .basic_auth("user", Some("pass"))
        .send()
        .await?;

    let json: serde_json::Value = res.json().await?;
    println!("Authenticated: {}", json["authenticated"]);
    Ok(())
}
```

### Bearer Token
```rust
use reqwest;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let token = "your-secret-jwt-or-api-key";

    let client = reqwest::Client::new();
    let res = client
        .get("http://httpbin.org/bearer")
        .bearer_auth(token)
        .send()
        .await?;

    let json: serde_json::Value = res.json().await?;
    println!("Token echoed: {}", json["access_token"]);
    Ok(())
}
```

### Custom API Key
```rust
use reqwest;
use reqwest::header::HeaderMap;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let api_key = "sk-12345";

    let mut headers = HeaderMap::new();
    headers.insert("X-API-Key", api_key.parse()?);

    let client = reqwest::Client::new();
    let res = client
        .get("http://httpbin.org/headers")
        .headers(headers)
        .send()
        .await?;

    let json: serde_json::Value = res.json().await?;
    println!("API Key echoed: {}", json["headers"]["X-Api-Key"]);
    Ok(())
}
```

Load secrets from env vars in production!

## Tips for Production

- **Timeouts**: `Client::builder().timeout(Duration::from_secs(10))`.
- **Error Handling**: Use `anyhow` or `thiserror` for richer errors.
- **Retries**: Pair with `reqwest-retry` for resilience.
- **Testing**: Mock with `wiremock` for unit tests.
