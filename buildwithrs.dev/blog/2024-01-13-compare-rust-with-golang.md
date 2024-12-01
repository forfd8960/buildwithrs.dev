---
slug: compare-rust-with-golang
title: Comapre Rust with Golang
authors: [forfd8960]
tags: [golang, rust]
---

I have been using golang long time to wirte backend business logic.

After writing some rust code and I want do a comparison between golang and rust. This is maybe another good way to help you learn rust.

<!-- truncate -->

## Basic Data Types

### Golang Boolean

```go
var truthy bool = true
var notTruthy bool = false
```

### Rust Boolean

```rust
let t: bool = true;
let f: bool = false;
```

### Golang Number

```go
var unum0 uint8 = 100
var unum2 uint16 = 100
var unum4 uint32 = 100
var unum5 uint64 = 100

var num1 int8 = 100
var num2 int16 = 100
var num3 int = 100
var num4 int32 = 100
var num5 int64 = 100

var fnum1 float32 = 1.001
var fnum2 float64 = 99.006
```

### Rust Number

```rust
let un0: u8 = 1;
let un1: u16 = 6;
let un2: u32 = 99;
let un3: u64 = 99;
let un4: u128 = 99;

let n0: i8 = 1;
let n1: i16 = 6;
let n2: i32 = 99;
let n3: i64 = 99;
let n4: i128 = 99;

let f1: f32 = 1.001;
let f2: f64 = 9.009;
```

### Golang String

```go
var helloWord string = "ABC"
```

### Rust String

```rust
let hello_world: String = String::from("Hello, World!");
let hello_world1: &str = "Hello, World!";
```

### Golang Array

```go
/*
[100 0 0 0 0 0 0 0 0 0]
[Hello, World         ]
[97 0 0 0 0 0 0 0 0 0]
*/
var arr [10]int
arr[0] = 100

fmt.Println(arr)

var arr1 [10]string
arr1[0] = "Hello, World"

fmt.Println(arr1)

var arr2 [10]rune
arr2[0] = 'a'

fmt.Println(arr2)
```

### Rust Array

```rust
let arr: [u8; 5] = [1; 5]; // [1, 1, 1, 1, 1]
println!("{:?}", arr);

let arr1: [&str; 5] = ["A"; 5]; // ["A", "A", "A", "A", "A"]
println!("{:?}", arr1);
```

### GolangSlice

```go
var slice = make([]string, 0, 10)
slice = append(slice, 1)

var slice1 = []int{1, 1}
```

### Rust Slice

```rust
let s = String::from("ABC");
let slice: &str = &s[0..2];
println!("{}", slice); //AB

let arr_int = [1, 2, 3];

let slice1: &[i32] = &arr_int[0..2];
println!("{:?}", slice1);   // [1, 2]
```

## Complex data structure

### Golang Struct

```go
type Programmer struct {
	Name   string
	Age    int
	Salary int
}

func NewProgrammer(name string, age, salary int) *Programmer {
	return &Programmer{
		Name:   name,
		Age:    age,
		Salary: salary,
	}
}

func (p *Programmer) String() string {
	return fmt.Sprintf("name: %s\n"+
		"age: %d\n"+
		"salary: %d\n", p.Name, p.Age, p.Salary,
	)
}
```

### Rust Struct

```rust
pub struct Programmer {
    pub name: String,
    pub age: i32,
    pub salary: i32,
}

impl Programmer {
    pub fn new(name: String, age: i32, salary: i32) -> Self {
        Self {
            name: name,
            age: age,
            salary: salary,
        }
    }

    pub fn string(&self) -> String {
        let data = vec![
            format!("name: {}", self.name),
            format!("age: {}", self.age),
            format!("salary: {}", self.salary),
        ];
        data.join("\n")
    }
}

let p = Programmer::new("Bob".to_string(), 28, 10000);
/*
name: Bob
age: 28
salary: 10000
*/
println!("{}", p.string());
```

### Golang HashMap

```go
var m = make(map[string]struct{}, 10)
m["A"] =  struct{}{}
m["B"] =  struct{}{}
```

### Rust HashMap

```rust

use std::collections::{HashMap, HashSet};

/*
{"B": 10, "A": 1, "C": 100}
1
*/

let mut m = HashMap::new();
m.insert("A", 1);
m.insert("B", 10);
m.insert("C", 100);

println!("{:?}", m);

let val: Option<&i32> = m.get("A");
match val {
    Some(v) => println!("{}", v),
    _ => println!("not found: {}", "A"),
}

```

### Golang Set

**Golang has no set data structure in std lib**

### Rust Set

```rust

use std::collections::{HashMap, HashSet};

/*
{"C", "A", "B"}
contains A: true
contains Z: false
*/

let mut set = HashSet::new();
set.insert("A");
set.insert("B");
set.insert("C");

println!("{:?}", set);

let exists = set.contains("A");
println!("contains A: {}", exists);
println!("contains Z: {}", set.contains("Z"));
```
