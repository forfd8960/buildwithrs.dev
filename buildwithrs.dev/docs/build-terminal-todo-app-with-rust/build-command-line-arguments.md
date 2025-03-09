---
sidebar_position: 2
---

# Build Command line Arguments

Since our todo app will support the following actions:

* Create Todo
* Move Todo to a new state.
* List Todos
* Delete Todo

So we will build 4 sub commands to support the above actions under `command/options.rs`:

## Create Options

```rust
use clap::Args;

#[derive(Debug, Args)]
pub struct CreateOpts {
    #[arg(short = 'n', long)]
    pub name: String,
    #[arg(short = 'd', long)]
    pub desc: String,
    #[arg(short = 'f', long)]
    pub finish_date: String,
}
```

## Move Options

```rust
#[derive(Debug, Args)]
pub struct MoveOpts {
    #[arg(short, long)]
    pub id: u64,
    #[arg(short, long)]
    pub status: String,
}
```

## List Options

```rust
#[derive(Debug, Args)]
pub struct ListOpts {
    #[arg(long)]
    pub sort: Option<String>,
    #[arg(long)]
    pub status: Option<String>,
    #[arg(short, long)]
    pub finsh_date: Option<String>,
}
```

## Delete Options

```rust
#[derive(Debug, Args)]
pub struct DeleteOpts {
    #[arg(short, long)]
    pub id: u64,
}
```

## Group the Sub Command into the Command `command/mod.rs`

```rust
use clap::Parser;
use options::{CreateOpts, DeleteOpts, ListOpts, MoveOpts};

pub mod options;

#[derive(Debug, Parser)]
#[command(name="task-planner", version, author, about="What Todo Next?", long_about = None)]
pub struct Command {
    #[command(subcommand)]
    pub cmd: SubCommand,
}

#[derive(Debug, Parser)]
pub enum SubCommand {
    #[command(name = "create", about = "create a todo list")]
    Create(CreateOpts),
    #[command(name = "delete", about = "delete a todo list")]
    Delete(DeleteOpts),
    #[command(name = "list", about = "list todos by filters")]
    List(ListOpts),
    #[command(name = "move", about = "move a todo to another status")]
    Move(MoveOpts),
}

```
