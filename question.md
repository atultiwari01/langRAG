# Questions and Answers

This file stores short Q&A for revision, interview prep, and concept checks.

## Format

Use this format for future additions:

### Question

Answer

---

## Current Q&A

### Why does an LLM need RAG?

An LLM needs RAG to answer with relevant external or private data instead of relying only on pretrained memory. This helps ground responses, improve factual accuracy, and reduce hallucinations.

### What are the 3 main parts of a RAG pipeline?

1. Indexing
2. Retrieval
3. Generation

### Why is RAG often better than fine-tuning for document question answering?

RAG is often better because it can use updated documents without retraining and is usually cheaper and more practical for factual document lookup.

### Give one real-world product that could use RAG.

An internal company knowledge assistant that answers questions from private documents, policies, and team knowledge bases.

### Can RAG help reduce data exposure risk?

Yes, if it runs locally or in a controlled environment. But the protection comes from system design, access control, storage policy, and provider data-handling guarantees, not from RAG alone.
