---
slug: how-to-do-database-query-with--sqlx-in-rust
title: How to Do Database Query with SQLX in Rust
authors: forfd8960
tags: [rust, sqlx, database]
---

## Introduction

* What is SQLx?

![alt text](sqlx.png)

* Key features of SQLx
![alt text](sqlx_features.png)


## Setting Up Your Rust Project

```sh
> cargo new --bin sqlx-emaple
    Creating binary (application) `sqlx-emaple` package
note: see more `Cargo.toml` keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
```


* Adding dependencies (sqlx, tokio, etc.)

```toml
[dependencies]
anyhow = "1.0.92"
serde = { version = "1.0.214", features = ["derive"] }
chrono = { version = "0.4.38", features = ["serde"] }
sqlx = { version = "0.8.2", features = [
  "chrono",
  "postgres",
  "runtime-tokio",
  "tls-rustls",
] }
thiserror = "2.0.1"
tokio = { version = "1.41.1", features = ["rt", "rt-multi-thread", "macros"] }
tracing = "0.1.40"
tracing-subscriber = { version = "0.3.18", features = ["env-filter"] }
```

* Setting up a local database instance

```sh
~> createdb my_todo_list --template=template0
 ~> psql -U db_manager -d my_todo_list
psql (14.12 (Homebrew))
Type "help" for help.

my_todo_list=> \q
 ~> psql -U db_manager -d my_todo_list
psql (14.12 (Homebrew))
Type "help" for help.

my_todo_list=> \dt
Did not find any relations.
my_todo_list=> \conninfo
You are connected to database "my_todo_list" as user "db_manager" via socket in "/tmp" at port "5432".
my_todo_list=>
```

* Configuring the connection string

```sh
sqlx-emaple (master)> touch .env

# set this env data in .env file
DATABASE_URL = "postgres://db_manager:super_admin8801@localhost:5432/my_todo_list"
```

## Connecting to the Database

* Establishing a Connection
* Using `sqlx::Pool` to manage connections

```rust
use sqlx::PgPool;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    println!("Hello, world!");

    let pool = PgPool::connect("postgres://db_manager:xxxx@localhost:5432/my_todo_list")
        .await?;
    println!("connected database: {:?}", pool);
    Ok(())
}
```

```sh
sqlx-emaple (master)> cargo run .

Hello, world!
connected database: Pool { size: 1, num_idle: 1, is_closed: false, options: PoolOptions { max_connections: 10, min_connections: 0, connect_timeout: 30s, max_lifetime: Some(1800s), idle_timeout: Some(600s), test_before_acquire: true } }
```

## Create Table in DB

* install `sqlx-cli`

```sh
> cargo install sqlx-cli

Finished `release` profile [optimized] target(s) in 1m 48s
Replacing ~/.cargo/bin/cargo-sqlx
Replacing ~/.cargo/bin/sqlx
Replaced package `sqlx-cli v0.7.4` with `sqlx-cli v0.8.2` (executables `cargo-sqlx`, `sqlx`)
```

* Sqlx Migrate

```sh
sqlx-emaple (master)> sqlx migrate add initial
Creating migrations/20241116140325_initial.sql
```

* Create Tables SQL

```sql
-- Add migration script here
CREATE TABLE IF NOT EXISTS users (
    id bigserial PRIMARY KEY,
    username VARCHAR(64) NOT NULL UNIQUE,
    email VARCHAR(64) NOT NULL UNIQUE,
    created_at timestamptz DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS todos (
    id bigserial PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),
    title text NOT NULL,
    description text NOT NULL DEFAULT '',
    status SMALLINT NOT NULL DEFAULT 0,
    created_at timestamptz DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamptz DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_created_at ON todos (user_id, created_at);
CREATE INDEX idx_user_updated_at ON todos (user_id, updated_at);
CREATE INDEX idx_user_title ON todos (user_id, title);
```

## Apply SQL Migrations

```sh
sqlx-emaple (master)> sqlx migrate run
Applied 20241116140325/migrate initial (36.58975ms)
```

The table `todos` and `users` is been created.

```sh
my_todo_list=> \dt
               List of relations
 Schema |       Name       | Type  |   Owner
--------+------------------+-------+------------
 public | _sqlx_migrations | table | db_manager
 public | todos            | table | db_manager
 public | users            | table | db_manager
(3 rows)

my_todo_list=> select *  from todos;
 id | user_id | title | description | status | created_at | updated_at
----+---------+-------+-------------+--------+------------+------------
(0 rows)

my_todo_list=> select * from users;
 id | username | email | created_at | updated_at
----+----------+-------+------------+------------
(0 rows)
```

## Executing Queries

Basic Query Execution

Writing SQL queries

Executing queries using query and query_as

Parameterized Queries

Using placeholders (? or :name)

Binding parameters in queries

Fetching Results

Retrieving rows with fetch and fetch_one

Handling optional results with fetch_optional

## Working with Structs and Mappings

Mapping Rows to Structs

Using `#[derive(sqlx::FromRow)]`

Customizing field mappings

Handling Complex Types

Working with JSON, arrays, and other complex types

Custom deserialization logic

Transactions and Batch Operations
Managing Transactions

Starting and committing transactions

Rolling back transactions on errors

Batch Operations

Executing multiple queries in a single transaction

Optimizing batch inserts and updates