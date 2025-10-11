# 使用 Rust 的 Axum 框架构建 Web 应用：初学者指南

## Axum 是什么

> axum 是一个专注于人体工程学和模块化的 Web 应用框架。

在这篇博客中，我们将深入探讨 Axum，这是一个由 Tokio 团队支持的用于构建 Rust REST API 的 Web 框架，它使用简单，并且与 Tower 高度兼容，Tower 是一个用于构建网络应用的强大、可重用、模块化组件库。

## Axum 中的路由

- 首先定义我们的 `hello world` 处理函数

它将返回一个静态字符串：`Hello, world!`，该字符串实现了 `IntoResponse`。

```rust
async fn index() -> impl IntoResponse {
    "Hello, World!"
}
```

- 然后在 `get_router` 中，我们使用 `Router::new()` 构建一个 Router。
- 并将 `index` 处理函数映射到 `/index` 路径：`.route("/index", get(index))`

```rust
async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index));
    api_router
}
```


```rust
use axum::{response::IntoResponse, routing::get, Router};
use tokio::net::TcpListener;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let router = get_router().await;

    let addr = format!("0.0.0.0:{}", "8080");
    let listener = TcpListener::bind(&addr).await?;
    println!("Listening on: {}", addr);

    axum::serve(listener, router.into_make_service()).await?;
    Ok(())
}

async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index));
    api_router
}

async fn index() -> impl IntoResponse {
    "Hello, World!"
}
```

### 如何通过路径参数定义 `get` 方法。

- 定义 `get_blog` 处理函数，它需要 `blog_id` 作为参数。

```rust
use axum::{extract::Path, response::IntoResponse, routing::{get, post, put}, Json, Router};

pub async fn get_blog(
    Path(blog_id): Path<u64>,
) -> impl IntoResponse {
    println!("get blog: {}", blog_id);

    Json(BlogResp {
        title: "Learn Rust".to_string(),
        content: "Rust is a systems programming language...".to_string(),
    })
}
```


- 在路由器中，将 `get_blog` 处理函数映射到路径：`/api/v1/blogs/{blog_id}`。

```rust
.route("/api/v1/blogs/{blog_id}", get(get_blog));
```

- 结果

```sh
GET /api/v1/blogs/1 HTTP/1.1
Host: localhost:8080
Connection: close
User-Agent: RapidAPI/4.4.3 (Macintosh; OS X/15.3.1) GCDHTTPRequest


HTTP/1.1 200 OK
content-type: application/json
content-length: 76
connection: close
date: Thu, 09 Oct 2025 06:57:18 GMT

{"title":"Learn Rust","content":"Rust is a systems programming language..."}
```

### 如何定义接收 JSON 请求的 `post` 处理函数。

简而言之：
    - 导入 `serde::{Deserialize, Serialize}`。
    - 定义一个派生 `Deserialize` 的请求结构体。
    - 定义一个派生 `Serialize` 的响应结构体。

- 首先定义请求和响应

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Deserialize)]
pub struct CreateBlogReq {
    pub title: String,
    pub content: String,
}

#[derive(Debug, Clone, Serialize)]
pub struct BlogResp {
    pub title: String,
    pub content: String,
}
```

- 定义 `create_blog` 处理函数

```rust
pub async fn create_blog(
    Json(payload): Json<CreateBlogReq>,
) -> impl IntoResponse {
    println!("cerate blog: {:?}", payload);

    let resp = BlogResp {
        title: payload.title,
        content: payload.content,
    };

    println!("created blog: {:?}", resp);
    Json(resp)
}
```

- 将 `create_blog` 映射到 `/api/v1/blogs`

```rust
async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        .route("/api/v1/blogs", post(create_blog));
    api_router
}
```

- 结果

```sh
# request
POST /api/v1/blogs HTTP/1.1
Content-Type: application/json; charset=utf-8
Host: localhost:8080
Connection: close
User-Agent: RapidAPI/4.4.3 (Macintosh; OS X/15.3.1) GCDHTTPRequest
Content-Length: 61

{"title":"learn rust","content":"Lifetime, Ownership, Trait"}

# response
HTTP/1.1 200 OK
content-type: application/json
content-length: 61
connection: close
date: Thu, 09 Oct 2025 06:34:56 GMT

{"title":"learn rust","content":"Lifetime, Ownership, Trait"}
```


### 如何定义接收 JSON 请求的 `put` 处理函数。

- 定义带有 `Deserialize` 的 put 处理函数请求。
- 设置 `update_blog` 处理函数，它接收：
```rust
Path(blog_id): Path<u64>,
Json(payload): Json<UpdateBlogReq>
```

- 使用 `Path(blog_id): Path<u64>` 来提取 HTTP 路径参数。
- 将处理函数映射到 `/api/v1/blogs/{blog_id}`：`.route("/api/v1/blogs/{blog_id}", put(update_blog))`

```rust
#[derive(Debug, Clone, Deserialize)]
pub struct UpdateBlogReq {
    pub id: u64,
    pub title: String,
    pub content: String,
}

