---
slug: how-to-use-rust-serde
title: How to use Rust Serde
authors: forfd8960
tags: [rust, serde]
---

# What is the Rust `serde`

Serde is the de facto standard serialization framework for Rust. It provides:
- A set of traits (`Serialize`, `Deserialize`) for converting Rust data structures to and from other representations.
- Powerful derive-macros to automatically implement (de)serialization for most types.
- A separation between data models and formats: format-specific crates (serde_json, serde_yaml, toml, bincode, etc.) implement the actual encoding/decoding using Serde's traits.

Why Serde matters:
- Type-safe serialization and deserialization.
- Works with most of the Rust type system (structs, enums, generics, lifetimes).
- Highly performant (zero-copy when possible, streaming formats).
- Extensible: you can implement custom behavior via attributes, helper functions, or by writing your own serializer/deserializer implementation.

<!-- truncate -->

---

## Common use cases

- Configuration files (TOML, YAML, JSON).
- Inter-process/network protocols (JSON, MessagePack, CBOR, Protobuf via prost/serde bridges).
- Persisting structured data (binary formats like bincode or CBOR).
- Embedding/serializing binary blobs (images, keys) safely.
- Converting Rust structs to formats consumed by web clients (REST APIs).
- Custom text/binary wire formats for low-level systems or embedded devices.

---

## Quick start: Cargo.toml

Add serde and one or more format crates:

```toml
[dependencies]
serde = { version = "1.0.228", features = ["derive"] }
serde_json = "1.0.145"
serde_yaml = "0.9"
toml = "0.7"         # for toml serialization/deserialization
bincode = "2.0.1"    # simple binary format
```

Enable `derive` on `serde` to get `#[derive(Serialize, Deserialize)]`.

---

## Basic usage: structs and enums

Example struct that can be serialized/deserialized with any Serde format:

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct Config {
    name: String,
    enabled: bool,
    timeout_ms: Option<u64>,
}
```

Serialize to JSON:

```rust
let cfg = Config { name: "demo".into(), enabled: true, timeout_ms: Some(300) };
let json = serde_json::to_string(&cfg)?;          // compact
let pretty = serde_json::to_string_pretty(&cfg)?; // pretty-printed
let from: Config = serde_json::from_str(&json)?;
```

Same struct, YAML:

```rust
let yaml = serde_yaml::to_string(&cfg)?;
let from_yaml: Config = serde_yaml::from_str(&yaml)?;
```

TOML (commonly used for config files):

```rust
let toml = toml::to_string(&cfg)?;
let from_toml: Config = toml::from_str(&toml)?;
```

### Full code

```rust
use bincode::{config, Decode, Encode};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct Config {
    name: String,
    enabled: bool,
    timeout_ms: Option<u64>,
}

#[derive(Encode, Decode, Debug, PartialEq)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() -> anyhow::Result<()> {
    to_json()?;
    to_yaml()?;

    to_toml()?;

    encode_decode_bincode()?;
    Ok(())
}

fn to_json() -> anyhow::Result<()> {
    let cfg = Config {
        name: "demo".into(),
        enabled: true,
        timeout_ms: None,
    };
    let json = serde_json::to_string(&cfg)?; // compact
    let pretty = serde_json::to_string_pretty(&cfg)?; // pretty-printed

    println!("JSON output:\n{}", json);
    println!();

    println!("Pretty JSON output:\n{}", pretty);
    println!();

    let from: Config = serde_json::from_str(&json)?;
    println!("Deserialized from JSON: {:?}", from);
    println!();
    Ok(())
}

fn to_yaml() -> anyhow::Result<()> {
    let cfg = Config {
        name: "demo".into(),
        enabled: true,
        timeout_ms: Some(300),
    };
    let yaml = serde_yaml::to_string(&cfg)?;
    println!("YAML output:\n{}", yaml);

    let from: Config = serde_yaml::from_str(&yaml)?;
    println!("Deserialized from YAML: {:?}", from);
    Ok(())
}

