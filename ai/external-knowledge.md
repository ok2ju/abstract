# Patterns for Giving AI Agents External Knowledge

Agentic AI systems — Claude Code, Cursor, Aider, LangGraph, CrewAI, and others — all run into the same wall: an LLM's context window is finite, but the knowledge it needs often isn't. Three architectural patterns address this, differing in _who_ performs the retrieval.

## Filesystem-based knowledge
Knowledge lives as plain text or markdown files (e.g. an Obsidian vault, a docs folder). The agent retrieves on its own using file tools — read, grep, glob — and follows links between documents. Simple and transparent, but doesn't scale cleanly past a few thousand files.

## RAG (retrieval-augmented generation)
An external pipeline pre-indexes the corpus and selects relevant chunks at query time, typically combining BM25 (lexical match), vector search (semantic similarity), and a re-ranker (precision). The agent receives a curated short list. Scales to massive corpora, but adds infrastructure and loses document-level context.

## Search subagent
A delegated agent instance handles the retrieval — using files, RAG, or both — and returns a synthesized summary to the main agent. Keeps the main context clean and allows reasoning during search, at the cost of extra tokens and latency.

The three patterns compose: a subagent running RAG over a markdown vault is a common production setup. The right choice depends on corpus size, how much reasoning the retrieval itself needs, and how much infrastructure you're willing to maintain.