---
slug: how-to-implement-jwt-auth-in-axum
title: How to Implement JWT Auth in Axum
authors: forfd8960
tags: [rust, axum, jwt]
---

## What is JWT

JWT, or JSON Web Token, is a compact and self-contained way of securely transmitting information between parties as a JSON object.

Here are the key points about JWT:

### Structure

A JWT consists of three parts separated by dots (.):

* Header
* Payload
* Signature

The typical format looks like this: xxxxx.yyyyy.zzzzz 1

### Components

* Header: Contains metadata about the token, such as the type of token and the hashing algorithm used.
* Payload: Contains claims or statements about the user and additional data.
* Signature: Ensures the token hasn't been altered. It's created by combining the encoded header, encoded payload, and a secret 1.

### Use Cases

* Authentication: The most common scenario. Once a user logs in, each subsequent request includes the JWT, allowing access to routes, services, and resources permitted with that token 1.
* Information Exchange: JWTs can securely transmit information between parties, as they can be signed to ensure the sender's authenticity and that the content hasn't been tampered with 1.

### Benefits

* Compact: JWTs can be sent through URLs, POST parameters, or inside HTTP headers, and are transmitted quickly due to their small size 1.
* Self-contained: The payload contains all the required information about the user, avoiding the need to query the database multiple times 1.
* Widely supported: JWTs are supported across different platforms and languages 2.

### Security

* JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA 1.
* While JWTs can be encrypted to provide secrecy between parties, they are typically used as signed tokens 1.

### Claims

   JWTs contain claims, which are statements about the entity (typically the user) and additional data. There are three types of claims:

* Registered claims: Predefined claims like iss (issuer), exp (expiration time), sub (subject), aud (audience) 1.
* Public claims: Defined at will by those using JWTs 1.
* Private claims: Custom claims to share information between parties 1.

### Workflow

1. The application requests authorization from the authorization server.
2. Upon authorization, the server returns an access token (JWT) to the application.
3. The application uses the token to access protected resources (like APIs).

## The code structure

```sh
➜  jwt-auth git:(main) tree .
.
├── Cargo.toml
├── README.md
├── create_blog.png
├── keys
│   ├── private.pem
│   └── public.pem
├── signin.png
├── src
│   ├── auth.rs
│   ├── config.rs
│   ├── errors.rs
│   ├── jwt.rs
│   ├── lib.rs
│   ├── main.rs
│   ├── middleware
│   │   └── mod.rs
│   └── models
│       ├── blog.rs
│       ├── mod.rs
│       └── user.rs
```

## Dependency

`Cargo.toml`

```toml
[package]
name = "jwt-auth"
version = "0.1.0"
edition = "2021"

[dependencies]
jwt-simple = { workspace = true }
serde = { workspace = true }
serde_json = { workspace = true }
uuid ={ workspace = true }
anyhow = { workspace = true }
axum = {workspace = true}
axum-extra = { workspace = true}
tokio = { workspace = true }
thiserror = {workspace = true }
```

## The models

* `models/mod.rs`

```rust
pub mod blog;
pub mod user;
```

* blog.rs

```rust
use serde::Serialize;
use uuid::Uuid;

use super::user::User;

#[derive(Debug, Serialize)]
pub struct Blog {
    id: String,
    author: User,
    title: String,
    content: String,
}

impl Blog {
    pub fn new(author: User, title: String, content: String) -> Self {
        Self {
            id: Uuid::new_v4().to_string(),
            author,
            title,
            content,
        }
    }
}

```

* user.rs

```rust
use serde::{Deserialize, Serialize};
use uuid::Uuid;

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct User {
    user_id: String,
    user_name: String,
    email: String,
}

impl User {
    pub fn new(user_name: String, email: String) -> Self {
        Self {
            user_id: Uuid::new_v4().to_string(),
            user_name,
            email,
        }
    }
}

```

## Generate the public(encode) and private(decode) keys

