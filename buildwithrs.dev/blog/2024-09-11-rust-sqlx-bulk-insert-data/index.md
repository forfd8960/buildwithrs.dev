---
slug: rust-how-to-bulk-insert-data-with-sqlx
title: Rust How to Bulk Insert Data with sqlx
authors: forfd8960
tags: [rust, sqlx, bulk_insert]
---

## Create the table in Postgres

```sql
CREATE TYPE gender AS ENUM('female', 'male', 'unknown');

CREATE TABLE user_info(
    email varchar(128) NOT NULL PRIMARY KEY,
    name varchar(64) NOT NULL,
    gender gender DEFAULT 'unknown',
);
```

## Main

```rust
use anyhow::Result;
use fake::{Fake, Faker};
use nanoid::nanoid;
use sqlx::{Executor, PgPool};

struct UserInfo {
    email: String,
    name: String,
    gender: Gender,
}

impl Default for UserInfo {
    fn default() -> Self {
        UserInfo {
            email: nanoid!(10),
            name: Faker.fake::<String>(),
            gender: Faker.fake::<Gender>(),
        }
    }
}

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let user_stats = Faker.fake::<UserStats>();
    println!("{:?}", user_stats);

    let pool = PgPool::connect("postgres://postgers@localhost:5432/user_info").await?;
    let users = (0..100).map(|_| UserInfo::default()).collect();

    bulk_insert(users, &pool).await?;

    Ok(())
}
```

## Bulk Insert

```rust
async fn bulk_insert(users: Vec<UserInfo>, pool: &sqlx::PgPool) -> Result<()> {
    let mut tx = pool.begin().await?;
    for user in users {
        let query = sqlx::query(
            r#"INSERT INTO user_stats (email, name) VALUES ($1, $2)"#
        )
        .bind(&user.email)
        .bind(&user.name);

        tx.execute(query).await?;
    }

    tx.commit().await?;
    Ok(())
}
```
