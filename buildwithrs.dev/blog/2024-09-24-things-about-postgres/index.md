---
slug: things-about-postgres
title: learn about postgres
authors: forfd8960
tags: [postgres]
---

## Check users

```sh
psql -U postgres
psql (14.12 (Homebrew))
Type "help" for help.

postgres=> \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  |                                                            | {}
 superadmin | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

postgres=>

```

## Create User

```sql

> psql -d story_collection
psql (14.12 (Homebrew))
Type "help" for help.

story_collection=# \du
                                    List of roles
 Role name  |                         Attributes                         | Member of
------------+------------------------------------------------------------+-----------
 db_manager | Create DB                                                  | {}
 postgres   |                                                            | {}
 super_admin  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
​
story_collection=# \conninfo
You are connected to database "story_collection" as user "super_admin" via socket in "/tmp" at port "5432".
story_collection=# SELECT SESSION_USER;
 session_user
--------------
 super_admin

story_collection=# CREATE USER db_manager WITH CREATEDB PASSWORD 'xxx';
CREATE ROLE
story_collection=#

story_collection=# ALTER DATABASE story_collection OWNER TO db_manager;
ALTER DATABASE
```

## Connect DB with user `db_manager`

```sh
✗ psql -U db_manager -d story_collection
psql (14.12 (Homebrew))
Type "help" for help.

story_collection=> \conninfo
You are connected to database "story_collection" as user "db_manager" via socket in "/tmp" at port "5432".
story_collection=>
```
