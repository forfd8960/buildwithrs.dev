---
sidebar_position: 17
---

# Rust Modules and Visibility

Rust’s module system organizes code into logical units, controls visibility, and manages access using `mod`, `pub`, and module paths. Below, I’ll explain the key concepts and provide code examples for different cases, keeping it concise yet comprehensive.

## Key Concepts

1. **Modules (`mod`)**
   - Defined with the `mod` keyword to group related code (functions, structs, enums, etc.).
   - Modules can be nested and are declared either inline or in separate files.
   - By default, module contents are private.

2. **Visibility (`pub`)**
   - The `pub` keyword makes items (functions, structs, etc.) accessible outside their module.
   - Without `pub`, items are private to their module or parent module.
   - Fine-grained control is possible, e.g., `pub(crate)` limits visibility to the current crate.

3. **Module Paths**
   - Access items using paths: absolute (`crate::module::item`) or relative (`self::item`, `super::item`).
   - The `use` keyword imports items to simplify paths.

4. **File-based Modules**
   - Modules can be defined in separate files, automatically recognized by Rust based on file names.
   - Use `mod.rs` for directory-based modules or direct file names for single-file modules.

## Code Examples

### 1. Basic Module and Visibility

Demonstrates `mod`, `pub`, and private items.

```rust
mod my_module {
    // Private by default
    fn private_function() {
        println!("This is private");
    }

    // Public function
    pub fn public_function() {
        println!("This is public");
        private_function(); // Can call private function within the same module
    }
}

fn main() {
    // my_module::private_function(); // Error: private_function is not accessible
    my_module::public_function(); // Works: prints "This is public" and "This is private"
}
```

**Explanation**:

- `private_function` is inaccessible outside `my_module`.
- `public_function` is accessible and can call `private_function` within the same module.

#### 2. Nested Modules and `pub` Variants

Shows nested modules and visibility control with `pub`, `pub(crate)`, and `pub(super)`.

```rust
mod outer {
    pub fn outer_function() {
        println!("Outer function");
    }

    pub(crate) mod inner {
        pub fn inner_function() {
            println!("Inner function");
        }

        pub(super) fn super_function() {
            println!("Accessible in parent (outer)");
        }
    }

    fn call_inner() {
        inner::inner_function(); // Accessible within the same crate
        inner::super_function(); // Accessible in parent module
    }
}

fn main() {
    outer::outer_function(); // Works
    outer::inner::inner_function(); // Works: inner_function is pub and accessible in crate
    // outer::inner::super_function(); // Error: super_function is only accessible in parent
    // outer::call_inner(); // Error: call_inner is private
}
```

**Explanation**:

- `pub(crate)` makes `inner_function` accessible within the crate.
- `pub(super)` restricts `super_function` to the parent module (`outer`).
- `call_inner` is private, so it’s inaccessible in `main`.

#### 3. Module Paths and `use`

Illustrates absolute and relative paths with `use` for imports.

```rust
mod frontend {
    pub mod ui {
        pub fn render() {
            println!("Rendering UI");
        }
    }
}

mod backend {
    pub fn process() {
        println!("Processing data");
    }
}

use crate::frontend::ui::render; // Absolute path
use self::backend::process; // Relative path

fn main() {
    render(); // Imported, so no need for full path
    process();
}
```

**Explanation**:

- `use crate::frontend::ui::render` imports `render` using an absolute path.
- `use self::backend::process` uses a relative path from the current module.
- Imports simplify calling functions without full paths.

#### 4. File-based Modules

Shows how to organize modules across files.

**Directory Structure**:

```bash
src/
  main.rs
  utils.rs
  services/
    mod.rs
    auth.rs
```

**src/main.rs**:

```rust
mod utils; // Loads utils.rs
mod services; // Loads services/mod.rs

fn main() {
    utils::greet(); // Call function from utils.rs
    services::auth::login(); // Call function from services/auth.rs
}
```

**src/utils.rs**:

```rust
pub fn greet() {
    println!("Hello from utils!");
}
```

**src/services/mod.rs**:

```rust
pub mod auth; // Declares auth module (loads auth.rs)
```

**src/services/auth.rs**:

```rust
pub fn login() {
    println!("Logging in...");
}
```

**Explanation**:

- `mod utils;` loads `utils.rs` as a module.
- `mod services;` loads `services/mod.rs`, which in turn declares the `auth` module.
- Functions in `utils.rs` and `auth.rs` are accessible if marked `pub`.

#### 5. Re-exporting with `pub use`

Demonstrates re-exporting items for easier access.

```rust
mod deep {
    pub mod nested {
        pub fn hidden_function() {
            println!("Deeply nested function");
        }
    }
}

// Re-export nested function to crate root
pub use deep::nested::hidden_function;

fn main() {
    hidden_function(); // Directly accessible due to re-export
}
```

**Explanation**:

- `pub use` makes `hidden_function` available at the crate root, simplifying access.

#### 6. Structs and Enums in Modules

Shows visibility with structs and enums.

```rust
mod shapes {
    pub struct Circle {
        pub radius: f32, // Public field
        center: (f32, f32), // Private field
    }

    pub enum ShapeType {
        Circle,
        Square,
    }

    impl Circle {
        pub fn new(radius: f32, center: (f32, f32)) -> Self {
            Circle { radius, center }
        }
    }
}

fn main() {
    let circle = shapes::Circle::new(5.0, (0.0, 0.0));
    println!("Radius: {}", circle.radius); // Works
    // println!("Center: {:?}", circle.center); // Error: center is private
}
```

**Explanation**:

- `Circle` and its `radius` field are public, but `center` is private.
- `ShapeType` enum variants are public if the enum is public.

### Key Takeaways

- Use `mod` to define modules, `pub` to control visibility.
- Paths (`crate::`, `self::`, `super::`) navigate module hierarchies.
- `use` simplifies access, and `pub use` re-exports items.
- File-based modules map to file/directory structures.
- Visibility applies to structs, enums, and their fields/methods.

For more details, check the Rust Book: https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html.