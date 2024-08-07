---
slug: rust-json-parser
title: Rust Json Parser 
authors: forfd8960
tags: [rust, blog]
---

## What is Json

[What is Json](https://www.json.org/json-en.html)

> JSON (JavaScript Object Notation) is a lightweight data-interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate. It is based on a subset of the JavaScript Programming Language Standard ECMA-262 3rd Edition - December 1999.

## What's the element in a Json

A json looks like:

```json
{
    "name": "Alex",
    "age": 30,
    "hobbies": ["coding", "reading", "music"],
    "isStudent": false,
    "address": {
        "street": "123 Main St",
        "city": "New York",
        "zip": "10001",
        "country": "USA",
        "phone": "123-456-7890"
    }
}
```

In the Json, there will be 6 types of elements.

- Object
- Array
- String
- Number
- Boolean
- Null

## Scan json into Tokens

first use scanner to scan the json text and group the chars into different Tokens.

![Scanner](Json_Scanner.png)

### The Token

```rust
#[derive(Debug, Clone, PartialEq)]
pub enum Token {
    Number(f64),
    String(String),
    Boolean(bool),
    ArrayStart,  // [
    ArrayEnd,    // ]
    ObjectStart, // {
    ObjectEnd,   // }
    Null,
    NewLine, // \r \n \t 
    Comma, //,
    Colon, // :
}
```

### The Scanner

```rust
pub struct Scanner {
    pub chars: Vec<char>, // the chars in the json text
    pub start: usize, // the start 
    pub current: usize, // current index
}
```

### Scan Tokens

```rust
fn scan_token(&mut self) -> Result<Token, LexerError> {
        let cur = self.advance();

        // let mut tokens: Vec<Token> = vec![];
        match cur {
            '{' => Ok(Token::ObjectStart),
            '}' => Ok(Token::ObjectEnd),
            '[' => Ok(Token::ArrayStart),
            ']' => Ok(Token::ArrayEnd),
            ':' => Ok(Token::Colon),
            ',' => Ok(Token::Comma),
            '"' => Ok(self.scan_string()?),
            '\n' | '\t' | '\r' | ' ' => Ok(Token::NewLine),
            _ => {
                if cur.is_numeric() {
                    return Ok(self.scan_number()?);
                } else if cur.is_alphabetic() {
                    return Ok(self.scan_identifier()?);
                } else {
                    return Err(LexerError::InvalidChar);
                }
            }
        }
    }
```

### Scan String

```rust
fn scan_string(&mut self) -> Result<Token, LexerError> {
        let mut s = String::new();

        while let Some(c) = self.peek() {
            if c == '"' || self.is_at_end() {
                break;
            }

            self.advance();
        }

        if self.is_at_end() {
            return Err(LexerError::InvalidString("unterminated string".to_string()));
        }

        self.advance();
        for c in self.chars[self.start + 1..self.current - 1].iter() {
            s.push(*c);
        }
        Ok(Token::String(s))
    }
```

### Scan number

```rust
fn scan_number(&mut self) -> Result<Token, LexerError> {
        self.peek_number();

        let s: String = self.chars[self.start..self.current].iter().collect();
        match s.parse::<f64>() {
            Ok(n) => Ok(Token::Number(n)),
            Err(_) => Err(LexerError::InvalidNumber(s)),
        }
    }

    fn peek_number(&mut self) {
        while let Some(c) = self.peek() {
            if !c.is_numeric() {
                break;
            }

            self.advance();
        }

        let cur = self.peek();
        if cur.is_none() {
            return;
        }

        if cur.unwrap() != '.' {
            return;
        }

        if let Some(c_n) = self.peek_next() {
            if !c_n.is_numeric() {
                return;
            }

            self.advance();

            while let Some(cc) = self.peek() {
                if !cc.is_numeric() {
                    break;
                }
                self.advance();
            }
        }
    }
```

### Scan Identifier

```rust
fn scan_identifier(&mut self) -> Result<Token, LexerError> {
        while let Some(c) = self.peek() {
            if !c.is_alphanumeric() {
                break;
            }
            self.advance();
        }

        let text: String = self.chars[self.start..self.current].iter().collect();
        match text.as_str() {
            "null" => Ok(Token::Null),
            "true" => Ok(Token::Boolean(true)),
            "false" => Ok(Token::Boolean(false)),
            _ => Err(LexerError::InvalidIdent(text)),
        }
    }
```



## Parse Tokens into Json
