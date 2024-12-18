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



## Error handling

### Rust can use `?` to propagate the error to upper level function

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum Error {
    #[error("empty elements")]
    EmptyElements,
    #[error("invalid num")]
    InvalidNum,
}

pub fn sum_nums(nums: &[i32]) -> Result<i32, Error> {
    if nums.len() == 0 {
        return Err(Error::EmptyElements);
    }

    let mut total = 0;
    let mut idx = 0 as usize;
    loop {
        if idx >= nums.len() {
            break;
        }

        let j = idx + 1;

        let mut next = 0 as i32;
        if j < nums.len() {
            next = nums[j];
        }

        let sum = sum2(nums[idx], next)?;
        total += sum;
        idx = j + 1;
    }

    Ok(total)
}

fn sum2(a: i32, b: i32) -> Result<i32, Error> {
    if b < 0 {
        return Err(Error::InvalidNum);
    }

    Ok(a + b)
}

```

### Go need explictily return the error to the upper level function

```go
package errors

import "fmt"

func Main(elements []int32) (int32, error) {
	sum, err := sumAll(elements...)
	if err != nil {
		return -1, err
	}

	return sum, nil
}

func sumAll(elements ...int32) (int32, error) {
	length := len(elements)
	if length == 0 {
		return -1, fmt.Errorf("empty elements")
	}

	var total int32
	for i := 0; i < len(elements); {
		j := i + 1
		next := int32(0)

		if j < len(elements) {
			next = elements[j]

		}
		sum, err := sum2(elements[i], next)
		if err != nil {
			return -1, err
		}

		total += sum

		i += 2
	}

	return total, nil
}

func sum2(a, b int32) (int32, error) {
	if b < 0 {
		return -1, fmt.Errorf("b is not less than 0")
	}

	return a + b, nil
}
```

## Unit Test

* Rust: put the test module under the code that be tested

with

```rust
#[cfg(test)]
mod tests {

    #[test]
    fn test_fn() {
        // call test fn
    }
}
```

```rust
#[cfg(test)]
mod tests {
    use super::sum_nums;

    #[test]
    fn test_sum_works() -> anyhow::Result<()> {
        let sum = sum_nums(vec![1, 2, 3].as_slice())?;
        assert_eq!(sum, 6);
        Ok(())
    }
}

```

### Put unit test code under same package and name with `xxx_test.go`

```go
package errors

import "testing"

func TestMain(t *testing.T) {
	tests := []struct {
		name    string
		args    []int32
		want    int32
		wantErr bool
	}{
		{
			name:    "happy path",
			args:    []int32{1, 3, 9, 189, 200},
			want:    213 + 189,
			wantErr: false,
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := Main(tt.args)
			if (err != nil) != tt.wantErr {
				t.Errorf("Main() error = %v, wantErr %v", err, tt.wantErr)
				return
			}

			if got != tt.want {
				t.Errorf("Main() = %v, want %v", got, tt.want)
			}
		})
	}
}

```

## Collections

### Rust

```rust
use std::collections::{BTreeMap, BTreeSet, HashMap, HashSet, VecDeque};

pub fn vec(nums: &[i32]) -> Vec<i32> {
    let mut num_list = Vec::new();
    for num in nums {
        num_list.push(*num);
    }

    num_list
}

pub fn hash_map(keys: &[&str], values: &[&str]) -> HashMap<String, String> {
    let mut idx = 0 as usize;
    let mut key_val: HashMap<String, String> = HashMap::new();
    loop {
        if idx >= keys.len() {
            break;
        }

        key_val.insert(keys[idx].to_string(), values[idx].to_string());
        idx += 1;
    }

    key_val
}

pub fn hash_set(elements: Vec<String>, key: &str) -> (HashSet<String>, bool) {
    let mut set: HashSet<String> = HashSet::new();
    for e in elements {
        set.insert(e);
    }

    let contain = set.contains(key);
    (set, contain)
}

pub fn btree_set(elements: Vec<String>) -> BTreeSet<String> {
    let mut bs = BTreeSet::new();
    for e in elements {
        bs.insert(e);
    }

    bs
}

pub fn vec_dequeue(elements: Vec<String>) -> VecDeque<String> {
    let mut queue = VecDeque::new();
    for e in elements {
        queue.push_front(e);
    }

    queue
}

pub fn btree_map(elements: Vec<(String, String)>) -> BTreeMap<String, String> {
    let mut btree_map = BTreeMap::new();
    for (key, val) in elements {
        btree_map.insert(key, val);
    }

    btree_map
}
```

### Golang

```go
package collections

func Array(nums ...int32) [3]int32 {
	arr := [3]int32{}
	for idx, num := range nums {
		if idx > 2 {
			break
		}
		arr[idx] = num
	}

	return arr
}

