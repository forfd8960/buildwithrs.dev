---
sidebar_position: 3
---

# The Todo Storage: `src/model/mod.rs`

## Todo File

```rust
const TODO_FILE: &str = "todos.txt";
```

## Todo struct

```rust
#[derive(Debug, PartialEq, Clone)]
pub struct Todo {
    pub id: u64,
    pub name: String,
    pub desc: String,
    pub status: Status,
    pub finish_date: String,
}
```

## Todo Status

```rust
#[derive(Debug, PartialEq, Clone, Copy)]
pub enum Status {
    PLAN,
    TODO,
    WIP,
    DONE,
}
```

## Parse string into Status

```rust
impl FromStr for Status {
    type Err = String;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s {
            "PLAN" => Ok(Status::PLAN),
            "TODO" => Ok(Status::TODO),
            "WIP" => Ok(Status::WIP),
            "DONE" => Ok(Status::DONE),
            _ => Err(format!("{} is not valid status", s)),
        }
    }
}
```

## Todo Store

```rust
pub struct TodoStore {
    max_id: u64, // the current max_id of all todos
    todos: Vec<Todo>,
}

impl TodoStore {
    pub fn new(max_id: u64, todos: Vec<Todo>) -> Self {
        TodoStore { max_id, todos }
    }

    pub fn load() -> Result<Self, String> {
        let todos = load_todos()?;
        let max_id = todos.iter().map(|t| t.id).max().unwrap_or(0);
        Ok(TodoStore::new(max_id, todos))
    }

    pub fn create(&mut self, todo: &Todo) -> Result<Todo, String> {
        let new_todo = gen_todo(self.max_id, todo)?;
        self.todos.push(new_todo.clone());
        self.max_id += 1;
        Ok(new_todo)
    }

    pub fn delete(&mut self, todo_id: u64) -> Result<(), String> {
        println!("delete todo: {}", todo_id);
        self.todos.retain(|t| t.id != todo_id);
        Ok(())
    }

    pub fn update_status(&mut self, todo_id: u64, target_status: Status) -> Result<(), String> {
        println!("move todo: {} to: {:?}", todo_id, target_status);

        self.todos.iter_mut().for_each(|t| {
            if t.id == todo_id {
                t.status = target_status;
                println!("updated todo: {:?}", t);
            }
        });
        Ok(())
    }

    pub fn list(&self) -> Result<Vec<Todo>, String> {
        Ok(self.todos.clone())
    }

    pub fn save(&self) -> Result<(), String> {
        let content = self
            .todos
            .iter()
            .map(|t| encode_todo(t))
            .collect::<Vec<String>>()
            .join("\n");

        fs::write(TODO_FILE, content).map_err(|e| e.to_string())?;
        Ok(())
    }
}
```

### Encode Todo

```rust
pub fn encode_todo(todo: &Todo) -> String {
    let bottom_line = "=".repeat(30);
    format!(
        "id: {}\nname: {}\ndesc: {}\nstatus: {:?}\nfinish_date: {}\n{}",
        todo.id, todo.name, todo.desc, todo.status, todo.finish_date, bottom_line
    )
}
```

### Decode todo

```rust
pub fn decode_todo(todo_str: &str) -> Result<Todo, String> {
    println!("decode todo str: {}", todo_str);

    let trimmed = todo_str.trim();

    println!("trimmed todo: {}", trimmed);

    let mut lines = trimmed.lines();
    let id = lines
        .next()
        .and_then(|l| l.split(": ").nth(1))
        .and_then(|id| id.parse::<u64>().ok())
        .unwrap_or(0);

    let name = lines
        .next()
        .and_then(|l| l.split(": ").nth(1))
        .unwrap_or("");

    let desc = lines
        .next()
        .and_then(|l| l.split(": ").nth(1))
        .unwrap_or("");

    let status = lines
        .next()
        .and_then(|l| l.split(": ").nth(1))
        .and_then(|s| Some(s.parse().unwrap_or(Status::PLAN)))
        .unwrap_or(Status::PLAN);

    let finish_date = lines
        .next()
        .and_then(|l| l.split(": ").nth(1))
        .unwrap_or("");

    Ok(Todo {
        id,
        name: name.to_string(),
        desc: desc.to_string(),
        status,
        finish_date: finish_date.to_string(),
    })
}
```

### Load todos from file

```rust


pub fn load_todos() -> Result<Vec<Todo>, String> {
    let content = std::fs::read_to_string(TODO_FILE).map_err(|e| e.to_string())?;
    let splitter = "=".repeat(30);
    let encode_todos: Vec<&str> = content.split(&splitter).collect();

    let todos = encode_todos
        .iter()
        .filter(|t| !t.is_empty())
        .map(|t| {
            decode_todo(t).unwrap_or_else(|e| {
                eprintln!("decode todo error: {}", e);
                Todo {
                    id: 0,
                    name: "".to_string(),
                    desc: "".to_string(),
                    status: Status::PLAN,
                    finish_date: "".to_string(),
                }
            })
        })
        .collect();
    Ok(todos)
}
```
