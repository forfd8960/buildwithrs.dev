---
slug: how-to-use-speckit-to-vibe-coding-a-rust-project
title: How to use Speckit to Vibe Coding a Rust Project
authors: forfd8960
tags: [speckit, vibe-coding, rust]
---

In this tutorial, we will explore how to use Speckit, a powerful tool for managing Rust projects, to vibe coding a Rust project. Speckit simplifies the process of setting up and maintaining Rust projects by providing a structured approach to project management.

## What is Speckit?

> An open source toolkit that allows you to focus on product scenarios and predictable outcomes instead of vibe coding every piece from scratch.

> Spec-Driven Development flips the script on traditional software development. For decades, code has been king — specifications were just scaffolding we built and discarded once the "real work" of coding began. Spec-Driven Development changes this: specifications become executable, directly generating working implementations rather than just guiding them.

<!-- truncate -->

## install speckit

```sh
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

## Init speckit under the project directory

```sh
db-query-rs> specify init --here --ai copilot
```

```sh
Warning: Current directory is not empty (1 items)
Template files will be merged with existing content and may overwrite existing files
Do you want to continue? [y/N]: y
╭──────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                      │
│  Specify Project Setup                                                                               │
│                                                                                                      │
│  Project         db-query-rs                                                                         │
│  Working Path    /Users/alexz/Documents/Code/Github/rust-vibe-coding/db-query-rs                     │
│                                                                                                      │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────╯

Selected AI assistant: copilot
Selected script type: sh
Initialize Specify Project
├── ● Check required tools (ok)
├── ● Select AI assistant (copilot)
├── ● Select script type (sh)
├── ● Fetch latest release (release v0.0.90 (59,640 bytes))
├── ● Download template (spec-kit-template-copilot-sh-v0.0.90.zip)
├── ● Extract template
├── ● Archive contents (39 entries)
├── ● Extraction summary (temp 3 items)
├── ● Ensure scripts executable (5 updated)
├── ● Cleanup
├── ● Initialize git repository (initialized)
└── ● Finalize (project ready)

Project ready.

╭─────────────────────────────────────── Agent Folder Security ────────────────────────────────────────╮
│                                                                                                      │
│  Some agents may store credentials, auth tokens, or other identifying and private artifacts in the   │
│  agent folder within your project.                                                                   │
│  Consider adding .github/ (or parts of it) to .gitignore to prevent accidental credential leakage.   │
│                                                                                                      │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────╯

╭───────────────────────────────────────────── Next Steps ─────────────────────────────────────────────╮
│                                                                                                      │
│  1. You're already in the project directory!                                                         │
│  2. Start using slash commands with your AI agent:                                                   │
│     2.1 /speckit.constitution - Establish project principles                                         │
│     2.2 /speckit.specify - Create baseline specification                                             │
│     2.3 /speckit.plan - Create implementation plan                                                   │
│     2.4 /speckit.tasks - Generate actionable tasks                                                   │
│     2.5 /speckit.implement - Execute implementation                                                  │
│                                                                                                      │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────╯

╭──────────────────────────────────────── Enhancement Commands ────────────────────────────────────────╮
│                                                                                                      │
│  Optional commands that you can use for your specs (improve quality & confidence)                    │
│                                                                                                      │
│  ○ /speckit.clarify (optional) - Ask structured questions to de-risk ambiguous areas before          │
│  planning (run before /speckit.plan if used)                                                         │
│  ○ /speckit.analyze (optional) - Cross-artifact consistency & alignment report (after                │
│  /speckit.tasks, before /speckit.implement)                                                          │
│  ○ /speckit.checklist (optional) - Generate quality checklists to validate requirements              │
│  completeness, clarity, and consistency (after /speckit.plan)  
```

## Build the general project idea

under the specs/instruction.md file, write the `Constitution`

```md
## Constitution

- 后端使用 Ergonomic Rust 风格来编写本项目的代码，确保代码可读性和一致性。
- 所有后端生成的数据，使用 camelCase 格式。
- 不需要 authentication, 任何用户都可以使用。
```

and also write the basic project idea

```md
## 基本思路

