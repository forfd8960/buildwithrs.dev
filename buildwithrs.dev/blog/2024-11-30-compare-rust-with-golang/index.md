---
slug: compare-rust-with-golang-programming
title: Compare Rust and Golang Programming
authors: forfd8960
tags: [rust, golang]
---

In this blog, I will share the programming difference between Rust and Golang, Include:

* How they use different syntax to declare data type.
* How rust and golang define the common behavior(interface vs trait).
* How rust and golang struct to implement the common behavior.
* What's the difference between rust and golang  about **Error Handling**.
* How they define complex data type(struct).
* How they define `enum`.
* How they Obtaining and using third-party libraries.
* What's the difference between rust and golang when write unit test.
* What's the difference  between rust and golang  about async programming.

<!-- truncate -->

## The Entry Point difference

```go
package main
func main() {
    fmt.Println("hello, world!")
}
```

```rust
fn main() {
    println!("Hello, World");
}
```

## Primitive Data Type

* Rust

```rust
// Signed integers
let a: i8 = 127;    // 8-bit
let b: i16 = 32767; // 16-bit
let c: i32 = 0;     // 32-bit (default integer type)
let d: i64 = 0;     // 64-bit
let e: i128 = 0;    // 128-bit
let f: isize = 0;   // pointer-sized

// Unsigned integers
let g: u8 = 255;    // 8-bit
let h: u16 = 65535; // 16-bit
let i: u32 = 0;     // 32-bit
let j: u64 = 0;     // 64-bit
let k: u128 = 0;    // 128-bit
let l: usize = 0;   // pointer-sized
```

* Golang

```go
// Signed integers
var a int8 = 127    // 8-bit
var b int16 = 32767 // 16-bit
var c int32 = 0     // 32-bit
var d int64 = 0     // 64-bit
var e int = 0       // platform dependent (32 or 64 bit)

// Unsigned integers
var f uint8 = 255   // 8-bit
var g uint16 = 0    // 16-bit
var h uint32 = 0    // 32-bit
var i uint64 = 0    // 64-bit
var j uint = 0      // platform dependent
```

## Function

* Rust use `fn` to declare an function.
* Golang use `func` to start a function.

* Rust use `pub fn` to declare an function can be access from module.
* Golang use `func UpperCaseName` to allow public access.

```rust
pub fn is_even(num: i64) -> bool {
    num % 2 == 0
}
```

```go
func isEven(num int64) bool {
    return num % 2 == 0
}
```

* Allow Public access for function

```rust
pub fn allow_access() -> String {
    "Allow".to_string()
}
```

```go
func AllowAccess() string {
    return "Allow"
}
```

* Receive multiple argsuments

```rust
pub fn merge(elements: Vec<String>) -> String {
    elements.join(",")
}
```

```go
import "strings"

func Merge(elements ...string) string {
    return strings.Join(elements, ",")
}
```

## How to use `struct`

```rust
#[derive(Debug, Clone)]
pub struct Person {
    pub name: String,
    pub age: u8,
    pub location: Location,
}

#[derive(Debug, Clone)]
pub struct Location {
    pub country: String,
    pub city: String,
}
```

```go
package structure

type Person struct {
    Name     string
    Age      uint8
    Location *Location
}

type Location struct {
    Country string
    City    string
}
```

### Implment method on struct

```rust
#[derive(Debug, Clone, PartialEq)]
pub struct Location {
    pub country: String,
    pub city: String,
}

impl Person {
    pub fn new(name: String, age: u8, loc: Location) -> Self {
        Self {
            name,
            age,
            location: loc,
        }
    }

    pub fn come_from_same_place(&self, other: Person) -> bool {
        self.location.eq(&other.location)
    }
}
```

* Golang `interface`

```go
func NewPerson(name string, age uint8, loc Location) *Person {
    return &Person{
        Name:     name,
        Age:      age,
        Location: &loc,
    }
}

func (p *Person) isSameLocation(other *Person) bool {
    return p.Location.Country == other.Location.Country && p.Location.City == other.Location.City
}
```

## `Enum`

```rust
pub enum Status {
    Pending,
    Scheduled,
    Running,
    Idle,
    Stopped,
}
```

```go
package enum

type Status int

const (
	Pending Status = iota
	Scheduled
	Running
	Idle
	Stopped
)
```

## `interface` VS `trait`

* Rust Trait

```rust
pub trait AppRunner {
    fn run_app(&self) -> Status;
}

pub struct Mac {}

impl AppRunner for Mac {
    fn run_app(&self) -> Status {
        println!("Mac runing app");
        Status::Pending
    }
}

pub struct IPhone {}

impl AppRunner for IPhone {
    fn run_app(&self) -> Status {
        println!("IPhone runing app");
        Status::Scheduled
    }
}

pub struct Android {}

impl AppRunner for Android {
    fn run_app(&self) -> Status {
        println!("Android runing app");
        Status::Running
    }
}

pub fn run_apps(runners: Vec<Box<dyn AppRunner>>) {
    for runner in runners {
        println!("{:?}", runner.run_app());
    }
}
```

Run in Main

```rust
use basic_of_rs::common_behavior::{run_apps, Android, AppRunner, IPhone, Mac};

fn main() {
    println!("Hello, world!");

    let systems: Vec<Box<dyn AppRunner>> =
        vec![Box::new(Mac {}), Box::new(IPhone {}), Box::new(Android {})];

    /*
    Hello, world!
    Mac runing app
    Pending
    IPhone runing app
    Scheduled
    Android runing app
    Running
    */
    run_apps(systems);
}

```

* Golang `interface`

```go
package commonbehave

import "fmt"

type Status int

const (
	Pending Status = iota
	Scheduled
	Running
	Idle
	Stopped
)

type AppRunner interface {
	RunApp() Status
}

type MacSystem struct{}
type IPhone struct{}
type Android struct{}

func (ms *MacSystem) RunApp() Status {
	fmt.Println("Mac running App")
	return Pending
}

func (ms *IPhone) RunApp() Status {
	fmt.Println("IPhone running App")
	return Scheduled
}

func (ms *Android) RunApp() Status {
	fmt.Println("Android running App")
	return Running
}

func RunApps(runners ...AppRunner) {
	for _, runner := range runners {
		fmt.Printf("run result: %v\n", runner.RunApp())
	}
}
```

* run apps

```go
package commonbehave

import "testing"

func TestRunApps(t *testing.T) {
    RunApps(&MacSystem{}, &Android{}, &IPhone{})
}

/*
Mac running App
run result: 0
Android running App
run result: 2
IPhone running App
run result: 1
*/
```


## Type System

## Generic

## Error handling

## Channel

## Async Programming

## Macro