// main ...

async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        .route("/api/v1/blogs", post(create_blog))
        .route("/api/v1/blogs/{blog_id}", put(update_blog));
    api_router
}

pub async fn update_blog(
    Path(blog_id): Path<u64>,
    Json(payload): Json<UpdateBlogReq>,
) -> impl IntoResponse {
    println!("update blog: {}, {:?}", blog_id, payload);

    let resp = BlogResp {
        title: payload.title,
        content: payload.content,
    };

    println!("updated blog: {:?}", resp);
    Json(resp)
}
```

- 结果

```sh
PUT /api/v1/blogs/1 HTTP/1.1
Content-Type: application/json; charset=utf-8
Host: localhost:8080
Connection: close
User-Agent: RapidAPI/4.4.3 (Macintosh; OS X/15.3.1) GCDHTTPRequest
Content-Length: 68

{"title":"learn rust","content":"Lifetime, Ownership, Trait","id":1}


HTTP/1.1 200 OK
content-type: application/json
content-length: 61
connection: close
date: Thu, 09 Oct 2025 06:41:30 GMT

{"title":"learn rust","content":"Lifetime, Ownership, Trait"}
```

### `delete` 方法

- 定义 `delete_blog` 处理函数。
- 将 delete 处理函数映射到路径：`/api/v1/blogs/{blog_id}`，使用 `.route("/api/v1/blogs/{blog_id}", put(update_blog).delete(delete_blog))`
- 删除后返回通用响应。

```rust
#[derive(Debug, Clone, Serialize)]
pub struct GenericResp {
    pub msg: String,
}


async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        .route("/api/v1/blogs", post(create_blog))
        .route("/api/v1/blogs/{blog_id}", put(update_blog).delete(delete_blog));
    api_router
}

pub async fn delete_blog(
    Path(blog_id): Path<u64>,
) -> impl IntoResponse {
    println!("delete blog: {}", blog_id);

    Json(GenericResp {msg: format!("deleted blog: {}", blog_id)})
}
```

- 结果

```sh
DELETE /api/v1/blogs/1 HTTP/1.1
Host: localhost:8080
Connection: close
User-Agent: RapidAPI/4.4.3 (Macintosh; OS X/15.3.1) GCDHTTPRequest

HTTP/1.1 200 OK
content-type: application/json
content-length: 25
connection: close
date: Thu, 09 Oct 2025 06:47:06 GMT

{"msg":"deleted blog: 1"}
```

### `AppState`：如何将全局状态共享给所有处理函数。

为了将一些全局状态共享给 Axum 应用中的所有路由，Axum 使用 `State` - 一个专用于保存您想要在应用路由中共享的所有变量的结构体。

例如，您可能希望将 **DB Pool**、**Redis 客户端**、**JWT 编码密钥** 等放入 State。

下面是一个包含 PostgresDB DB pool 的 `AppState` 示例。

- 定义 `AppState` 结构体。
- 为 `AppState` 实现 `Deref`

```rust
#[derive(Debug, Clone)]
pub struct AppState {
    pub inner: Arc<AppStateInner>,
}

#[derive(Debug, Clone)]
pub struct AppStateInner {
    pub app_name: String,
    pub pool: sqlx::PgPool,
}

impl Deref for AppState {
    type Target = AppStateInner;

    fn deref(&self) -> &Self::Target {
        &self.inner
    }
}
```

- 将 AppState 共享给所有路由：`with_state(app_state)`。

```rust
async fn get_router(app_state: AppState) -> Router {
    let api_router = Router::new()
        // ...
        .with_state(app_state);
    api_router
}

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let addr = format!("0.0.0.0:{}", "8080");
    let listener = TcpListener::bind(&addr).await?;
    println!("Listening on: {}", addr);
    
    let app_state = AppState {
        inner: Arc::new(AppStateInner {
            app_name: "axum-app".to_string(),
            pool: sqlx::PgPool::connect("postgres://postgres:password@localhost:5432/axum_db").await?,
        }),
    };
    let router = get_router(app_state).await;

    axum::serve(listener, router.into_make_service()).await?;
    Ok(())
}
```

- 如何在处理函数中使用共享状态： 

   **将 `State(app_state): State<AppState>` 作为处理函数的第一个参数**

```rust
pub async fn create_blog(
    State(app_state): State<AppState>,
    Json(payload): Json<CreateBlogReq>,
) -> impl IntoResponse {
    println!("{} cerate blog: {:?}", app_state.app_name, payload);

    let resp = BlogResp {
        title: payload.title,
        content: payload.content,
    };

    println!("created blog: {:?}", resp);
    Json(resp)
}

