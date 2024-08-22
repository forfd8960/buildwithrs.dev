---
slug: how-git-works
title: How Git Works
authors: forfd8960
tags: [git, blob, commit, tree, index]
---

## The internals want to know about Git

We as developers often use git to manage our code. But do you know how git works?

And as a Curious developer, I want to know how git works.

Some questions I want to know are:

* How git stores the data(the code, the image etc)
* What happens when run `git add`
* How does git knowns which file is untracked.
* Where does the data is been stored.
* How git store the commit when you run `git commit -m "message"`
* How git restore the repo when checkout to another branch.

After reading many blogs and article about git. I learned some internals of git.

## Git is a content-addressable filesystem

Git is a content-addressable filesystem. It means that it stores data in a way that allows you to retrieve it later using a hash of the data itself. This is in contrast to a traditional filesystem, which stores data in a way that allows you to retrieve it later using a path to the data.

Git uses a hash function to generate a unique identifier for each piece of data it stores. This identifier is called a "blob" in Git terminology. When you add a file to Git, it generates a blob for the file's contents and stores it in the repository. When you later want to retrieve the file, Git can use the blob's hash to find the data.

## The git directory

The git directory is where Git stores all of its data. It is located in the `.git` directory in the root of your repository. The git directory contains several subdirectories, each of which stores a different type of data.

```shell
tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

8 directories, 17 files
```

* the `HEAD` and (yet to be created) `index` files, and the `objects` and `refs` directories. These are the core parts of Git.
* The `objects` directory stores all the content for your database. 
* The `refs` directory stores pointers into commit objects in that data (branches, tags, remotes and more).
* the `HEAD` file points to the branch you currently have checked out.
* and the `index` file is where Git stores your staging area information.
