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

<!-- truncate -->

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

## Set Models

### Set the data model(`struct`) for each table

`sqlx-example/src/models.rs`

When set model for a Row, use `FromRow` macro to do map the Row to the Struct:

**mapping Rows to Structs, Using `#[derive(sqlx::FromRow)]`**

```rust
use chrono::{DateTime, Utc};
use serde::{Deserialize, Serialize};
use sqlx::FromRow;

#[derive(Debug, FromRow, Serialize, Deserialize)]
pub struct Todo {
    pub id: i64,
    pub user_id: i64,
    pub title: String,
    pub description: String,
    pub status: i16,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Clone, FromRow, Serialize, Deserialize, PartialEq)]
pub struct User {
    pub id: i64,
    pub username: String,
    pub email: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}
```

## Executing Queries

### Create User

`sqlx-example/src/user.rs`

* Use `sqlx::query_as` to set the SQL that to create the User.

> Execute a single SQL query as a prepared statement (transparently cached).Maps rows to Rust types using [`FromRow`].

* Use `bind` to set the values in the SQL.
* Use `fetch_one` to get one row from the DB.

> Execute the query, returning the first row or [`Error::RowNotFound`] otherwise.


```rust
use serde::Deserialize;
use sqlx::PgPool;

use crate::{errors::AppError, models::User};

#[derive(Debug, Deserialize)]
pub struct CreateUser {
    pub username: String,
    pub email: String,
}

pub async fn create_user(input: &CreateUser, pool: &PgPool) -> Result<User, AppError> {
    let user = sqlx::query_as("INSERT INTO users(username, email) VALUES($1, $2) RETURNING id,username,email,created_at,updated_at")
        .bind(&input.username)
        .bind(&input.email)
        .fetch_one(pool)
        .await?;
    Ok(user)
}
```

### Get Uer by email and id

* Use `fetch_optional` to get one `Option<Row>`.

> Execute the query, returning the first row or `None` otherwise.


```rust
pub async fn get_user_by_email(email: &str, pool: &PgPool) -> Result<Option<User>, AppError> {
    let user = sqlx::query_as(
        "SELECT id,username,email,created_at,updated_at FROM users WHERE email=$1",
    )
    .bind(email)
    .fetch_optional(pool)
    .await?;
    Ok(user)
}

pub async fn get_user_by_id(id: i64, pool: &PgPool) -> Result<Option<User>, AppError> {
    let user =
        sqlx::query_as("SELECT id,username,email,created_at,updated_at FROM users WHERE id=$1")
            .bind(id)
            .fetch_optional(pool)
            .await?;
    Ok(user)
}
```

### Create Todo

In `sqlx-example/src/todolist.rs`

```rust
use serde::Deserialize;
use sqlx::PgPool;

use crate::{errors::AppError, models::Todo};

#[derive(Debug, Deserialize)]
pub struct CreateTodo {
    pub user_id: i64,
    pub title: String,
    pub description: String,
    pub status: i16,
}

pub async fn create_todo(input: &CreateTodo, pool: &PgPool) -> Result<Todo, AppError> {
    let todo = sqlx::query_as(
        r#"INSERT INTO todos (user_id,title,description,status)
                VALUES ($1,$2,$3,$4)
                RETURNING id,user_id,title,description,status,created_at,updated_at;
                "#,
    )
    .bind(input.user_id)
    .bind(&input.title)
    .bind(&input.description)
    .bind(input.status)
    .fetch_one(pool)
    .await?;

    Ok(todo)
}
```

### List Todos

* Use `fetch_all` to get all the rows.

```rust
#[derive(Debug)]
pub struct GetTodosArgs {
    pub user_id: i64,
    pub offset: i64,
    pub limit: i64,
    pub status: Option<i16>,
}

pub async fn get_todos_by_user_id(
    args: GetTodosArgs,
    pool: &PgPool,
) -> Result<Vec<Todo>, AppError> {
    let mut offset = args.offset;
    if args.offset < 0 {
        offset = 0;
    }

    let mut limit = args.limit;
    if args.limit <= 0 || args.limit > 100 {
        limit = 100
    }

    let mut status = 0 as i16;
    if args.status.is_some() {
        status = args.status.unwrap();
    }

    let todos = sqlx::query_as(
        r#"
        SELECT id,user_id,title,description,status,created_at,updated_at
        FROM todos
        WHERE user_id=$1 AND status=$2
        ORDER BY id DESC
        LIMIT $3
        OFFSET $4;
    "#,
    )
    .bind(args.user_id)
    .bind(status)
    .bind(limit)
    .bind(offset)
    .fetch_all(pool)
    .await?;

    Ok(todos)
}
```

### Update Todo Status

