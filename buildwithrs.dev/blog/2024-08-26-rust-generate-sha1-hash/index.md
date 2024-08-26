---
slug: rust-generate-sha1-hash
title: Rust Generate SHA1 Hash
authors: forfd8960
tags: [rust, sha1, hash]
---

## the crate

```toml
[dependencies]
sha1-checked = "0.10.0"
base16ct = { version="0.2.0", features=["alloc"] }
```

## Generate SHA1 Hash bytes

* build sha1 hasher with `Sha1::new()`
* update the hasher with `hasher.update(content)`
* call `hasher.try_finalize()` to get the hash
* generate hash bytes with 

```rust
let arr = hf.hash();
arr.to_vec()
```

```rust
use sha1_checked::{Digest, Sha1};

pub fn compute_hash(t: &ObjectType, content: &[u8]) -> Vec<u8> {
    let mut hasher = Sha1::new();
    hasher.update(object_type_bytes(t));
    hasher.update(b" ");
    hasher.update(format!("{}", content.len()).as_bytes());
    hasher.update(b"\0");
    hasher.update(content);

    let hf = hasher.try_finalize();
    let arr = hf.hash();
    arr.to_vec()
}
```

## Generate SHA1 Hash String

```rust
fn gen_hash_str(hash_bytes: &[u8]) -> String {
    base16ct::lower::encode_string(hash_bytes)
}

```
