---
slug: implment-git-with-rust-part1
title: Implement Git with Rust Part1 - Add and Commit Sub-Command
authors: forfd8960
tags: [rust, blog]
---

## Add add sub-commmand

```rust
#[derive(Debug, Subcommand)]
pub enum GitSubCommand {
    #[command(name = "init", about = "init a git repo")]
    Init(InitOpts),
    #[command(name = "add", about = "add files into index")]
    Add(AddOpts),
}
```

<!-- truncate -->

```rust
#[derive(Debug, Parser)]
pub struct AddOpts {
    /// Files to add content from
    #[arg(short, long = "path")]
    pub path_spec: String,

    /// If no <pathspec> is given when -A option is used, all files in the entire working tree are updated
    #[arg(short, long = "all")]
    pub all: bool,
}
```


## Add commit Sub-Command

```rust
#[derive(Debug, Subcommand)]
pub enum GitSubCommand {
    #[command(name = "init", about = "init a git repo")]
    Init(InitOpts),
    #[command(name = "add", about = "add files into index")]
    Add(AddOpts),
    #[command(
        name = "commit",
        about = "Create a new commit containing the current contents of the index and the given log message describing the changes"
    )]
    Commit(CommitOpts),
}
```

```rust
#[derive(Debug, Parser)]
pub struct CommitOpts {
    /// Tell the command to automatically stage files that have been modified and deleted, but new files you have not told Git about are not affected.
    #[arg(short, long)]
    pub all: bool,

    /// Record changes to the repository
    #[arg(short, long)]
    pub message: String,

    /// The message taken from file with -F, command line with -m, and from commit object with -C are usually used as the commit log message unmodified. This option lets you further edit the message taken from these sources.
    #[arg(short, long = "edit")]
    pub edit: bool,

    /// Use the selected commit message without launching an editor
    #[arg(short, long = "no-edit")]
    pub no_edit: bool,

    /// Replace the tip of the current branch by creating a new commit.
    #[arg(long)]
    pub amend: bool,
}
```

## cargo run

```sh

cargo run -- help add
   Compiling git-rs v0.1.0
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.32s
     Running `target/debug/git-rs help add`
add files into index

Usage: git-rs add [OPTIONS] --path <PATH_SPEC>

Options:
  -p, --path <PATH_SPEC>  Files to add content from
  -a, --all               If no <pathspec> is given when -A option is used, all files in the entire working tree are updated
  -h, --help              Print help

cargo run -- help commit
   Compiling git-rs v0.1.0
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.98s
     Running `target/debug/git-rs help commit`
Create a new commit containing the current contents of the index and the given log message describing the changes

Usage: git-rs commit [OPTIONS] --message <MESSAGE>

Options:
  -a, --all                Tell the command to automatically stage files that have been modified and deleted, but new files you have not told Git about are not affected
  -m, --message <MESSAGE>  Record changes to the repository
  -e, --edit               The message taken from file with -F, command line with -m, and from commit object with -C are usually used as the commit log message unmodified. This option lets you further edit the message taken from these sources
  -n, --no-edit            Use the selected commit message without launching an editor
      --amend              Replace the tip of the current branch by creating a new commit
  -h, --help               Print help
```