```rust
pub async fn update_todo_status(id: i64, status: i16, pool: &PgPool) -> Result<Todo, AppError> {
    let todo = sqlx::query_as(
        r#"
        UPDATE todos SET status=$1 WHERE id=$2
        RETURNING id,user_id,title,description,status,created_at,updated_at;
    "#,
    )
    .bind(status)
    .bind(id)
    .fetch_one(pool)
    .await?;

    Ok(todo)
}
```

## Qeury data with models in Main

`sqlx-example/src/main.rs`

```rust
use sqlx::PgPool;
use sqlx_emaple::{
    errors::AppError,
    models::{Todo, User},
    todolist::{self, CreateTodo, GetTodosArgs},
    user::{self, CreateUser},
};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    println!("Hello, world!");

    let pool = PgPool::connect("postgres://db_manager:super_admin8801@localhost:5432/my_todo_list")
        .await?;
    println!("connected database: {:?}", pool);

    let user_id = match create_user(&pool).await {
        Ok(u) => {
            println!("created user: {:?}", u);
            u.id
        }
        Err(e) => {
            println!("failed to create user: {}", e);
            let u = user::get_user_by_id(1, &pool).await?.unwrap();
            u.id
        }
    };

    let todo = create_todo(&pool, user_id).await?;
    println!("created todo: {:?}", todo);

    let todos = get_todo_by_user_id(&pool, user_id).await?;
    println!("get_todo_by_user_id: {:?}", todos);

    Ok(())
}

async fn create_user(pool: &PgPool) -> Result<User, AppError> {
    let user = CreateUser {
        username: "Alice".to_string(),
        email: "alice@acme.org".to_string(),
    };

    user::create_user(&user, pool).await
}

async fn create_todo(pool: &PgPool, user_id: i64) -> Result<Todo, AppError> {
    let create_todo = CreateTodo {
        user_id,
        title: "Query Data with SQLX".to_string(),
        description: "sqlx usage".to_string(),
        status: 0,
    };

    todolist::create_todo(&create_todo, pool).await
}

async fn get_todo_by_user_id(pool: &PgPool, user_id: i64) -> Result<Vec<Todo>, AppError> {
    todolist::get_todos_by_user_id(
        GetTodosArgs {
            user_id,
            offset: 0,
            limit: 100,
            status: None,
        },
        pool,
    )
    .await
}
```

The data has been inserted in Postgres

```sh
my_todo_list=> select * from users;
 id | username |     email      |          created_at           |          updated_at
----+----------+----------------+-------------------------------+-------------------------------
  1 | Alice    | alice@acme.org | 2024-11-17 12:13:40.819274+08 | 2024-11-17 12:13:40.819274+08
(1 row)


my_todo_list=> select * from todos;
 id | user_id |        title         | description | status |          created_at           |          updated_at
----+---------+----------------------+-------------+--------+-------------------------------+-------------------------------
  1 |       1 | Query Data with SQLX | sqlx usage  |      0 | 2024-11-17 12:24:10.611699+08 | 2024-11-17 12:24:10.611699+08
  2 |       1 | Query Data with SQLX | sqlx usage  |      0 | 2024-11-17 12:27:07.354045+08 | 2024-11-17 12:27:07.354045+08
(2 rows)
```

```sh
sqlx-emaple (master)> cargo run .
   Compiling sqlx-emaple v0.1.0 (~/sqlx-emaple)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.75s
     Running `target/debug/sqlx-emaple .`
Hello, world!
connected database: Pool { size: 1, num_idle: 1, is_closed: false, options: PoolOptions { max_connections: 10, min_connections: 0, connect_timeout: 30s, max_lifetime: Some(1800s), idle_timeout: Some(600s), test_before_acquire: true } }

failed to create user: user email already been used: alice@acme.org

created todo: Todo { id: 2, user_id: 1, title: "Query Data with SQLX", description: "sqlx usage", status: 0, created_at: 2024-11-17T04:27:07.354045Z, updated_at: 2024-11-17T04:27:07.354045Z }

get_todo_by_user_id: [Todo { id: 2, user_id: 1, title: "Query Data with SQLX", description: "sqlx usage", status: 0, created_at: 2024-11-17T04:27:07.354045Z, updated_at: 2024-11-17T04:27:07.354045Z }, Todo { id: 1, user_id: 1, title: "Query Data with SQLX", description: "sqlx usage", status: 0, created_at: 2024-11-17T04:24:10.611699Z, updated_at: 2024-11-17T04:24:10.611699Z }]
```

## Managing Transactions

* Starting and committing transactions

`let mut tx = pool.begin().await?;`

### Set Model: `Account`

```rust
#[derive(Debug, Clone, FromRow, PartialEq)]
pub struct Account {
    pub account_id: i64,
    pub user_id: i64,
    pub balance: BigDecimal,
    pub currency: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub is_active: bool,
}
```

### Create Account

`sqlx-example/src/account.rs`

