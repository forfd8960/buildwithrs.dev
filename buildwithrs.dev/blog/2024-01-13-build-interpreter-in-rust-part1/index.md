---
slug: build-interpreter-with-rust-part1
title: Build A Simple Interpreter with Rust part1
authors: [forfd8960]
tags: [rust, interpreter, lexer]
---

The first component of this interpreter is a Lexer which used for scan and group the chars into Tokens.

Before start to build the Lexer, let's see what's finnal script we will to run:

```go
var v1 = 132
var v2 = "ABC"
var v3 = 2.56

var myArr = ["Hello, World", 256, 10.99, v1, v3]
print(myArr[0])

var myMap = hash(10)
myMap.insert("k", 1024)

var result = v1 + v3
var result1 = (v1 + v3) * (2 / 100)

var v5 = 10
if (v5 < result) {
    print("%d is less than: %d", v5, result)
}

var i = 0
for (; i <= 10; i=i+1) {
    print(i)
    if i % 2 == 0 {
        print("%s is even")
    } else {
        print("%d is odd")
    }
}

var j = 10
while j >= 0 {
    print(j)
    j = j -1
}

fn myFunc(a, b, c) {
    print(a, b, c)
    return a+b+c
}

fn fib(n) {
    if (n == 0 || n == 1) {
        return n
    }
    return fib(n-1) + fib(n-2)

}

class MyClass {

}

```

## What is Token

the token is the foundamental element that used to build the grammar of the language.

> The first step in any compiler or interpreter is scanning. The scanner takes in raw source code as a series of characters and groups it into a series of chunks we call tokens. These are the meaningful “words” and “punctuation” that make up the language’s grammar.

## So the tokens that the interpreter will Support is:

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

The Lexer is responsible for scan and parse the chars and group them into **Tokens**.

- Define a Lexer in rust

```rust
pub struct Lexer {
    chars: Vec<char>,
    start: usize,
    current: usize,
}
```
