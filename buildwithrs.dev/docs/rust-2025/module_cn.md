---
sidebar_position: 17
---

# Rust 模块和可见性

Rust 的模块系统将代码组织成逻辑单元，控制可见性，并使用 `mod`、`pub` 和模块路径管理访问。

## 关键概念

1. **模块 (`mod`)**
   - 使用 `mod` 关键字定义，用于对相关代码（函数、结构体、枚举等）进行分组。
   - 模块可以嵌套，并且可以在内联或单独的文件中声明。
   - 默认情况下，模块内容是私有的。

2. **可见性 (`pub`)**
   - `pub` 关键字使项目（函数、结构体等）可以在其模块外部访问。
   - 如果没有 `pub`，则项目对其模块或父模块是私有的。
   - 可以进行细粒度控制，例如，`pub(crate)` 将可见性限制为当前 crate。

3. **模块路径**
   - 使用路径访问项目：绝对路径 (`crate::module::item`) 或相对路径 (`self::item`, `super::item`)。
   - `use` 关键字导入项目以简化路径。

4. **基于文件的模块**
   - 模块可以在单独的文件中定义，Rust 会根据文件名自动识别。
   - 对于基于目录的模块，使用 `mod.rs`，对于单文件模块，使用直接文件名。

## 代码示例

### 1. 基本模块和可见性


```rust
mod my_module {
    // 默认私有
    fn private_function() {
        println!("这是私有的");
    }

    // 公有函数
    pub fn public_function() {
        println!("这是公有的");
        private_function(); // 可以在同一模块内调用私有函数
    }
}

fn main() {
    // my_module::private_function(); // 错误：private_function 不可访问
    my_module::public_function(); // 有效：打印 "这是公有的" 和 "这是私有的"
}
```

- `private_function` 在 `my_module` 外部不可访问。
- `public_function` 可以访问，并且可以在同一模块内调用 `private_function。`

### 2. 嵌套模块和 pub 变体

显示嵌套模块和使用 `pub、pub(crate)` 和 `pub(super)` 的可见性控制。

```rust
mod outer {
    pub fn outer_function() {
        println!("外部函数");
    }

    pub(crate) mod inner {
        pub fn inner_function() {
            println!("内部函数");
        }

        pub(super) fn super_function() {
            println!("在父模块 (outer) 中可访问");
        }
    }

    fn call_inner() {
        inner::inner_function(); // 在同一 crate 内可访问
        inner::super_function(); // 在父模块中可访问
    }
}

fn main() {
    outer::outer_function(); // 有效
    outer::inner::inner_function(); // 有效：inner_function 是 pub 并且在 crate 中可访问
    // outer::inner::super_function(); // 错误：super_function 仅在父模块中可访问
    // outer::call_inner(); // 错误：call_inner 是私有的
}
```

- `pub(crate)` 使 `inner_function` 在 `crate` 内可访问。
- `pub(super)` 将 `super_function` 限制为父模块 (`outer`)。
- `call_inner` 是私有的，因此在 main 中不可访问。


### 3. 模块路径和 use

使用 use 导入绝对路径和相对路径。

```rust
mod frontend {
    pub mod ui {
        pub fn render() {
            println!("渲染 UI");
        }
    }
}

mod backend {
    pub fn process() {
        println!("处理数据");
    }
}

use crate::frontend::ui::render; // 绝对路径
use self::backend::process; // 相对路径

fn main() {
    render(); // 已导入，因此无需完整路径
    process();
}
```

- `use crate::frontend::ui::render` 使用绝对路径导入 `render。`
- `use self::backend::process` 使用从当前模块开始的相对路径。
- 导入简化了调用函数，而无需完整路径。


### 4. 基于文件的模块

显示如何在文件中组织模块。

目录结构:

```sh
src/
  main.rs
  utils.rs
  services/
    mod.rs
    auth.rs
```

- src/main.rs:

```rust
mod utils; // 加载 utils.rs
mod services; // 加载 services/mod.rs

fn main() {
    utils::greet(); // 调用 utils.rs 中的函数
    services::auth::login(); // 调用 services/auth.rs 中的函数
}
```

- src/utils.rs:

```rust
pub fn greet() {
    println!("来自 utils 的问候！");
}
```

- src/services/mod.rs:

```rust
pub mod auth; // 声明 auth 模块 (加载 auth.rs)
```

- src/services/auth.rs:

```rust
pub fn login() {
    println!("登录中...");
}
```

- `mod utils; 将 utils.rs 加载为模块。`
- `mod services; 加载 services/mod.rs，后者又声明 auth` 模- 块。
- 如果标记为 pub，则可以访问 utils.rs 和 auth.rs 中的函数。


### 5. 使用 pub use 重新导出

重新导出项目以简化访问。

```rust

mod deep {
    pub mod nested {
        pub fn hidden_function() {
            println!("深度嵌套函数");
        }
    }
}

// 重新导出嵌套函数到 crate 根目录
pub use deep::nested::hidden_function;

fn main() {
    hidden_function(); // 由于重新导出而直接可访问
}
```

## 总结

- 使用 mod 定义模块，使用 pub 控制可见性。
- 路径 `(crate::、self::、super::)` 导航模块层次结构。
- `use` 简化访问，`pub use` 重新导出项目。
- 基于文件的模块映射到文件/目录结构。
- 可见性适用于结构体、枚举及其字段/方法。