这是一个数据库查询工具，用户可以添加一个 DB URL，然后系统会连接到数据库，获取数据库的 metadata, 然后数据库中的 table, view 的信息会展示在左侧的树形结构中，用户可以点击某个 table/view, 然后右侧会展示该 table/view 的 columns 信息，以及允许用户输入 SQL 查询语句，执行查询后，结果会展示在下方的表格中。
同时用户也可以通过自然语言输入查询需求，系统会将自然语言转换为 SQL 语句，然后执行查询并展示结果。
```

and also the basic project functions:

```md
## 基本想法

- 数据库连接 URL 和数据库的 metadata 都会存储到 sqlite 中. 我们可以根据 postgres 的功能来查询系统中的表和视图的信息，然后用 LLM 来讲这些信息转换成 json 格式，然后存储到 sqlite 中.这个信息以后可以复用。

- 当用户通过 LLM 来生成 sql 查询时，我们可以把系统中的表和视图的信息，以及 columns 信息，传递给 LLM, 让 LLM 帮助生成 SQL 查询语句.

- 任何输入的 sql 语句都需要经过 sqlparse 来解析，确保语法正确，并且仅包含 select 语句，如果语法不正确，需要给出错误信息。
  - 如何查询不包含 Limit 子句， 则默认添加 limit 1000 子句.
- 输出格式是 json, 前端将其组织成表格，并显示出来。

- 使用 Rust / Axum / SQLx / openai_api_rust sdk 来实现
- OpenAI 的 API key 通过环境变量(OPENAI_API_KEY)传递给后端.
- 数据库连接 URL 和 metadata 存储在 sqlite 中, 放在 ./db-query/db_query.db 中.
- 后端 API 需要支持 CORS, 允许所有 origin.
```

and the API list

```sh
  POST /api/v1/databases/  - 添加一个新的数据库连接 URL

  {"url": "postgresql://user:pass@host:port/dbname"}

  GET /api/v1/databases/  - 获取所有的数据库连接 URL 列表

  PUT /api/v1/databases/{db_name}/  - 更新指定的数据库连接 URL

  {"url": "postgresql://user:pass@host:port/dbname"}

  GET /api/v1/databases/{db_name}/metadata/  - 获取指定数据库的 metadata (tables, views, columns)

  POST /api/v1/databases/{db_name}/query/  - 执行 SQL 查询

  {"sql": "SELECT * FROM table_name LIMIT 10"}

  POST /api/v1/databases/{db_name}/nl-query/  - 执行自然语言查询

  {"nl_query": "Get me the first 10 rows from table_name"}
```

![alt text](speckit_commands.png)

## Generate Constitution based on the ``specs/instructions.md#constitution` 

```sh
/speckit.constitution update the project constitution based on the provided instructions and context
```

## Generate the specification for the DB-Query-RS project based on the constitution and context

```sh
/speckit.specify create a feature specification for the database query tool based on instructions's context basic ideas.
```


## Generate project implementation plan based on the specification

```sh
/speckit.plan create a project implementation plan for the database query tool based on the specification
```

## Generate the implementation tasks

```sh
/speckit.tasks this project is simple, pls split the tasks phases into no more than 3 phases.
```

