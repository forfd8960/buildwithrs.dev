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

## The Routers

## How to Encode and Decode the user token

## How to apply auth in the request

## Summary
