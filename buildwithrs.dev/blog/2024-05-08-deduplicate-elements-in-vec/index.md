---
slug: deduplicate-elements-in-vec
title: Deduplicate elements in Vec in Rust
authors: [forfd8960]
tags: [deduplicate, rust, vec]
---

## How to Deduplicate the elements in a Vector

### First Version

in the first version of the deduplicate:

- First create data_set to matain the data exists state.
- Iter over the data and check if the specific data in idx is exists
- If exists then skip.
- If not exists then push the data to new result: `dedup_data`.

<!-- truncate -->

```rust
fn deduplicate(data: Vec<Vec<String>>, idx: usize) -> Vec<Vec<String>> {
    let mut dedup_data: Vec<Vec<String>> = Vec::with_capacity(data.len() as usize);
    let mut data_set = HashSet::new();
    for record in data {
        let r = record.clone();
        let v = r[idx].clone();
        if data_set.contains(&v) {
            continue;
        }

        data_set.insert(v);
        dedup_data.push(record);
    }

    dedup_data
}
```

### Optimized Version

use vec `retain` method to `Retains only the elements specified by the predicate.`

the HashSet `insert` will return `false` if the data not exists in the data set. which expected(don't skip the data not exists)

```rust
fn deduplicate1(data: &mut Vec<Vec<String>>, idx: usize) {
    let mut data_set = HashSet::new();
    data.retain(|r| data_set.insert(r[idx].clone()))
}
```