// output
// axum-app cerate blog: CreateBlogReq { title: "learn rust1", content: "Vector, Iterator, Async Programming" }
// created blog: BlogResp { title: "learn rust1", content: "Vector, Iterator, Async Programming" }
```

### Axum 中的中间件

由于 `Axum` 与 `tower`  crate 高度兼容，这意味着我们可以有效地为我们的 Rust API 使用任何 `Tower` 中间件！例如，我们可以添加一个 Tower 中间件来压缩响应：

- 添加 `tower-http`

```toml
tower = "0.5.2"
tower-http = { version = "0.6.6", features = ["cors", "compression-zstd", "decompression-zstd", "trace", "timeout"] }
```

```rust
use tower_http::compression::CompressionLayer;

async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        .route("/api/v1/blogs", post(create_blog))
        .route("/api/v1/blogs/{blog_id}", get(get_blog).put(update_blog).delete(delete_blog))
        .layer(CompressionLayer::new().zstd(true));
    api_router
}

```

- 超时中间件

```rust
use tower_http::timeout::TimeoutLayer;

async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        .route("/api/v1/blogs", post(create_blog))
        .route("/api/v1/blogs/{blog_id}", get(get_blog).put(update_blog).delete(delete_blog))
        .layer(CompressionLayer::new().zstd(true))
        .layer(TimeoutLayer::new(Duration::from_secs(50))); // 设置超时为 50 秒
    api_router
}
```

我们也可以通过编写函数来创建自己的中间件。

- `check_json_header` 将检查 HTTP 头是否包含 `"application/json"`，如果不包含则返回坏请求。

- `Request` 是：`use axum::{extract::{Path, Request}`
- `Next` 是：`axum::middleware::Next`

```rust
async fn check_json_header(
    req: Request,
    next: Next
) -> Result<Response, StatusCode> {
    // 需要 http crate 来获取头名称
    if req.headers().get("Content-Type").unwrap() != "application/json" {
        return Err(StatusCode::BAD_REQUEST);
    }

    Ok(next.run(req).await)
}
```

- 将中间件添加到路由器

```rust
async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        // ...
        .layer(axum::middleware::from_fn(check_json_header));
    api_router
}

```

- 结果

```sh
# 坏请求
GET /index HTTP/1.1
Host: localhost:8080
Connection: close

HTTP/1.1 400 Bad Request
connection: close
content-length: 0
date: Thu, 09 Oct 2025 07:34:29 GMT

# 成功通过头检查
GET /index HTTP/1.1
Content-Type: application/json
Host: localhost:8080
Connection: close

HTTP/1.1 200 OK
content-type: text/plain; charset=utf-8
content-length: 13
connection: close
date: Thu, 09 Oct 2025 07:35:47 GMT

Hello, World!
```

## 在 Axum 中提供静态文件

假设您想使用 Axum 提供一些静态文件。

Axum 本身没有提供静态文件的能力；然而，它与 tower-http 具有超级强大的兼容性，tower-http 提供了用于提供您自己的静态文件的实用工具。

- 启用 `fs` 特性

```toml
tower-http = { version = "0.6.6", features = ["cors", "fs"] }
```

- 在 Router 中调用 `nest_service`。

```rust
use tower_http::services::ServeDir;

async fn get_router() -> Router {
    let api_router = Router::new()
        .nest_service("/static", ServeDir::new("examples"))
        // ...
        .layer(TimeoutLayer::new(Duration::from_secs(50)));
    api_router
}
```

![Serve Static](serve_static.png)

## 在 Docker 容器中部署

### 构建镜像

- 在项目根目录下创建 `Dockerfile`。

```sh
FROM lukemathwalker/cargo-chef:latest-rust-1 AS chef

# 创建并切换到应用目录。
WORKDIR /app

FROM chef AS planner
COPY . ./
RUN cargo chef prepare --recipe-path recipe.json

FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json

# 构建依赖 - 这是缓存的 Docker 层！
RUN cargo chef cook --release --recipe-path recipe.json

# 构建应用
COPY . ./
RUN cargo build --release

CMD ["./target/release/axum-app"]
```

- 构建 Docker 镜像

```sh
docker build -t axum-app .

➜  axum-examples git:(master) ✗ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
axum-app     latest    2940a69d9a44   9 minutes ago   6.3GB

```

- 运行 Docker 镜像

```sh
➜  axum-examples git:(master) ✗ docker run -p 8080:8080 axum-app
Listening on: 0.0.0.0:8080
```

## 总结

在这篇博客文章中，我们分享了如何使用 Axum 构建 Web 应用（API）

- 使用 `Router::new()` 为您的应用构建路由。
- 定义 `async fn your_handler()` 来定义您的处理函数。
- 使用 `axum::routing::{get, post, put, delete}` 来包装处理函数并映射到路由路径。
- 使用 `Path(param): Path<ParamType>` 来提取路径参数。
- 使用 `State(state): State<AppState>` 将全局状态共享给所有路由。
- 使用 `tower_http` 设置中间件：`.layer(CompressionLayer::new().zstd(true))`，`.layer(TimeoutLayer::new(Duration::from_secs(50)))`
- 通过以下方式构建您自己的中间件：
```rust
async fn your_middleware(
    req: Request,
    next: Next
) -> Result<Response, StatusCode>
```

- 如何使用 Docker 构建和部署您的 Axum 应用。