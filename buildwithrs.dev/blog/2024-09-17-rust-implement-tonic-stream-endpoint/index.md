---
slug: rust-implement-tonic-stream-endpoint
title: Rust Implement Tonic Stream Endpoint
authors: forfd8960
tags: [rust, protobuf, tonic, stream]
---

Today want share how to implement a stream endpoint in Rust using the Tonic framework.

## the dependency

```toml
[dependencies]
anyhow = "1.0.82"
chrono = { version = "0.4.38", features = ["serde"] }
derive_builder = "0.20.0"
futures = "0.3.30"
prost = "0.12.4"
prost-types = "0.12.4"
tokio = { version = "1.40.0", features = ["rt", "rt-multi-thread", "macros"] }
tokio-stream = "0.1.15"
tonic = { version = "0.11.0", features = ["zstd", "tls"] }
tonic-build = "0.11.0"
uuid = { version = "1.10.0", features=["v4"]}
fake = { version="2.9.2", features=["derive", "chrono"]}
tracing = "0.1.40"
tracing-subscriber = { version = "0.3.18", features = ["env-filter"] }

[build-dependencies]
anyhow = "1.0.82"
tonic-build = "0.11.0"
```

## Add new stream method to protobuf

```protobuf
syntax = "proto3";

package social;

import "google/protobuf/timestamp.proto";

service SocialService {
    rpc PostFeed(PostFeedRequest) returns (stream PostFeedResponse) {}
}

message PostFeedRequest{
    int32 limit = 1;
}

message PostFeedResponse{
    repeated Post posts = 1;
}

message User {
    string user_id = 1;
    string nick_name = 2;
    string user_name = 3;
    string avatar = 4;
    string bio = 5;
    string email = 6;
    string web_site = 7;
    string birthday = 8;
    google.protobuf.Timestamp created_at = 9;
    google.protobuf.Timestamp updated_at = 10;
}

message Content {
    string text = 1;
    repeated string images = 2;
    repeated string videos = 3;
}

message Post {
    string post_id = 1;
    User user = 2;
    Content content = 3;
    google.protobuf.Timestamp created_at = 4;
    google.protobuf.Timestamp updated_at = 5;
}
```

## And then implement the stream method in SocialServiceServer

* Init a `mpsc::channel` to send the post.
* In the `tokio::spawn` start a loop and send the post to the channel every 3 seconds.
* In the send loop, call `Post::fake()` to generate the post we want to send.
* Init the receiver stream with channel `rx`: `let stream = ReceiverStream::new(rx);` and `Box::pin`, and return the response.

path: `src/abi/service.rs`

```rust

use std::{ops::Deref, pin::Pin, sync::Arc, time::Duration};

use futures::Stream;
use tokio::{sync::mpsc, time::sleep};
use tokio_stream::wrappers::ReceiverStream;
use tonic::{async_trait, Request, Response, Status};
use tracing::warn;

use crate::pb::social::{
    social_service_server::{SocialService, SocialServiceServer},
    GreetRequest, GreetResponse, Post, PostFeedRequest, PostFeedResponse,
};

const CHAN_SIZE: usize = 10;

type ResponseStream = Pin<Box<dyn Stream<Item = Result<PostFeedResponse, Status>> + Send>>;

#[async_trait]
impl SocialService for SocialServiceImpl {
    type PostFeedStream = ResponseStream;
    async fn post_feed(
        &self,
        request: Request<PostFeedRequest>,
    ) -> Result<Response<Self::PostFeedStream>, Status> {
        let req = request.into_inner();
        println!("req: {:?}", req);

        let (tx, rx) = mpsc::channel(CHAN_SIZE);

        tokio::spawn(async move {
            loop {
                println!("send post to sender");

                let response = PostFeedResponse {
                    posts: vec![Post::fake()],
                };
                let _ = tx.send(Ok(response)).await.map_err(|err| {
                    warn!("send post failed: {:?}", err);
                });

                sleep(Duration::from_secs(3)).await;
            }
        });

        let stream = ReceiverStream::new(rx);
        Ok(Response::new(Box::pin(stream)))
    }
}
```

## Fake the post

`src/abi/post.rs`

```rust
use uuid::Uuid;

use crate::pb::social::{Content, Post, User};

use super::now_ts;

impl Post {
    pub fn fake() -> Self {
        Self {
            post_id: Uuid::new_v4().to_string(),
            user: Some(User::fake()),
            content: Some(Content::fake()),
            created_at: Some(now_ts()),
            updated_at: Some(now_ts()),
        }
    }
}
```

## Fake the user

`src/abi/user.rs`

```rust
use crate::pb::social::User;
use fake::faker::internet::en::SafeEmail;
use fake::faker::name::en::Name;
use fake::Fake;
use fake::Faker;
use uuid::Uuid;

use super::now_ts;

impl User {
    pub fn fake() -> Self {
        Self {
            user_id: Uuid::new_v4().to_string(),
            nick_name: Name().fake(),
            user_name: Name().fake(),
            avatar: Faker.fake::<String>(),
            bio: Faker.fake::<String>(),
            email: SafeEmail().fake(),
            web_site: Faker.fake::<String>(),
            birthday: Faker.fake::<String>(),
            created_at: Some(now_ts()),
            updated_at: Some(now_ts()),
        }
    }
}

```

