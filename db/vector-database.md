# Vector Databases

A **vector database** is a database built to store and search data by *meaning* instead of by exact matches. It's a core building block for modern AI features like semantic search, recommendations, and chatbots.

## The core idea

A regular database (like SQL) finds things by exact match: you ask for the word "car" and it returns rows containing the word "car". It has no idea that "vehicle" or "automobile" mean almost the same thing.

A vector database fixes this by turning data into **vectors** — long lists of numbers that capture the *meaning* of the original content. Similar things end up with similar numbers, so the database can find results that are *related*, not just identical.

## Vector embeddings

A **vector embedding** is the numerical fingerprint of a piece of data. An AI model reads some input — text, an image, audio — and outputs a list of numbers (the vector) describing it.

The key property: things with similar meaning get similar vectors. So "car" and "vehicle" land close together, while "car" and "banana" land far apart. This lets the computer compare meaning using math.

## How it works

1. **Store** — Each piece of data is converted into a vector ahead of time and saved (along with metadata like the original text or a link to it).
2. **Index** — The database organizes vectors so it doesn't have to compare your query against every single one. It uses *Approximate Nearest Neighbor (ANN)* algorithms (e.g. **HNSW**, **LSH**) that trade a tiny bit of accuracy for huge speed gains.
3. **Search** — When you ask a question, it's turned into a query vector. The database finds the vectors *closest* to it and returns those results.

**Closeness** is measured with a distance metric, such as:
- **Cosine similarity** — compares the direction of two vectors (most common for text)
- **Euclidean distance** — straight-line distance between two points
- **Dot product** — another way to score how aligned two vectors are

## Why use one

- **Understands meaning** — finds related results, not just exact keyword matches
- **Fast** — searches millions of items in milliseconds
- **Scalable** — keeps performing as data grows
- **Flexible** — works with any unstructured data: text, images, video, audio

## Common use cases

- **RAG (Retrieval-Augmented Generation)** — feeding relevant context to an LLM so it answers accurately
- **Semantic search** — search by meaning instead of keywords
- **Recommendations** — "items similar to this" for shopping or streaming
- **Anomaly & fraud detection** — spotting data points that don't fit the pattern

## Vector vs. traditional databases

| | Traditional (SQL) | Vector |
|---|---|---|
| Best for | Structured rows & columns | Unstructured data (text, images, audio) |
| Finds results by | Exact match (keywords, tags) | Similarity of meaning |
| Returns | Exact hits | Closest / most relevant matches |

In short: traditional databases answer *"find the rows that exactly match this"*, while vector databases answer *"find the things most similar in meaning to this"*.

## Sources

- [What is a vector database? — IBM](https://www.ibm.com/think/topics/vector-database)