```sh
openssl genpkey -algorithm ed25519 -out private.pem

✗ cat private.pem
-----BEGIN PRIVATE KEY-----
MC4CAQAwBQYDK2VwBCIEICOsPn3KRn4b6dSDm6BsWTMPLxr4DsydA72X2A7xhPHj
-----END PRIVATE KEY-----


openssl pkey -in private.pem -pubout -out public.pem
```

## The Routers

in `lib.rs`, we create the routes:

```rust
use axum::{
    extract::State, http::StatusCode, middleware::from_fn_with_state, response::IntoResponse,
    routing::post, Extension, Json, Router,
};
use config::AppConfig;
use errors::AppError;
use jwt::{DecodingKey, EncodingKey};
use middleware::verify_token;
use models::{blog::Blog, user::User};
use serde::{Deserialize, Serialize};
use std::{ops::Deref, sync::Arc};

pub mod auth;
pub mod config;
pub mod errors;
pub mod jwt;
pub mod middleware;
pub mod models;

#[derive(Clone)]
pub struct AppState {
    state: Arc<AppStateInner>,
}

#[derive(Clone)]
pub struct AppStateInner {
    pk: EncodingKey,
    dk: DecodingKey,
}

impl AppState {
    pub fn new(config: &AppConfig) -> Result<Self, AppError> {
        let pk = EncodingKey::load(&config.private_pem)?;
        let dk = DecodingKey::load(&config.public_pem)?;

        Ok(Self {
            state: Arc::new(AppStateInner { pk, dk }),
        })
    }
}

impl Deref for AppState {
    type Target = AppStateInner;
    fn deref(&self) -> &Self::Target {
        &self.state
    }
}

#[derive(Debug, Serialize, Deserialize)]
pub struct SignInUser {
    username: String,
    password: String,
}

#[derive(Debug, Serialize)]
pub struct SignInResponse {
    token: String,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct CreateBlog {
    title: String,
    content: String,
}

pub async fn get_router(state: AppState) -> Result<Router, AppError> {
    let api = Router::new()
        .route("/blog", post(create_blog))
        .layer(from_fn_with_state(state.clone(), verify_token::<AppState>))
        .route("/signin", post(signin_handler))
        .with_state(state);

    Ok(api)
}

#[axum::debug_handler]
async fn signin_handler(
    State(state): State<AppState>,
    Json(input): Json<SignInUser>,
) -> Result<impl IntoResponse, AppError> {
    if input.username.is_empty() {
        return Ok((StatusCode::BAD_REQUEST, "invalid user name").into_response());
    }

    if input.password.len() < 10 {
        return Ok((StatusCode::BAD_REQUEST, "too short password").into_response());
    }

    let user = User::new(input.username, input.password);
    let token = state.pk.sign(user)?;
    Ok((StatusCode::OK, Json(SignInResponse { token })).into_response())
}

#[axum::debug_handler]
async fn create_blog(
    Extension(user): Extension<User>,
    State(_state): State<AppState>,
    Json(create_blog): Json<CreateBlog>,
) -> Result<impl IntoResponse, AppError> {
    let blog = Blog::new(user, create_blog.title, create_blog.content);
    Ok((StatusCode::OK, Json(blog)).into_response())
}

```

## How to Encode and Decode the user token

we use `jwt-simple` to encoding and decoding user token

`jwt.rs`