## Fake the content

`src/abi/content.rs`

```rust
use fake::{Fake, Faker};

use crate::pb::social::Content;

impl Content {
    pub fn fake() -> Self {
        Self {
            text: Faker.fake::<String>(),
            images: vec![],
            videos: vec![],
        }
    }
}

```

## How the Client recevier Stream

* Connect the server with: `SocialServiceClient::connect(addr).await?;`
* Init the post feed request: `Request::new(PostFeedRequest { limit: 10 });`
* Call post_feed: `let response = client.post_feed(request).await?;`
* Use `while` continuously get data from the stream: `while let Some(post_resp) = resp.message().await?`

```rust
use std::str::FromStr;

use social::pb::social::{
    social_service_client::SocialServiceClient, PostFeedRequest,
};
use tonic::{
    transport::{Channel, Endpoint},
    Request,
};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    println!("starting social service client");
    let addr = Endpoint::from_str("http://[::1]:9090")?;
    let mut client = SocialServiceClient::connect(addr).await?;

    call_post_feed(&mut client).await?;
    Ok(())
}

async fn call_post_feed(client: &mut SocialServiceClient<Channel>) -> anyhow::Result<()> {
    let request = Request::new(PostFeedRequest { limit: 10 });
    println!("call greet with request: {:?}", request);

    let response = client.post_feed(request).await?;
    let mut resp = response.into_inner();
    while let Some(post_resp) = resp.message().await? {
        println!("get posts: {:?}", post_resp.posts);
    }

    Ok(())
}

```


## Run the server

```sh
social (main)> cargo run --bin social
   Compiling social v0.1.0
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 5.35s
     Running `target/debug/social`
2024-09-17T01:31:43.485993Z  INFO social: starting social service server
2024-09-17T01:31:43.486214Z  INFO social: init social service server...
2024-09-17T01:31:43.486295Z  INFO social: serve social service at [::1]:9090

req: PostFeedRequest { limit: 10 }
send post to sender
send post to sender
send post to sender
2024-09-17T01:36:06.394098Z  WARN social::abi::service: send post failed: SendError { .. }
send post to sender
2024-09-17T01:36:09.398233Z  WARN social::abi::service: send post failed: SendError { .. }
send post to sender
2024-09-17T01:36:12.400842Z  WARN social::abi::service: send post failed: SendError { .. }
```

## Run the Client

```sh
social (main) [SIGINT]> cargo run --bin social-client
   Compiling social v0.1.0 
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.30s
     Running `target/debug/social-client`
starting social service client

call greet with request: Request { metadata: MetadataMap { headers: {} }, message: PostFeedRequest { limit: 10 }, extensions: Extensions }

get posts: [Post { post_id: "8d9854b3-f0a4-4d21-9dbd-0718963d36cb", user: Some(User { user_id: "8aedf511-28cd-4d66-b2a7-7c74b88a69dd", nick_name: "Ofelia Swift", user_name: "Emerald Halvorson", avatar: "refxsVCzlDP2D6LxWd4", bio: "zzFINJYfdhnDPfGnn4", email: "carleton@example.net", web_site: "Uel4iJ", birthday: "wUL6SmuVyJMtXA", created_at: Some(Timestamp { seconds: 1726536960, nanos: 384597000 }), updated_at: Some(Timestamp { seconds: 1726536960, nanos: 384602000 }) }), content: Some(Content { text: "7tRRvoJqwBFHbk1", images: [], videos: [] }), created_at: Some(Timestamp { seconds: 1726536960, nanos: 384604000 }), updated_at: Some(Timestamp { seconds: 1726536960, nanos: 384604000 }) }]

get posts: [Post { post_id: "4eb28096-5573-4e30-973d-9d3f11ee2dd7", user: Some(User { user_id: "0a643764-f439-48a5-add6-d7a5b775a8c2", nick_name: "Ozella Kutch", user_name: "Brendon Kub", avatar: "1Kb0gIloZ2rv", bio: "ksYKahDai", email: "destiney@example.com", web_site: "O7GJ0Ui", birthday: "MkrA0wCirsd4hGQDLMf", created_at: Some(Timestamp { seconds: 1726536963, nanos: 388874000 }), updated_at: Some(Timestamp { seconds: 1726536963, nanos: 388885000 }) }), content: Some(Content { text: "cgCedOYkEmXuqev8", images: [], videos: [] }), created_at: Some(Timestamp { seconds: 1726536963, nanos: 388901000 }), updated_at: Some(Timestamp { seconds: 1726536963, nanos: 388902000 }) }]
```


**Thanks for your reading!, Hope this can help you learn how to implement stream endpoint in Tonic.**