```rust
use serde::Deserialize;
use sqlx::{PgPool, Postgres, Transaction};

use crate::{errors::AppError, models::Account};

#[derive(Debug, Deserialize)]
pub struct CreateAccount {
    pub user_id: i64,
    pub balance: f64,
    pub currency: String,
    pub is_active: bool,
}

pub async fn create_account(input: &CreateAccount, pool: &PgPool) -> Result<Account, AppError> {
    let account = sqlx::query_as("INSERT INTO account(user_id, balance, currency, is_active) VALUES($1, $2, $3, $4) RETURNING account_id,user_id, balance, currency,created_at,updated_at,is_active")
        .bind(&input.user_id)
        .bind(&input.balance)
        .bind(&input.currency)
        .bind(&input.is_active)
        .fetch_one(pool)
        .await?;
    Ok(account)
}
```

### Transfer `amount` to another Account

`sqlx-example/src/main.rs`

```rust
use sqlx::PgPool;
use sqlx_emaple::{
    account::{self, CreateAccount},
    errors::AppError,
    models::{Account, Todo, User},
    todolist::{self, CreateTodo, GetTodosArgs},
    user::{self, CreateUser},
};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    println!("Hello, world!");

    let pool = PgPool::connect("postgres://db_manager:super_admin8801@localhost:5432/my_todo_list")
        .await?;
    println!("connected database: {:?}", pool);
    println!();

    // let user1 = create_user(&pool, "Alex".to_string()).await?;
    // let user2 = create_user(&pool, "Bob".to_string()).await?;

    let account1 = create_account(&pool, 2 as i64, 1000.0 as f64).await?;
    let account2 = create_account(&pool, 3 as i64, 1000.0 as f64).await?;

    println!(
        "transfer from account1: {:?}, to account2: {:?}",
        account1, account2
    );
    transfer_from_to(
        &pool,
        account1.account_id,
        account2.account_id,
        100.0 as f64,
    )
    .await?;

    Ok(())
}
```

### transfer_from_to

`sqlx-example/src/main.rs`

```rust
async fn transfer_from_to(
    pool: &PgPool,
    from_user: i64,
    to_user: i64,
    amount: f64,
) -> Result<(), AppError> {
    println!(
        "start transaction transafer from: {} to: {}, with amount: {}",
        from_user, to_user, amount
    );

    let mut tx = pool.begin().await?;

    let from_account = account::withdrawal(from_user, amount, &mut tx).await?;
    println!("withdrawal {} from acount: {:?}", amount, from_account);

    let to_account = account::deposit(to_user, amount, &mut tx).await?;
    println!("deposit {} from acount: {:?}", amount, to_account);

    match tx.commit().await {
        Ok(()) => {
            println!("success transfer {} to {}", amount, to_account.account_id);
        }
        Err(e) => {
            println!(
                "transfer {} to {} failed: {:?}",
                amount, to_account.account_id, e
            );
        }
    }

    Ok(())
}
```

### Withdrawal

`sqlx-example/src/account.rs`

```rust
pub async fn withdrawal(
    account_id: i64,
    amount: f64,
    tx: &mut Transaction<'static, Postgres>,
) -> Result<Account, AppError> {
    // 1 lock thw account for update

    let _ = sqlx::query("SELECT * FROM account WHERE account_id=$1 FOR UPDATE")
        .bind(account_id)
        .fetch_one(&mut **tx)
        .await?;

    // 2 update account balance
    let now: DateTime<Utc> = Utc::now();
    let account = sqlx::query_as(
        "UPDATE account SET balance=balance - $1, updated_at=$2 
            WHERE account_id=$3
            RETURNING account_id,user_id, balance, currency,created_at,updated_at,is_active",
    )
    .bind(amount)
    .bind(now)
    .bind(account_id)
    .fetch_one(&mut **tx)
    .await?;
    Ok(account)
}
```

### Deposit

`sqlx-example/src/account.rs`

```rust
pub async fn deposit<'a>(
    account_id: i64,
    amount: f64,
    tx: &mut Transaction<'static, Postgres>,
) -> Result<Account, AppError> {
    // 1 lock thw account for update

    let _ = sqlx::query("SELECT * FROM account WHERE account_id=$1 FOR UPDATE")
        .bind(account_id)
        .execute(&mut **tx)
        .await?;

    // 2 update account balance
    let now: DateTime<Utc> = Utc::now();
    let account = sqlx::query_as(
        "UPDATE account SET balance=balance + $1, updated_at=$2 
            WHERE account_id=$3
            RETURNING account_id,user_id, balance, currency,created_at,updated_at,is_active",
    )
    .bind(amount)
    .bind(now)
    .bind(account_id)
    .fetch_one(&mut **tx)
    .await?;
    Ok(account)
}

```

### Run Transaction

