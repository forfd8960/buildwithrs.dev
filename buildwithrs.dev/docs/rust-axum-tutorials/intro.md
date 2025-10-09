# Building Web Apps with Rust's Axum Framework: A Beginner's Guide


## What is Axum

> axum is a web application framework that focuses on ergonomics and modularity.

In this blog, we are going to do a deep dive into Axum, a web framework for making Rust REST APIs backed by the Tokio team that's simple to use and has hyper-compatibility with Tower, a robust library of reusable, modular components for building network applications.

## Routing in Axum

- first define our `hello world` handler

it will return a static str: `Hello, world!` which implement `IntoResponse`.

```rust
async fn index() -> impl IntoResponse {
    "Hello, World!"
}
```

- then in the get_router, we build a Router with `Router::new()`.
- and map the index handler with `/index` path:  `.route("/index", get(index))`

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

### How to define a `get` method by path parameter.

- define the `get_blog` handler, which need to know `blog_id` as parameter.

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


- in the router, map the `get_blog` handler to path: `/api/v1/blogs/{blog_id}`.

```rust
.route("/api/v1/blogs/{blog_id}", get(get_blog));
```

- result

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

### How to define a `post` handler receieve a Json request.

Tldr:
    - import `serde::{Deserialize, Serialize}`.
    - define a request struct which derive `Deserialize`.
    - define a response struct which derive `Serialize`.

- first define the request and response

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

- define `create_blog` handler

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

- map the `create_blog` with `/api/v1/blogs`

```rust
async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        .route("/api/v1/blogs", post(create_blog));
    api_router
}
```

- result

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


### How to define a `put` handler receieve a Json request.

- define the put handler request with: `Deserialize`.
- set update_blog handler which receive:
```rust
Path(blog_id): Path<u64>,
Json(payload): Json<UpdateBlogReq>
```

- use `Path(blog_id): Path<u64>` to extract the http path parameter.
- map the handler under `/api/v1/blogs/{blog_id}`: `.route("/api/v1/blogs/{blog_id}", put(update_blog))`

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

- result

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

### `delete` method

- define `delete_blog` handler.
- map the delete handler under the path: `/api/v1/blogs/{blog_id}`, with `.route("/api/v1/blogs/{blog_id}", put(update_blog).delete(delete_blog))`
- return Generic response after delete.

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

- result

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

### `AppState`: How to share global state to all handlers.

So to share some global state to all the routes in the Axum App, axum use `State` - a struct dedicated to holding all of your variables that you want to share across your routes on the app.

For example, you may want put the **DB Pool**, and **Redis Client**, **JWT Encoding Key**, etc to the State.

below is an example of `AppState` that contains a DB pool to PostgresDB.

- define the `AppState` struct.
- implement   `Deref` for `AppState`

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

- Share the AppState to all Routes: `with_state(app_state)`.

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

- How to use the shared state in the handler: 

   **Put `State(app_state): State<AppState>` as the first parameter in the handler function**

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

### Middleware in Axum

Since `Axum` is hyper-compatible with the `tower` crates, which means that we can effectively use any `Tower` middleware that we want for our Rust API! For example, we can add a Tower middleware to compress responses:

- add `tower-http`

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

- timeout middleware

```rust
use tower_http::timeout::TimeoutLayer;

async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        .route("/api/v1/blogs", post(create_blog))
        .route("/api/v1/blogs/{blog_id}", get(get_blog).put(update_blog).delete(delete_blog))
        .layer(CompressionLayer::new().zstd(true))
        .layer(TimeoutLayer::new(Duration::from_secs(50))); // set time out to be 50s
    api_router
}
```

We can also create our own middleware by writing a function.

- `check_json_header` will check if the http header contains `"application/json"`, if not return bad request.

- the `Request` is: `use axum::{extract::{Path, Request}`
- the `Next` is: `axum::middleware::Next`

```rust
async fn check_json_header(
    req: Request,
    next: Next
) -> Result<Response, StatusCode> {
    // requires the http crate to get the header name
    if req.headers().get("Content-Type").unwrap() != "application/json" {
        return Err(StatusCode::BAD_REQUEST);
    }

    Ok(next.run(req).await)
}
```

- add the middleware to router

```rust
async fn get_router() -> Router {
    let api_router = Router::new()
        .route("/index", get(index))
        // ...
        .layer(axum::middleware::from_fn(check_json_header));
    api_router
}

```

- result

```sh
# bad request
GET /index HTTP/1.1
Host: localhost:8080
Connection: close

HTTP/1.1 400 Bad Request
connection: close
content-length: 0
date: Thu, 09 Oct 2025 07:34:29 GMT

# success pass header check
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

## Serving Static Files in Axum

Let's say you want to serve some static files using Axum.

Axum does not by itself have capabilities to be able to do this; however, what it does have is super-strong compatibility with tower-http, which offers utility for serving your own static files.

- enable `fs` feature

```toml
tower-http = { version = "0.6.6", features = ["cors", "fs"] }
```

- call `nest_service` in the Router.

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

## Deploy in Docker Container

### build the image

- create `Dockerfile` under the project root.

```sh
FROM lukemathwalker/cargo-chef:latest-rust-1 AS chef

# Create and change to the app directory.
WORKDIR /app

FROM chef AS planner
COPY . ./
RUN cargo chef prepare --recipe-path recipe.json

FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json

# Build dependencies - this is the caching Docker layer!
RUN cargo chef cook --release --recipe-path recipe.json

# Build application
COPY . ./
RUN cargo build --release

CMD ["./target/release/axum-app"]
```

- build docker image

```sh
docker build -t axum-app .

➜  axum-examples git:(master) ✗ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
axum-app     latest    2940a69d9a44   9 minutes ago   6.3GB

```

- run docker image

```sh
➜  axum-examples git:(master) ✗ docker run -p 8080:8080 axum-app
Listening on: 0.0.0.0:8080
```

## Summary

In this blog post, we shared how to use Axum to build web app(API)

- use `Router::new()` to build the routes for your app.
- define `async fn your_handler()` to define your handler.
- use `axum::routing::{get, post, put, delete}` to wrap the handler and map in the route path.
- use `Path(param): Path<ParamType>` to extract path parameter.
- use  `State(state): State<AppState>` to share the global state to all routes.
- set middleware with `tower_http`: `.layer(CompressionLayer::new().zstd(true))`, `.layer(TimeoutLayer::new(Duration::from_secs(50)))`
- build your own middeware with:
```rust
async fn your_middleware(
    req: Request,
    next: Next
) -> Result<Response, StatusCode>
```

- how to build and deploy your axum app with Docker. 