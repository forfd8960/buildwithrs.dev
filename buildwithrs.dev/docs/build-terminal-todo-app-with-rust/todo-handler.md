---
sidebar_position: 4
---

# The Todo Handler

`src/handler/mod.rs`

```rust
use crate::{
    command::options::{CreateOpts, DeleteOpts, ListOpts, MoveOpts},
    model::{Status, Todo, TodoStore},
};

pub mod create;
pub mod delete;
pub mod list;
pub mod update;
```

## Create Todo

```rust
pub fn create_todo(todo_store: &mut TodoStore, opts: CreateOpts) -> Result<Todo, String> {
    println!("create todo: {:?}", opts);

    let new_todo = todo_store.create(&Todo {
        id: 0,
        name: opts.name,
        desc: opts.desc,
        status: Status::PLAN,
        finish_date: opts.finish_date,
    })?;

    todo_store.save()?;
    Ok(new_todo)
}
```

## Delete Todo

```rust
pub fn delete_todo(todo_store: &mut TodoStore, opts: DeleteOpts) -> Result<(), String> {
    println!("delete todo: {:?}", opts);
    todo_store.delete(opts.id)?;

    todo_store.save()
}
```

## List todo

```rust
pub fn list_todos(store: &mut TodoStore, opts: ListOpts) -> Result<Vec<Todo>, String> {
    println!("list todos: {:?}", opts);
    store.list()
}

```

## Update Todo

```rust
pub fn update_todo(todo_store: &mut TodoStore, opts: MoveOpts) -> Result<(), String> {
    println!("update todo options: {:?}", opts);

    match opts.status.as_str().parse() {
        Ok(status) => {
            todo_store.update_status(opts.id, status)?;
            todo_store.save()
        }
        Err(e) => Err(e),
    }
}
```
