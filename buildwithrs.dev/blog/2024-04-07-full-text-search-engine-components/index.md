---
slug: full-text-search-engine-components
title: FullText Search Engine Componenets
authors: [forfd8960]
tags: [fulltext, search, engine]
---

## What's the Core Components in a Full Text Search Engine

Look into Bleve Modern Search Lib Code, and Found that it has the following core components：

| component | function of the component                                               |     |
| --------- | ----------------------------------------------------------------------- | --- |
| analysis  | mainly contains analyzer(char filters, token filters, tokenizer)        |     |
| cmd       | command line client                                                     |     |
| document  | define Document and Related Doc Fields(Text Field, Datetime Field, etc) |     |
| geo       | Implement geo point and geo shape query                                 |     |
| index     | Analyze and read, write indexed fields to under key-value store         |     |
| mapping   | implment DocumentMapping: How the Document should be indexed.           |     |
| search    | Implment search functionality                                           |     |

## List of pkg

```sh
❯ tree . -R -L 1
.
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
├── analysis
├── builder.go
├── builder_test.go
├── cmd
├── config
├── config.go
├── config_app.go
├── config_disk.go
├── data
├── doc.go
├── docs
├── document
├── error.go
├── examples_test.go
├── geo
├── go.mod
├── go.sum
├── http
├── index
├── index.go
├── index_alias.go
├── index_alias_impl.go
├── index_alias_impl_test.go
├── index_impl.go
├── index_meta.go
├── index_meta_test.go
├── index_stats.go
├── index_test.go
├── mapping
├── mapping.go
├── mapping_vector.go
├── numeric
├── pre_search.go
├── query.go
├── query_bench_test.go
├── registry
├── search
├── search.go
├── search_knn.go
├── search_knn_test.go
├── search_no_knn.go
├── search_test.go
├── size
├── test
└── util
```