```rust
use jwt_simple::{claims::Claims, prelude::*};

use crate::models::user::User;

const JWT_DURATION: u64 = 64 * 64 * 24 * 7;
const JWT_ISS: &str = "how-to-jwt-auth";
const JWT_AUD: &str = "jwt-user";

#[derive(Clone)]
pub struct EncodingKey(Ed25519KeyPair);

#[derive(Debug, Clone)]
pub struct DecodingKey(Ed25519PublicKey);

impl EncodingKey {
    pub fn load(pem: &str) -> Result<Self, jwt_simple::Error> {
        Ok(Self(Ed25519KeyPair::from_pem(pem)?))
    }

    pub fn sign(&self, user: impl Into<User>) -> Result<String, jwt_simple::Error> {
        let claims: JWTClaims<User> = Claims::with_custom_claims(user.into(), Duration::from_secs(JWT_DURATION));

        let claims = claims.with_issuer(JWT_ISS).with_audience(JWT_AUD);
        self.0.sign(claims)
    }
}

impl DecodingKey {
    pub fn load(pem: &str) -> Result<Self, jwt_simple::Error> {
        Ok(Self(Ed25519PublicKey::from_pem(pem)?))
    }

    pub fn verify(&self, token: &str) -> Result<User, jwt_simple::Error> {
        let opts = VerificationOptions {
            allowed_issuers: Some(HashSet::from_strings(&[JWT_ISS])),
            allowed_audiences: Some(HashSet::from_strings(&[JWT_AUD])),
            ..Default::default()
        };

        let claims = self.0.verify_token::<User>(token, Some(opts))?;
        Ok(claims.custom)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use crate::models::user::User;
    use anyhow::Result;

    #[test]
    fn test_generate_keys() -> Result<()> {
        let encoding_pem = include_str!("../keys/private.pem");
        let decoding_pem = include_str!("../keys/public.pem");
        let ek = EncodingKey::load(encoding_pem)?;
        let dk = DecodingKey::load(decoding_pem)?;

        let user = User::new("AlexZ".to_string(), "alex@example.com".to_string());

        let token = ek.sign(user.clone())?;
        println!("sign token: {:?}", token);

        let verify_user = dk.verify(&token)?;
        println!("verify_user: {:?}", verify_user);

        assert_eq!(user, verify_user);
        Ok(())
    }
}

```

## How to apply auth in the request

we implment verity_token middleware in the `middleware/mod.rs`

* first get the `bearer` token from header.
* then verify the token and extract user from the token insert it to request.
* if the token is invalid, then return `FORBIDDEN` error.


```rust
use crate::auth::TokenVeirfy;
use axum::{
    extract::{FromRequestParts, Request, State},
    http::StatusCode,
    middleware::Next,
    response::{IntoResponse, Response},
};
use axum_extra::{
    headers::{authorization::Bearer, Authorization},
    TypedHeader,
};

pub async fn verify_token<T>(State(state): State<T>, req: Request, next: Next) -> Response
where
    T: TokenVeirfy + Clone + Send + Sync + 'static,
{
    let (mut parts, body) = req.into_parts();
    let token =
        match TypedHeader::<Authorization<Bearer>>::from_request_parts(&mut parts, &state).await {
            Ok(TypedHeader(Authorization(bearer))) => bearer.token().to_string(),
            Err(e) => {
                let msg = format!("parse authorization error: {}", e);
                return (StatusCode::UNAUTHORIZED, msg).into_response();
            }
        };

    let req = match state.vetify(&token) {
        Ok(user) => {
            let mut req = Request::from_parts(parts, body);
            req.extensions_mut().insert(user);
            req
        }
        Err(e) => {
            let msg = format!("verify token failed: {:?}", e);
            return (StatusCode::FORBIDDEN, msg).into_response();
        }
    };

    next.run(req).await
}

```

## Summary

the general flow of jwt auth(with `jwt-simple` crate) is:

* generate public and private keys with `openssl`
* use `Ed25519KeyPair.sign` to generate JWT Token.
* use `Ed25519PublicKey.verify_token` to decode the custom claims from the token: `self.0.verify_token::<User>(token, Some(opts))?`.
* implement the signin route to generate the token if the user verified.
* implement the middleware in Axum to verify and decode the token. `middleware/mod.rs`
* implement the proteced route to use the extracted custom claims(`User`).

## Code

[Code Repo](https://github.com/forfd8960/how-to-do-x-in-rust/tree/main/jwt-auth)
