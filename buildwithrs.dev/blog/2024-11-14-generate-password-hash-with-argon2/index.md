---
slug: how-to-generate-password-hash-with-argon2
title: How to Generate Password Hash With Argon2
authors: forfd8960
tags: [rust, password-hash]
---

In the Axum web development with Rust, need to implement generate a safe password hash.

In this blog, I will introduce how to generate it with `Rust` `Argon2`.

## Cargo.toml

```toml
password-hash = "0.5.0"
argon2 = "0.5.3"
rand_core = { version = "0.6.4", features = ["std"] }
thiserror = "2.0.1"
```

## App Errors

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("pwd hash error: {0}")]
    PasswordHashError(String),
}
```

## Generate safe Password Hash with Argon2

* Generate Salt String with `SaltString`.
* Init `argon2` with `Argon2::default()`.
* Generate password hash with `argon2.hash_password`.

```rust
use argon2::{Argon2, PasswordHash, PasswordVerifier};
use password_hash::{PasswordHasher, SaltString};
use rand_core::OsRng;

use crate::errors::AppError;

pub fn gen_password_hash(password: &str) -> Result<String, AppError> {
    let salt = SaltString::generate(&mut OsRng);
    let argon2 = Argon2::default();
    let pwd_hash_res = argon2.hash_password(password.as_bytes(), &salt);

    match pwd_hash_res {
        Ok(hash) => Ok(hash.to_string()),
        Err(e) => Err(AppError::PasswordHashError(format!("{}", e))),
    }
}
```

## Verify the password

* New password hash with `PasswordHash::new`.
* If the password hash instance is init success, then use `argon2.verify_password` to verify the password, and the password-hash.
* If the verify result is `Ok(())`, then the password is verified.
* If the verify result is `Err(_)`, then the password is not right.

```rust
pub fn verify_password(password: &str, hash_value: &str) -> Result<bool, AppError> {
    let pwd_hash_res = PasswordHash::new(hash_value);

    match pwd_hash_res {
        Ok(pwd_hash) => {
            let argon2 = Argon2::default();
            match argon2.verify_password(password.as_bytes(), &pwd_hash) {
                Ok(()) => Ok(true),
                Err(_) => Ok(false),
            }
        }
        Err(e) => Err(AppError::PasswordHashError(format!("{}", e))),
    }
}
```

## Test

```rust
#[cfg(test)]
mod tests {
    use anyhow::Result;

    use super::{gen_password_hash, verify_password};

    #[test]
    fn test_hash_verify_password() -> Result<()> {
        let pwd_hash = gen_password_hash("abc132569")?;
        let result = verify_password("abc132569", &pwd_hash)?;
        assert!(result);
        Ok(())
    }

    #[test]
    fn test_bad_hash() -> Result<()> {
        let pwd_hash1 = gen_password_hash("def132569")?;
        let verify_res = verify_password("abc132569", &pwd_hash1)?;
        assert!(!verify_res);

        let verify_res = verify_password("def132569", "test-hash");
        assert!(verify_res.is_err());
        Ok(())
    }
}
```

Test result

```sh
(main)> cargo test handlers::utils::tests::test_hash_verify_password -- --exact
running 1 test
test handlers::utils::tests::test_hash_verify_password ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 9 filtered out; finished in 0.77s

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


> cargo test handlers::utils::tests::test_bad_hash -- --exact
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.64s
running 1 test
test handlers::utils::tests::test_bad_hash ... ok

```