func Slice(nums ...int32) []int32 {
	numList := make([]int32, 0, len(nums))
	numList = append(numList, nums...)
	return numList
}

func HashMap(keys []string, values []string) map[string]string {
	idx := 0
	hashMap := make(map[string]string, len(keys))
	for {
		if idx >= len(keys) {
			break
		}

		hashMap[keys[idx]] = values[idx]
		idx++
	}

	return hashMap
}

func HashSet(keys []string) map[string]struct{} {
	set := make(map[string]struct{}, len(keys))
	for _, key := range keys {
		set[key] = struct{}{}
	}

	return set
}
```

## Concurency

### Rust Concurrency Programming

```rust
use std::{
    sync::{
        mpsc::{self, Receiver},
        Arc, Mutex,
    },
    thread,
};

pub fn threads() {
    let handle1 = thread::spawn(|| {
        println!("thread1 running");
    });

    let handle2 = thread::spawn(|| {
        println!("running thread2");
    });

    let handle3 = thread::spawn(|| {
        println!("thread3 running");
    });

    for handle in vec![handle1, handle2, handle3] {
        handle.join().unwrap();
    }
}

pub fn sequare_nums() {
    let nums = vec![2, 3, 8, 9, 100];
    let (rx, tx) = mpsc::channel();
    let h1 = thread::spawn(move || {
        for num in nums {
            rx.send(num).unwrap();
        }
    });

    /*
    ---- concur::tests::test_sequare_nums stdout ----
    calculating num sequare
    received: 4
    received: 9
    received: 64
    received: 81
    received: 10000
        */
    let h2 = thread::spawn(|| {
        for num in sq(tx) {
            println!("received: {}", num);
        }
    });

    h1.join().unwrap();
    h2.join().unwrap();
}

fn sq(nums: Receiver<i32>) -> Receiver<i32> {
    println!("calculating num sequare");
    let (rx, tx) = mpsc::channel();
    let _ = thread::spawn(move || {
        for num in nums {
            rx.send(num * num).unwrap();
        }
    });

    tx
}

pub fn concurrent_counter() -> i32 {
    let counter = Arc::new(Mutex::new(0));
    let mut handlers = vec![];

    for _ in 0..100 {
        let cnt_clone = counter.clone();
        let h = thread::spawn(move || {
            let mut cnt = cnt_clone.lock().unwrap();
            *cnt += 1;
        });

        handlers.push(h);
    }

    for handler in handlers {
        handler.join().unwrap();
    }

    let value = *counter.lock().unwrap();
    value
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_channel() {
        threads();
    }

    #[test]
    fn test_sequare_nums() {
        sequare_nums();
    }

    #[test]
    fn test_concurrent_counter() {
        let count = concurrent_counter();
        assert_eq!(count, 100);
    }
}

```

### Golang Concurrency Programming

```go
package concurrency

import (
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)

func RuntThreads() {
	go func() {
		fmt.Println("running thread1")
	}()

	go func() {
		fmt.Println("running thread2")
	}()

	go func() {
		fmt.Println("running thread3")
	}()

	// waiting all thread done
	time.Sleep(1000 * time.Millisecond)
}

func SendMessage(nums ...int32) []int32 {
	inputCh := sendNums(nums)
	valuesCh := sequereNums(inputCh)

	var result []int32
	for val := range valuesCh {
		result = append(result, val)
	}
	return result
}

func sendNums(nums []int32) chan int32 {
	numsCh := make(chan int32, len(nums))
	go func() {
		for _, num := range nums {
			numsCh <- num
		}
		close(numsCh)
	}()

	return numsCh
}

func sequereNums(input chan int32) <-chan int32 {
	valuesCh := make(chan int32, len(input))
	go func() {
		for num := range input {
			valuesCh <- num * num
		}
		close(valuesCh)
	}()

	return valuesCh
}

func Counter(initVal int64, times int64) int64 {
	var mu sync.Mutex
	var wg sync.WaitGroup

	var idx = int64(0)
	for idx < times {

		wg.Add(1)
		go func() {
			fmt.Printf("adding one to init value\n")

			defer wg.Done()
			mu.Lock()
			initVal += 1
			mu.Unlock()
		}()
		idx++
	}

	wg.Wait()
	return initVal
}

func AtomicCounter1(initVal int64, times int64) int64 {
	var wg sync.WaitGroup

	var idx = int64(0)
	for idx < times {
		wg.Add(1)
		go func() {
			defer wg.Done()
			atomic.AddInt64(&initVal, 1)
		}()

		idx++
	}

	wg.Wait()
	return initVal
}

```