fn to_toml() -> anyhow::Result<()> {
    let cfg = Config {
        name: "demo".into(),
        enabled: true,
        timeout_ms: Some(300),
    };

    println!();

    let toml = toml::to_string(&cfg)?;
    println!("TOML output:\n{}", toml);

    let from: Config = toml::from_str(&toml)?;
    println!("Deserialized from TOML: {:?}", from);
    Ok(())
}

fn encode_decode_bincode() -> anyhow::Result<()> {
    let person = Person {
        name: "Alice".to_string(),
        age: 30,
        email: "alice@example.com".to_string(),
    };

    println!();

    // Encode to Vec<u8>
    let encoded = bincode::encode_to_vec(&person, config::standard())?;
    println!("Encoded bytes: {:?}", encoded);
    println!("Size: {} bytes", encoded.len());

    // Decode from bytes
    let (decoded, _): (Person, usize) = bincode::decode_from_slice(&encoded, config::standard())?;
    println!("Decoded: {:?}", decoded);

    assert_eq!(person, decoded);
    println!();
    Ok(())
}
```

```sh
JSON output:
{"name":"demo","enabled":true,"timeout_ms":null}

Pretty JSON output:
{
  "name": "demo",
  "enabled": true,
  "timeout_ms": null
}

Deserialized from JSON: Config { name: "demo", enabled: true, timeout_ms: None }

YAML output:
name: demo
enabled: true
timeout_ms: 300

Deserialized from YAML: Config { name: "demo", enabled: true, timeout_ms: Some(300) }

TOML output:
name = "demo"
enabled = true
timeout_ms = 300

Deserialized from TOML: Config { name: "demo", enabled: true, timeout_ms: Some(300) }
Serialized json: {"desc":"Example data","bytes":"deadbeef"}
Deserialized from json: Data { desc: "Example data", bytes: [222, 173, 190, 239] }

Encoded bytes: [5, 65, 108, 105, 99, 101, 30, 17, 97, 108, 105, 99, 101, 64, 101, 120, 97, 109, 112, 108, 101, 46, 99, 111, 109]
Size: 25 bytes
Decoded: Person { name: "Alice", age: 30, email: "alice@example.com" }
```

---



## Binary formats

For compact, efficient binary serialization for storage or low-latency transport, use crates like `bincode`, `postcard` (embedded), `rmp-serde` (MessagePack) or `serde_cbor`:

bincode:

```rust
use bincode::{config, Decode, Encode};

#[derive(Encode, Decode, Debug, PartialEq)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn encode_decode_bincode() -> anyhow::Result<()> {
    let person = Person {
        name: "Alice".to_string(),
        age: 30,
        email: "alice@example.com".to_string(),
    };

    println!();

    // Encode to Vec<u8>
    let encoded = bincode::encode_to_vec(&person, config::standard())?;
    println!("Encoded bytes: {:?}", encoded);
    println!("Size: {} bytes", encoded.len());

    // Decode from bytes
    let (decoded, _): (Person, usize) = bincode::decode_from_slice(&encoded, config::standard())?;
    println!("Decoded: {:?}", decoded);

    assert_eq!(person, decoded);
    println!();
    Ok(())
}
```

---

## Useful serde attributes

Serde has many attributes to control (de)serialization. A few common ones:

- `#[serde(rename = "otherName")]` — rename field in serialized representation.
- `#[serde(default)]` — provide Default value if field absent.
- `#[serde(skip_serializing_if = "Option::is_none")]` — omit fields.
- `#[serde(skip)]` — skip both serialize and deserialize.
- `#[serde(with = "module")]` — use custom (de)serializer functions in `module`.
- `#[serde(serialize_with = "func", deserialize_with = "func")]` — per-field function.
- `#[serde(flatten)]` — merge a nested map into parent map.
- `#[serde(tag = "type", content = "data")]` — enum tagging (internal/external/adjacent/untagged).
- `#[serde(borrow)]` and lifetime-aware `Deserialize<'de>` types for zero-copy.

Example: default and skip:

```rust
#[derive(Serialize, Deserialize)]
struct Item {
    id: u64,
    #[serde(default)]
    tags: Vec<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    note: Option<String>,
}
```

---

## Working with binary data in text formats

