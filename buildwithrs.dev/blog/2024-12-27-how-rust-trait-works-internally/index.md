---
slug: how-rust-trait-works-internally
title: How Rust Trait Works Internally
authors: forfd8960
tags: [rust, trait]
---


Rust's traits are a powerful feature that enables polymorphism and code reuse. Internally, traits are implemented using a combination of **static dispatch (monomorphization)** and **dynamic dispatch (vtable)** mechanisms, depending on how they are used. Here's a detailed breakdown of how traits work internally:

<!-- truncate -->

---

## 1. **Trait Definition**

A trait defines a set of methods that types can implement. For example:

```rust
trait Draw {
    fn draw(&self);
}
```

This is essentially a blueprint for behavior that types can adhere to.

---

## 2. **Trait Implementation**

Types can implement a trait by providing concrete implementations for its methods:

```rust
struct Circle;

impl Draw for Circle {
    fn draw(&self) {
        println!("Drawing a circle");
    }
}
```

At this stage, the compiler knows that `Circle` implements the `Draw` trait.

---

## 3. **Static Dispatch (Monomorphization)**

When traits are used with **generics**, Rust performs **static dispatch**. This means the compiler generates specialized code for each concrete type that implements the trait.

Example:

```rust
fn draw_shape<T: Draw>(shape: &T) {
    shape.draw();
}
```

Here, the compiler generates a separate version of `draw_shape` for each type that implements `Draw`. For instance, if `Circle` and `Square` both implement `Draw`, the compiler generates:

```rust
fn draw_shape_for_circle(circle: &Circle) {
    circle.draw();
}

fn draw_shape_for_square(square: &Square) {
    square.draw();
}
```

This process is called **monomorphization**, and it results in highly efficient code because the method calls are resolved at compile time.

---

## 4. **Dynamic Dispatch (Vtables)**

When traits are used with **trait objects**, Rust performs **dynamic dispatch**. This allows for runtime polymorphism, where the exact method to call is determined at runtime.

Example:

```rust
fn draw_shape(shape: &dyn Draw) {
    shape.draw();
}
```

Here, `shape` is a **trait object**, which is a fat pointer consisting of:

- A pointer to the actual data.
- A pointer to a **vtable** (virtual method table).

The **vtable** is a lookup table that contains function pointers to the trait methods for the specific type. For example, if `Circle` implements `Draw`, the vtable for `Circle` will contain a pointer to `Circle::draw`.

At runtime, the correct method is looked up in the vtable and called.

---

## 5. **Trait Objects and Memory Layout**

A trait object (`&dyn Trait`, `Box<dyn Trait>`, etc.) has the following memory layout:

```rust
struct TraitObject {
    data: *mut (),            // Pointer to the actual data
    vtable: *mut VTable,      // Pointer to the vtable
}

struct VTable {
    drop: fn(*mut ()),        // Destructor
    size: usize,              // Size of the type
    align: usize,             // Alignment of the type
    method1: fn(*mut ()),     // Pointer to the first method
    method2: fn(*mut ()),     // Pointer to the second method
    // ...
}
```

When you call a method on a trait object, the compiler uses the vtable to find the correct function pointer and calls it.

---

## 6. **Trait Bounds and Where Clauses**

Trait bounds (`T: Trait`) and `where` clauses are used to constrain generic types to only those that implement specific traits. These are resolved at compile time and enable static dispatch.

Example:

```rust
fn process<T>(item: T)
where
    T: Draw + Clone,
{
    item.draw();
    let cloned = item.clone();
}
```

The compiler ensures that `T` implements both `Draw` and `Clone`, and generates specialized code accordingly.

---

## 7. **Associated Types and Default Methods**

Traits can also include **associated types** and **default methods**:

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    fn count(self) -> usize {
        // Default implementation
        let mut count = 0;
        while self.next().is_some() {
            count += 1;
        }
        count
    }
}
```

Associated types are resolved at compile time, and default methods can be overridden by implementers.

---

## 8. **Trait Coherence and Orphan Rules**

Rust enforces **trait coherence** to ensure that trait implementations are unambiguous. The **orphan rule** states that you can only implement a trait for a type if either:

- The trait is defined in your crate, or
- The type is defined in your crate.

This prevents conflicting implementations from different crates.

---

## Summary

- **Static Dispatch**: Used with generics; resolved at compile time via monomorphization.
- **Dynamic Dispatch**: Used with trait objects; resolved at runtime via vtables.
- **Trait Objects**: Fat pointers containing a data pointer and a vtable.
- **Trait Bounds**: Constrain generics to types that implement specific traits.
- **Associated Types and Default Methods**: Extend traits with additional functionality.
