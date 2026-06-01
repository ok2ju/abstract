# Hybrid Search: BM25, Semantic Search, and RRF

**Search** is the problem of taking a query and deciding which documents to show first. Three terms come up constantly: **BM25**, **semantic search**, and **RRF**. They're three pieces of one story — BM25 and semantic search are *two ways to find results*, and RRF is the *glue that merges them*. Combining them is called **hybrid search**, and it beats either one alone.

## The problem search has to solve

Given a query like *"how do I fix a flat tire?"*, a search engine has to rank thousands of documents from most to least relevant. There are two fundamentally different ways to judge relevance:

- by the **exact words** the document contains (lexical / keyword search) → **BM25**
- by the **meaning** of the document, regardless of wording (semantic / vector search) → **semantic search**

Each approach is strong exactly where the other is weak, which is why production systems usually run both.

## BM25 — search by exact words

**BM25** ("Best Matching 25") is the classic keyword-search ranking algorithm, a refined version of the older TF-IDF. It scores a document higher when it literally contains the words you typed, based on three common-sense rules:

| Rule | Plain meaning | Example |
|---|---|---|
| **Term frequency** | A document that uses your word more is more relevant — but with diminishing returns. | The 10th "tire" matters less than the 2nd. |
| **Rare words count more** (inverse document frequency) | Matching a rare word beats matching a common one. | Matching "quantum" >> matching "the". |
| **Length normalization** | Long documents shouldn't rank highly just for being long (more words = more accidental matches). | A 50-page doc isn't more relevant because "tire" appears once. |

- **Great at:** exact terms, names, error codes, IDs, technical jargon. Fast, and needs no AI model or training.
- **Weak at:** meaning. Searching "car" will *not* match a document that only says "automobile" or "vehicle." This is the **vocabulary mismatch** problem.

## Semantic search — search by meaning

**Semantic search** converts your query and every document into **embeddings** — lists of numbers that capture meaning (see [Vector Databases](../db/vector-database.md)). It then returns the documents whose vectors are *closest* to the query's vector.

Because it works on meaning rather than spelling, searching *"how do I fix a flat tire?"* can match a document titled *"repairing a punctured wheel"* — even with zero shared words.

- **Great at:** synonyms, paraphrases, intent, fuzzy natural-language questions.
- **Weak at:** exact details. It can miss a specific product code, name, or rare keyword, and sometimes returns results that are "vibe-similar" but not what you wanted. Requires an embedding model and a vector database.

## Why combine them — hybrid search

The strengths and weaknesses of the two methods are mirror images:

| | BM25 (keywords) | Semantic (meaning) |
|---|---|---|
| Exact terms, codes, names | Excellent | Can miss |
| Synonyms & paraphrases | Misses | Excellent |
| Setup cost | Low | Higher (model + vectors) |

Where one fails, the other succeeds. So the natural move is to **run both searches and merge the results** — this is **hybrid search**, and it is consistently more reliable than either method alone.

The catch: **how do you merge two lists?** BM25 produces scores on one scale (e.g. 0–30) and semantic search on a completely different one (e.g. cosine similarity 0–1). You can't just add them — that's like adding Fahrenheit to km/h. This is exactly the problem RRF solves.

## RRF — Reciprocal Rank Fusion (the glue)

**Reciprocal Rank Fusion** merges multiple ranked lists into one using a clever shortcut: **ignore the raw scores and use only each document's *rank* (position)** in each list.

For each document, its RRF score is:

```
score = sum over each list of   1 / (k + rank)
```

where `rank` is the document's position in that list (1st, 2nd, 3rd…) and `k` is a constant (commonly **60**, the value from the original paper).

Why it works:

- `1 / rank` means **being near the top is worth a lot, and the value falls off quickly** further down the list.
- A document ranked high in *both* lists accumulates a large combined score.
- Using *position* instead of *score* sidesteps the "different scales" problem entirely — no normalization and no weight-tuning required.
- `k` dampens how sharply the top ranks dominate; larger `k` flattens the curve.

### Worked example

Two search engines return these rankings (k = 60):

- BM25 list: `[DocA, DocB, DocC]`
- Semantic list: `[DocB, DocD, DocA]`

| Doc | In BM25 | In Semantic | RRF score | |
|---|---|---|---|---|
| **DocB** | rank 2 → 1/62 | rank 1 → 1/61 | **0.0325** | winner |
| **DocA** | rank 1 → 1/61 | rank 3 → 1/63 | **0.0323** | runner-up |
| DocD | — | rank 2 → 1/62 | 0.0161 | |
| DocC | rank 3 → 1/63 | — | 0.0159 | |

**DocB wins** because it sits near the top of *both* lists. DocA is a close second (also in both). DocC and DocD trail because each appeared in only one list. RRF rewards **agreement** between the two search methods.

## How it all fits together

```
                  ┌───────────────┐
   your query ───▶│  BM25 search  │──▶ ranked list A (keywords)
              │   └───────────────┘                        │
              │   ┌───────────────┐                        ▼
              └──▶│Semantic search│──▶ ranked list B ──▶ RRF merge ──▶ final results
                  └───────────────┘    (meaning)
```

This **BM25 + semantic + RRF** recipe is the standard hybrid-search setup, and it's the retrieval backbone of modern **RAG** systems (see [Patterns for Giving AI Agents External Knowledge](../ai/external-knowledge.md)). An optional **re-ranker** can refine the top few results at the end for extra precision.

## In short

- **BM25** finds documents by *exact words* — precise, but blind to meaning.
- **Semantic search** finds documents by *meaning* — understands synonyms, but can miss exact terms.
- **RRF** merges the two ranked lists by *position*, not score — giving you the best of both.

## Related notes

- [Vector Databases](../db/vector-database.md) — the embeddings and similarity search that power semantic search.
- [Patterns for Giving AI Agents External Knowledge](../ai/external-knowledge.md) — where hybrid search fits into RAG.

## Sources

- [Okapi BM25 — Wikipedia](https://en.wikipedia.org/wiki/Okapi_BM25)
- Cormack, Clarke & Büttcher (2009), *Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods* (SIGIR) — the original RRF paper.
- [What is hybrid search? — Pinecone](https://www.pinecone.io/learn/hybrid-search-intro/)