```sh

sqlx-emaple (master)> cargo run .

warning: `sqlx-emaple` (bin "sqlx-emaple") generated 3 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/sqlx-emaple .`

Hello, world!
connected database: Pool { size: 1, num_idle: 1, is_closed: false, options: PoolOptions { max_connections: 10, min_connections: 0, connect_timeout: 30s, max_lifetime: Some(1800s), idle_timeout: Some(600s), test_before_acquire: true } }

transfer from account1: Account { account_id: 4, user_id: 2, balance: BigDecimal(sign=Plus, scale=0, digits=[1000]), currency: "USD", created_at: 2024-11-17T09:13:07.588104Z, updated_at: 2024-11-17T09:13:07.588104Z, is_active: true }, to account2: Account { account_id: 5, user_id: 3, balance: BigDecimal(sign=Plus, scale=0, digits=[1000]), currency: "USD", created_at: 2024-11-17T09:13:07.617288Z, updated_at: 2024-11-17T09:13:07.617288Z, is_active: true }
start transaction transafer from: 4 to: 5, with amount: 100
withdrawal 100 from acount: Account { account_id: 4, user_id: 2, balance: BigDecimal(sign=Plus, scale=0, digits=[900]), currency: "USD", created_at: 2024-11-17T09:13:07.588104Z, updated_at: 2024-11-17T09:13:07.622722Z, is_active: true }
deposit 100 from acount: Account { account_id: 5, user_id: 3, balance: BigDecimal(sign=Plus, scale=0, digits=[1100]), currency: "USD", created_at: 2024-11-17T09:13:07.617288Z, updated_at: 2024-11-17T09:13:07.629311Z, is_active: true }
success transfer 100 to 5

```

```sh
my_todo_list=> select * from account;
 account_id | user_id | balance | currency |          created_at           |          updated_at           | is_active
------------+---------+---------+----------+-------------------------------+-------------------------------+-----------
          4 |       2 |  900.00 | USD      | 2024-11-17 17:13:07.588104+08 | 2024-11-17 17:13:07.622722+08 | t
          5 |       3 | 1100.00 | USD      | 2024-11-17 17:13:07.617288+08 | 2024-11-17 17:13:07.629311+08 | t
(5 rows)
```

## Complex Queries

### Batch Insert Rows

#### bulk_create_todos

> **sqlx-example/src/models/todolist.rs**

```rust
pub async fn bulk_create_todos(pool: &PgPool, todos: &[CreateTodo]) -> Result<(), sqlx::Error> {
    let user_ids: Vec<i64> = todos.iter().map(|s| s.user_id).collect();
    let titles: Vec<String> = todos.iter().map(|s| s.title.clone()).collect();
    let descs: Vec<String> = todos.iter().map(|s| s.description.clone()).collect();
    let status: Vec<i16> = todos.iter().map(|s| s.status).collect();

    sqlx::query!(
        r#"INSERT INTO todos (user_id,title,description,status)
         SELECT * FROM UNNEST($1::bigint[], $2::text[], $3::text[], $4::smallint[])"#,
        &user_ids[..],
        &titles[..],
        &descs[..],
        &status[..],
    )
    .execute(pool)
    .await?;

    Ok(())
}
```

#### bulk insert main

`sqlx-example/examples/bulk_insert.rs`

```rust
use sqlx::PgPool;
use sqlx_example::todolist::{self, CreateTodo};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    println!("Bulk Insert example");

    let pool = PgPool::connect("postgres://db_manager:xxx@localhost:5432/my_todo_list").await?;

    println!("connected database: {:?}", pool);
    println!();

    let todos = vec![
        CreateTodo {
            user_id: 1,
            title: "learn axum".to_string(),
            description: "axum".to_string(),
            status: 0,
        },
        CreateTodo {
            user_id: 2,
            title: "learn sqlx".to_string(),
            description: "sqlx".to_string(),
            status: 0,
        },
        CreateTodo {
            user_id: 1,
            title: "learn react".to_string(),
            description: "react".to_string(),
            status: 0,
        },
    ];

    todolist::bulk_create_todos(&pool, &todos).await?;
    println!("bulk create todos success!");
    Ok(())
}

```

```sh
my_todo_list=> select * from todos order by created_at desc limit 3;
 id | user_id |    title    | description | status |          created_at           |          updated_at
----+---------+-------------+-------------+--------+-------------------------------+-------------------------------
  3 |       1 | learn axum  | axum        |      0 | 2024-11-18 08:44:24.291281+08 | 2024-11-18 08:44:24.291281+08
  4 |       2 | learn sqlx  | sqlx        |      0 | 2024-11-18 08:44:24.291281+08 | 2024-11-18 08:44:24.291281+08
  5 |       1 | learn react | react       |      0 | 2024-11-18 08:44:24.291281+08 | 2024-11-18 08:44:24.291281+08
(3 rows)
```
