---
slug: how-full-text-index-works
title: How a Full Text Search engine works - Index
authors: [forfd8960]
tags: [fulltext, search]
---

Recently I am start working on build a Full Text Search engine with rust. But I don't know how to get started.
And have many questions:

- What is the basic concept in the full-text search Engine.
- What are the core conponents in the Engine.
- And how they works together.
- How the document/text is analyzed and filtered,
- How the analyzed indexed is stored on the disk.
- How the engine searchs the results according user's query.

After Read about Elastic Search Docuemnts, Melisearch blogs, Couchbase blog, and read some source code from Bleve.

- [How to deliver the best search results: inside a full text search engine](https://blog.meilisearch.com/how-full-text-search-engines-work/)

* [Text Analysis within a Full-Text Search Engine](https://www.couchbase.com/blog/full-text_search_text_analysis/)

- [Anatomy of an analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html)
- [Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)

* [Bleve](https://github.com/blevesearch/bleve)

I have some basic knowledge about full-text search engines.

In this blog post Will first write about the Basic Concept and the Index Steps when index the document.

## The Basic Concept.

### What is Full Text Search

Full-text search refers to **the process of searching a database or collection of text documents for all instances of a word or phrase**. It is a comprehensive way to query text content, as opposed to searching metadata or parts of the original texts. Full-text search is widely used in search engines, document management systems, and databases to quickly find information within large volumes of text data.

**`The core functionality of full-text search allows users to retrieve documents or database records that contain specific words or phrases, regardless of their position within the text`**. This is achieved by indexing the content of the documents, where the index stores the location of words within the documents. When a search query is performed, the search engine looks up the query terms in the index and retrieves the documents that contain those terms.

### The Core Conpoenets in a Full Text Search Engine

Full-text search engines are sophisticated systems designed to efficiently search through large volumes of text data to find matches to user queries. These engines are crucial for navigating the vast amounts of unstructured data that exist today. The components and algorithms behind full-text search engines enable them to provide relevant, accurate search results quickly. Here's an overview of the key components and some of the algorithms that power them:

#### Components of a Full-Text Search Engine

![Components](fts_engine.png)

1. **Indexing**: Before a search engine can start searching through documents, it needs to process and organize the data. This is done through indexing. During indexing, the engine scans the text of documents and creates a list of search terms, often called an index or concordance. This index is a data structure that allows for fast searching.

2. **Text Analysis**: This component processes the text to be indexed or searched. It involves several sub-tasks such as tokenization (breaking text into words or terms), stemming (reducing words to their base or root form), and removing stop words (common words like "the" and "and" that are ignored during searching).

3. **Query Processor**: The query processor interprets the user's search query and converts it into a form that can be efficiently executed by the search engine. It handles syntax parsing, spell correction, synonym expansion, and other query enhancements.

4. **Search Algorithm**: Once the query is processed, the search algorithm takes over to find matches in the indexed data. It uses the index to quickly locate documents that contain the terms mentioned in the query.

5. **Ranking Algorithm**: After finding the matching documents, the search engine needs to rank them according to their relevance to the query. This is where ranking algorithms come into play, evaluating factors like term frequency, document length, and term proximity.

#### Algorithms Behind Full-Text Search Engines

1. **Boolean Retrieval Model**: This is one of the simplest models, where the search query is expressed using Boolean operators (AND, OR, NOT). It's effective for precise queries but lacks the nuance of ranking results by relevance.

2. **Vector Space Model (VSM)**: In VSM, documents and queries are represented as vectors in a multi-dimensional space. The relevance of a document to a query is determined by the cosine similarity between their vectors. This model allows for ranking results based on how closely they match the query.

3. **TF-IDF (Term Frequency-Inverse Document Frequency)**: This is a statistical measure used to evaluate how important a word is to a document in a collection or corpus. It's often used in conjunction with VSM to calculate the weight of terms in documents and queries.

4. **PageRank**: Developed by Google, PageRank is a link analysis algorithm used to rank web pages in search engine results. While not a full-text search algorithm per se, it's an important component of web search engines, determining the importance of documents based on the link structure of the web.

5. **BM25**: A more advanced ranking function that builds upon the probabilistic retrieval model. It considers term frequency, document length, and other factors to rank documents. BM25 is known for its effectiveness in handling the variability of term importance across different documents.

In summary, full-text search engines are complex systems that rely on a combination of text processing, indexing, and sophisticated algorithms to provide fast and relevant search results. The components work together to handle the challenges of searching through vast amounts of unstructured text data, while the algorithms ensure that the results are ranked by relevance to the user's query.

## Step 1: Index

The indexing process will scan the document, and apply the Analyzers, Tokenize, Filters on the Text.
And finnaly produce the index.

![Index Process](indexing_process.png)

### Index Example with Bleve

```go
messages := []struct {
		Id   string
		From string
		Body string
	}{
		{
			Id:   "index-example1",
			From: "forfd8960",
			Body: "test with bleve indexing",
		},
		{
			Id:   "index-example2",
			From: "forfd8960",
			Body: "another document",
		},
	}

	mapping := bleve.NewIndexMapping()
	index, err := bleve.New("example.bleve", mapping)
	if err != nil {
		panic(err)
	}

	for _, msg := range messages {
		if err := index.Index(msg.Id, msg); err != nil {
			fmt.Printf("index err: %v\n", err)
			return
		}
	}

	fmt.Printf("index: %v has been successful build\n", index)
```
