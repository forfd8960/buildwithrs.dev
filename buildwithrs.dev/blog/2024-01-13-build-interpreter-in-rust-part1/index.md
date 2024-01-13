---
slug: build-interpreter-with-rust-part1
title: Build A Simple Interpreter with Rust part1
authors: [forfd8960]
tags: [rust, interpreter, lexer]
---

So after introduction in the whole design of a simpel interpreter, you can see the first part is the a lexer, which used to parse the text into tokens.

## What is Token

the token is the foundamental element that used to build the grammar of the language.

> The first step in any compiler or interpreter is scanning. The scanner takes in raw source code as a series of characters and groups it into a series of chunks we call tokens. These are the meaningful “words” and “punctuation” that make up the language’s grammar.

## What the tokens this simple interpreter support

```rust
pub enum Token {
    Null,
    WhiteSpace,
    Ident(String),
    Integer(i64),
    Float(f64),
    SString(String),
    True,
    False,

    Var,   // keyword: var
    Print, // keyword: print
    If,
    Else,
    For,
    While,
    Return,

    Assign(char), // =
    Plus(char),   // +
    Minus(char),  // -
    Star(char),   // *
    Slash(char),  // /
    Bang,         // !

    BitOr,  // |
    Or,     // ||
    BitAnd, // &
    And,    // &&

    LParent(char), // left parenthesis (
    RParent(char), // right parenthesis )
    LBrace(char),  // left brace {
    RBrace(char),  // right brace }
    // left square brackets [
    LSBracket(char),
    // right square brackets ]
    RSBracket(char),

    Lt(String),    // less than <
    LtEQ(String),  // <=
    Gt(String),    // > greater than
    GtEQ(String),  // >=
    EQ(String),    // ==
    NotEQ(String), // !=
    EOF,
    Unkown,
}
```

## What is Lexer
