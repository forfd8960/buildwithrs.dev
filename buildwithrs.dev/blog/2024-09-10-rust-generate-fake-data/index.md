---
slug: rust-generate-fake-data
title: Rust Generate Fake Data
authors: forfd8960
tags: [rust, fake]
---

## Fake crate

use [Fake](https://docs.rs/fake/latest/fake/trait.Dummy.html) crate to generate fake data.

```toml
[dependencies]
rng = "0.1.0"
rand = "0.8.5"
nanoid = "0.4.0"
fake = { version="2.9.2", features=["derive", "chrono"]}
```

## Apply Dummy trait on struct

* Assume need to generate some UserStats fake data.
* Apply `Dummy` on `UserStats` struct

```rust
use chrono::{DateTime, Days, Utc};
use fake::faker::chrono::en::DateTimeBetween;
use fake::faker::internet::en::SafeEmail;
use fake::faker::name::en::Name;
use fake::{Dummy, Fake, Faker};

#[derive(Debug, Clone, Dummy, Deserialize, Serialize, PartialEq, Eq)]
enum Gender {
    Male,
    Female,
    Unknown,
}

#[derive(Debug, Clone, Dummy, Deserialize, Serialize, PartialEq, Eq)]
struct UserStats {
    #[dummy(faker = "UniqueEmail")]
    email: String,
    #[dummy(faker = "Name()")]
    name: String,
    gender: Gender,
    #[dummy(faker = "DateTimeBetween(
        start(365*5),
        end()
    )")]
    created_at: DateTime<Utc>,
    #[dummy(faker = "DateTimeBetween(
        start(365*3),
        end()
    )")]
    last_visited_at: DateTime<Utc>,
    #[dummy(faker = "DateTimeBetween(
        start(30),
        end()
    )")]
    last_watched_at: DateTime<Utc>,
    #[dummy(faker = "IntList(10, 1, 100)")]
    recent_watched: Vec<i32>,
    #[dummy(faker = "IntList(10, 1, 100)")]
    viewed_but_not_started: Vec<i32>,
    #[dummy(faker = "IntList(10, 1, 100)")]
    started_but_not_finished: Vec<i32>,
    #[dummy(faker = "IntList(10, 1, 100)")]
    finished: Vec<i32>,
    #[dummy(faker = "DateTimeBetween(
        start(30),
        end()
    )")]
    last_email_notification: DateTime<Utc>,
    #[dummy(faker = "DateTimeBetween(
        start(20),
        end()
    )")]
    last_in_app_notification: DateTime<Utc>,
    #[dummy(faker = "DateTimeBetween(
        start(10),
        end()
    )")]
    last_sms_notification: DateTime<Utc>,
}
```

* Use `UniqueEmail` to generate a unique email.

```rust
use rand::Rng;
use fake::Dummy;
use nanoid::nanoid;


struct UniqueEmail;

impl Dummy<UniqueEmail> for String {
    fn dummy_with_rng<R: Rng + ?Sized>(_: &UniqueEmail, rng: &mut R) -> String {
        let alphabet: [char; 16] = [
            '1', '2', '3', '4', '5', '6', '7', '8', '9', '0', 'a', 'b', 'c', 'd', 'e', 'f',
        ];

        let id = nanoid!(10, &alphabet);
        let email: String = SafeEmail().fake_with_rng(rng);
        let at = email.find('@').unwrap();
        format!("{}.{}@{}", &email[..at], id, &email[at..])
    }
}

```

* Use `DateTimeBetween` to generate a datetime.
* Use `IntList` to generate  a vec of int

```rust
use rand::Rng;
use fake::Dummy;

struct IntList(pub i32, pub i32, pub i32);

impl Dummy<IntList> for Vec<i32> {
    fn dummy_with_rng<R: Rng + ?Sized>(v: &IntList, rng: &mut R) -> Vec<i32> {
        let (max, start, len) = (v.0, v.1, v.2);
        let size = rng.gen_range(0..max);
        (0..size)
            .map(|_| rng.gen_range(start..start + len))
            .collect()
    }
}
```

## Run 

```rust
fn main() {
    let user_stats = Faker.fake::<UserStats>();
    println!("{:?}", user_stats);
}
```

```sh
> cargo run --example gen

UserStats { email: "amiya.fa798e1912@@example.net", name: "Guido Marks", gender: Unknown, created_at: 2022-08-28T14:16:17.578570Z, last_visited_at: 2021-10-22T16:33:17.578585Z, last_watched_at: 2024-08-15T03:30:17.578586Z, recent_watched: [92, 37, 65], viewed_but_not_started: [72, 44, 65, 28, 18, 7, 58, 63, 16], started_but_not_finished: [19], finished: [74, 91, 54, 60, 50, 68, 1], last_email_notification: 2024-08-31T15:57:17.578598Z, last_in_app_notification: 2024-08-22T10:31:17.578599Z, last_sms_notification: 2024-09-08T19:50:17.578600Z }
```