If you need to embed raw bytes in JSON/TOML/YAML you can:
- Use base64 crate and custom (de)serializer functions.
- Use `serde_bytes` (for formats that can handle raw bytes).
- Serialize as hex via helper functions or newtype.

Example: hex encoding for `Vec<u8>` using helper functions

```rust
mod hex_serde {
    use serde::{Serializer, Deserializer, Deserialize, Serialize};
    use hex::{FromHex, ToHex};

    pub fn serialize<S>(bytes: &Vec<u8>, s: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let hex = bytes.encode_hex::<String>();
        s.serialize_str(&hex)
    }

    pub fn deserialize<'de, D>(d: D) -> Result<Vec<u8>, D::Error>
    where
        D: Deserializer<'de>,
    {
        let s = String::deserialize(d)?;
        Vec::from_hex(&s).map_err(serde::de::Error::custom)
    }
}

#[derive(Serialize, Deserialize)]
struct Packet {
    #[serde(with = "hex_serde")]
    payload: Vec<u8>,
}
```

You can also use base64 with the `base64` crate in similar fashion.

---

## Custom serializer and deserializer for a type

Often you don't need to implement `Serialize`/`Deserialize` manually: use `serialize_with`/`deserialize_with` or `#[serde(with = "...")]`. But sometimes you want full control. Two common patterns:

1) Per-field custom (de)serializer functions (recommended for most use cases).
2) Implement `Serialize` and `Deserialize` for your type manually (for complex cases or formats).

Example — implement custom functions for hex (re-using previous example):

```rust
#[derive(Serialize, Deserialize, Debug)]
struct Data {
    desc: String,
    #[serde(serialize_with = "serialize_hex", deserialize_with = "deserialize_hex")]
    bytes: Vec<u8>,
}

fn serialize_hex<S>(bytes: &Vec<u8>, serializer: S) -> Result<S::Ok, S::Error>
where
    S: serde::Serializer,
{
    let s = hex::encode(bytes);
    serializer.serialize_str(&s)
}

fn deserialize_hex<'de, D>(deserializer: D) -> Result<Vec<u8>, D::Error>
where
    D: serde::Deserializer<'de>,
{
    let s = String::deserialize(deserializer)?;
    hex::decode(&s).map_err(serde::de::Error::custom)
}

fn work_with_data_hex() -> anyhow::Result<()> {
    let data = Data {
        desc: "Example data".into(),
        bytes: vec![0xde, 0xad, 0xbe, 0xef],
    };

    let encode = serde_json::to_string(&data)?;
    println!("Serialized json: {}", encode);

    let decoded: Data = serde_json::from_str(&encode)?;
    println!("Deserialized from json: {:?}", decoded);

    Ok(())
}
```
**Result**
```sh
Serialized json: {"desc":"Example data","bytes":"deadbeef"}
Deserialized from json: Data { desc: "Example data", bytes: [222, 173, 190, 239] }
```

## Best practices

- Prefer derive-based (de)serialization for most types.
- Use `serde_bytes` for raw byte arrays, and base64/hex only when interoperating with systems expecting textual encodings.
- Use `serde(with = "...")` to reuse a module of utility (de)serializers.
- Keep wire formats stable (use explicit renames and versioning if needed).
- For high-performance binary formats, benchmark `bincode`, `postcard`, `rmp`, `cbor` for your workload.
- When adding a new format, try to reuse existing crates rather than writing your own unless necessary.

---
## Resources / references

- Serde book: https://serde.rs/ (excellent documentation and guide)
- serde_json, serde_yaml crates docs
- Examples in the `serde` repository and format crates for advanced custom serializer/deserializer implementations.
- `serde_with` crate — collection of common (de)serializers for common patterns.

---

## Conclusion

Serde gives Rust strong, flexible, and high-performance serialization. Whether you're moving JSON over HTTP, writing compact binary blobs to disk, or implementing a custom wire format, Serde provides the primitives and patterns to make it safe and efficient. Start with `derive`-based (de)serialization, use serde attributes for customization, reach for per-field helpers for binary/text conversions, and implement full serializers/deserializers only when you need to support a new on-the-wire format.