```md
Task Generation Complete
Generated File
Location: tasks.md

Task Organization (Simplified 3-Phase Structure)
As requested, tasks are consolidated into 3 phases instead of the standard multi-phase structure:

Phase 1: Setup & Foundation (12 tasks, 1-2 hours)
Project initialization (Cargo.toml, directory structure)
SQLite schema creation
Core infrastructure (config, errors, DTOs, CORS)
Server setup with routing structure
Phase 2: Core Features (22 tasks, 6-8 hours)
Combines US1 (P1), US2 (P2), US3 (P3) - the essential MVP features:

User Story 1 - Database Connections (8 tasks):

DatabaseConnection model
Add/list/update database services
API handlers for CRUD operations
Integration tests
User Story 2 - Metadata Browsing (7 tasks):

TableMetadata and ColumnMetadata models
Fetch from PostgreSQL, cache to SQLite
Metadata retrieval with caching logic
API handler and tests
User Story 3 - Query Execution (7 tasks):

QueryResult model
SQL validator (sqlparser-rs integration)
LIMIT injection logic
Query execution service
API handler and comprehensive tests
Phase 3: Advanced Features (8 tasks, 3-4 hours)
User Story 4 - Natural Language Query (P4):

NaturalLanguageRequest model
OpenAI client setup
Prompt engineering with schema context
SQL generation from natural language
Validation and execution pipeline
Error handling and tests
Task Summary
Total Tasks: 42
Parallel Tasks: 15 marked with [P]
Estimated Duration: 10-14 hours total
MVP Milestone: After Phase 2 (34 tasks)
Key Features
✅ All tasks follow strict format: - [ ] [ID] [P?] [Story?] Description with file path

Example: - [ ] T013 [P] [US1] Create DatabaseConnection model in src/models/database.rs with validation
✅ Task Dependencies Documented:

Sequential critical paths defined for each phase
Parallel execution opportunities identified
User story dependencies visualized
✅ Validation Checklists:

Phase 1: Server runs, SQLite initialized, CORS active
Phase 2: All CRUD operations work, metadata cached, queries validated
Phase 3: Natural language translation functional
✅ Implementation Strategy:

MVP scope clearly defined (Phases 1-2)
Incremental delivery milestones
Team size recommendations (1-3 developers)
File Paths Referenced
All tasks include exact file paths:

src/models/*.rs - Data models
src/services/*.rs - Business logic
src/api/handlers/*.rs - HTTP handlers
src/storage/*.rs - SQLite operations
src/validation/*.rs - SQL validation
tests/integration/*.rs - Integration tests
tests/unit/*.rs - Unit tests
migrations/*.sql - Schema definitions
Independent Testing Criteria
Each user story includes "Independent Test" description:

US1: Add/list/update databases, verify SQLite persistence
US2: Request metadata, verify caching and camelCase
US3: Execute queries, verify validation and LIMIT injection
US4: Submit natural language, verify SQL generation and execution
```

## Implement the phase1 tasks

```sh
/speckit.implement finish the development of phase1 tasks
```

## Implement the phase2 tasks

```sh
/speckit.implement finish the development of phase2 tasks
```

## Implement the phase3 tasks

```sh
/speckit.implement finish the development of phase3 tasks
```

## Final Result 

```sh
db-query-rs (001-db-query-tool)> tree src
src
├── api
│   ├── dto.rs
│   ├── handlers
│   │   ├── database_handlers.rs
│   │   ├── metadata_handlers.rs
│   │   ├── mod.rs
│   │   └── query_handlers.rs
│   ├── mod.rs
│   └── routes.rs
├── config.rs
├── main.rs
├── models
│   ├── database.rs
│   ├── error.rs
│   ├── metadata.rs
│   ├── mod.rs
│   └── query.rs
├── services
│   ├── database_service.rs
│   ├── llm_service.rs
│   ├── metadata_service.rs
│   ├── mod.rs
│   └── query_service.rs
├── storage
│   ├── mod.rs
│   └── repository.rs
└── validation
    ├── mod.rs
    └── sql_validator.rs

7 directories, 23 files
```


```sh
➜  db-query-rs git:(001-db-query-tool) ✗ cargo run .

warning: `db-query-rs` (bin "db-query-rs") generated 7 warnings (run `cargo fix --bin "db-query-rs" -p db-query-rs` to apply 2 suggestions)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.33s
     Running `target/debug/db-query-rs .`
2025-12-19T06:45:11.297846Z  INFO db_query_rs: Starting db-query-rs server...
2025-12-19T06:45:11.303895Z  INFO db_query_rs: SQLite database initialized at ./db-query/db_query.db
2025-12-19T06:45:11.304228Z  INFO db_query_rs: Database health check passed
2025-12-19T06:45:11.305813Z  INFO db_query_rs: Server listening on 0.0.0.0:8000
```

