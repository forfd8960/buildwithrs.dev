---
slug: implment-git-with-rust-part0
title: Implement Git with Rust Part0
authors: forfd8960
tags: [rust, blog]
---

## What is Git

We use git to manage our code, and do version control.

* generally, I use `git init` or `git clone` to start a project locally.
* And do some work in the git repo.
* And then use `git add` to add the files to the staging area.
* And then use `git commit` to commit the changes to the local repository.
* And then use `git push` to push the files to the remote repository.
* And then use `git pull origin master` to pull the changes from the remote repository.
* use `git checkout -b <new_branch>` to create and checkout to a new branch.
* use `git rebase master` to rebase changes from master to the current branch.

## Why use rust to implement git

Just want to know deeply how Git works internally by building it.

## The first step

There are some questions I want to know:

* What happends when init a new git repo.
* What happends when add a new file to the staging area.
* What happends when commit the changes to the local repository.

* How git store the files?
* How git store the history?
* How git store the branches?
* How git store the tags?
* How git store the remote repository?
* How git knows the current branch.
* How git restore the files when checkout to another branch.

I checked some blogs on how git works inernaly to understand the basic concepts.

* (Freecodecamp) [https://www.freecodecamp.org/news/git-internals-objects-branches-create-repo/]

* (GitScm) [https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain]

Some core git objects:

### Blob: `Binary Large Object`

In git, the contents of files are stored in objects called blobs, binary large objects.
Blobs, on the other hand, are just contents — binary streams of data. A blob doesn’t register its creation date, its name, or anything but its contents.

Every blob in git is identified by its `SHA-1 hash`. `SHA-1 hashes` consist of 20 bytes, usually represented by 40 characters in hexadecimal form.

A blob object:

```sh
git cat-file -p 2373d25e28b1fa10d1e9cee7b0380860b59451f4
[package]
name = "git-rs"
version = "0.1.0"
edition = "2021"

[dependencies]
```

### Commit: `Pointer to a tree of changes.`

Stored as Objects on the filesystem.

In git, a snapshot is a commit. A commit object includes a pointer to the main tree (the root directory), as well as other meta-data such as the committer, a commit message and the commit time.

Every commit holds the entire snapshot, not just diffs from the previous commit(s).

A commit object:

```sh
git cat-file -p 1321f01cf1ef8ac81b839e9f7976d740d2d27246
tree 9a8f8cfb359e7a106b96c749c69aa13a5e74a09a
author forfd8960 <forfd8960@gmail.com> 1723947713 +0800
committer forfd8960 <forfd8960@gmail.com> 1723947713 +0800

init git-rs
```

### Tree

A tree is basically a directory listing, referring to blobs as well as other trees.

Trees are identified by their SHA-1 hashes as well. Referring to these objects, either blobs or other trees, happens via the SHA-1 hash of the objects.

A tree object:

```sh
git cat-file -p 9a8f8cfb359e7a106b96c749c69aa13a5e74a09a
100644 blob ea8c4bf7f35f6f77f75d92ad8ce8349f6e81ddba	.gitignore
100644 blob 2373d25e28b1fa10d1e9cee7b0380860b59451f4	Cargo.toml
100644 blob c83c092da787cf77f810b961909987b55ccf8db9	README.md
040000 tree 305157a396c6858705a9cb625bab219053264ee4	src
```

### Branch

## Init the Project

```sh
cargo new --bin git-rs
```

### init the git commands

* `src/ccommand/mod.rs``

```rust
use clap::{Parser, Subcommand};

#[derive(Debug, Parser)]
#[command(name="simple-git", version="0.0.1", about, long_about = None)]
pub struct SimpleGit {
    #[command(subcommand)]
    pub command: GitSubCommand,
}

#[derive(Debug, Subcommand)]
pub enum GitSubCommand {
    #[command(name = "init", about = "init a git repo")]
    Init(InitOpts),
}

#[derive(Debug, Parser)]
pub struct InitOpts {
    /// Only print error and warning messages; all other output will be suppressed.
    #[arg(short, long)]
    pub quiet: bool,

    /// Specify the given object <format> (hash algorithm) for the repository. The valid values are sha1 and (if enabled) sha256. sha1 is the default.
    #[arg(long = "object-format", default_value = "sha1")]
    pub object_format: String,

    #[arg(long = "ref-format", default_value = "files")]
    pub ref_format: String,

    /// Specify the directory from which templates will be used.
    #[arg(long = "template")]
    pub template: String,

    /// Use <branch-name> for the initial branch in the newly created repository. If not specified, fall back to the default name.
    #[arg(short, long = "initial-branch")]
    pub branch: String,
}

```

* `src/main.rs`

```rust
use clap::Parser;
use git_rs::command::SimpleGit;

fn main() {
    let cmd = SimpleGit::parse();
    println!("{:?}", cmd);
}

```

## run

```sh
cargo run  -- --help
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/git-rs --help`
Usage: git-rs <COMMAND>

Commands:
  init  init a git repo
  help  Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version
```

```sh
cargo run -- help init
   Compiling git-rs v0.1.0
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.93s
     Running `target/debug/git-rs help init`
init a git repo

Usage: git-rs init [OPTIONS] --template <TEMPLATE> --initial-branch <BRANCH>

Options:
  -q, --quiet                          Only print error and warning messages; all other output will be suppressed
      --object-format <OBJECT_FORMAT>  Specify the given object <format> (hash algorithm) for the repository. The valid values are sha1 and (if enabled) sha256. sha1 is the default [default: sha1]
      --ref-format <REF_FORMAT>        [default: files]
      --template <TEMPLATE>            Specify the directory from which templates will be used
  -b, --initial-branch <BRANCH>        Use <branch-name> for the initial branch in the newly created repository. If not specified, fall back to the default name
  -h, --help                           Print help

```
