# RAG Learning Plan

This plan is based on the current workspace:

- `study-map.md` shows fundamentals are done and retrieval is the current focus.
- `notes.md` already covers the RAG pipeline, sparse retrieval, dense retrieval, and hybrid retrieval at a concept level.
- `question.md` has a small revision set, but needs more retrieval and implementation questions.
- `documents/company_policy_handbook.md` is the reusable source document for local chunking and retrieval experiments.
- `Practice/rag_local_01_retrival.ipynb` contains a first local retrieval prototype using chunking, embeddings, Chroma, and Ollama.
- `Practice/rag_local_02_chunking_experiments.ipynb` loads the handbook from disk and compares chunking settings.
- `rag-from-scratch/` has reference notebooks for overview, query transformations, routing, indexing, reranking, CRAG, and long-context generation.
- `ResearchPaper/` contains the right papers for later deep dives into DPR, RAG, REALM, Atlas, ColBERT, RAPTOR, Self-RAG, and long-context failure modes.

## Current Level

You are past the beginner definition stage. The next growth step is to become strong at diagnosing and improving retrieval quality.

Current focus:

- sparse retrieval
- dense retrieval
- hybrid retrieval
- chunking and embeddings as they affect retrieval
- turning retrieval results into grounded answers

## Step 1: Clean Retrieval Fundamentals

Goal: understand retrieval as a search problem, not just a LangChain method call.

Do this:

1. Revise these sections in `notes.md`:
   - What retrieval is
   - Sparse retrieval
   - Dense retrieval
   - Sparse vs dense retrieval
   - Hybrid retrieval
2. Add 8 to 10 questions to `question.md`:
   - Why can dense retrieval fail?
   - When is sparse retrieval better?
   - Why is hybrid retrieval useful?
   - What does `k` control in retrieval?
   - What happens when the wrong chunks are retrieved?
3. In `Practice/rag_local_01_retrival.ipynb`, run the same questions with different `k` values:
   - `k=1`
   - `k=2`
   - `k=4`

Checkpoint:

- You should be able to explain why a retrieved chunk was good or bad.

## Step 2: Chunking Experiments

Goal: learn how chunk size and overlap change retrieval quality.

Do this in:

- `Practice/rag_local_02_chunking_experiments.ipynb`

Experiments:

1. Use `documents/company_policy_handbook.md`.
2. Compare chunk sizes:
   - 100
   - 200
   - 500
3. Compare overlaps:
   - 0
   - 30
   - 80
4. For each setting, ask the same 5 queries.
5. Record:
   - retrieved chunk text
   - whether it contains the answer
   - whether it includes extra noise

Checkpoint:

- You should know why smaller chunks improve precision but can lose context.
- You should know why larger chunks preserve context but can add noise.

## Step 3: Build a Retrieval Evaluation Habit

Goal: stop judging RAG only by final answer quality.

Do this:

1. Create 10 test questions for your policy documents.
2. For each question, write the expected source sentence or chunk.
3. Measure retrieval manually:
   - hit or miss
   - rank of correct chunk
   - whether top-1 was enough
4. Add simple metrics:
   - recall@1
   - recall@2
   - recall@4

Suggested notebook:

- `Practice/rag_local_03_retrieval_eval.ipynb`

Checkpoint:

- You should be able to say: "My retriever fails because of wording, chunking, missing metadata, or embedding weakness."

## Step 4: Add Generation Carefully

Goal: understand that generation quality depends on retrieval quality.

Use:

- `rag-from-scratch/rag_from_scratch_1_to_4.ipynb`
- your current `Practice/rag_local_01_retrival.ipynb`

Do this:

1. Build a prompt that uses only retrieved context.
2. Ask the model to say "I don't know from the provided context" when context is missing.
3. Test with:
   - answerable questions
   - unanswerable questions
   - typo-heavy questions
   - questions with misleading terms
4. Compare final answers against retrieved chunks.

Checkpoint:

- You should be able to separate retrieval failure from generation failure.

## Step 5: Hybrid Search

Goal: combine exact keyword search and semantic search.

Do this:

1. Implement a simple keyword retriever with BM25 or TF-IDF.
2. Keep the existing dense retriever with Chroma.
3. Run both retrievers for the same query.
4. Merge results and remove duplicates.
5. Compare:
   - sparse only
   - dense only
   - hybrid

Suggested notebook:

- `Practice/rag_local_04_hybrid_retrieval.ipynb`

Checkpoint:

- You should know when exact words beat embeddings and when embeddings beat exact words.

## Step 6: Query Transformation

Goal: learn how better queries improve retrieval.

Use:

- `rag-from-scratch/rag_from_scratch_5_to_9.ipynb`

Study in this order:

1. Multi-query retrieval
2. RAG-Fusion
3. Decomposition
4. Step-back prompting
5. HyDE

Practice:

- Take 5 weak queries from your retrieval notebook.
- Rewrite each query manually.
- Then test multi-query retrieval.

Checkpoint:

- You should understand that query transformation is useful when the user question is vague, incomplete, or phrased differently from the documents.

## Step 7: Metadata Filtering and Routing

Goal: make retrieval more controlled.

Use:

- `rag-from-scratch/rag_from_scratch_10_and_11.ipynb`

Do this:

1. Add metadata to your sample documents:
   - department
   - policy type
   - employee type
   - source
2. Filter retrieval by metadata.
3. Route questions to different document groups.

Checkpoint:

- You should know why metadata filtering is often more reliable than hoping embeddings understand everything.

## Step 8: Reranking

Goal: improve result order after initial retrieval.

Use:

- `rag-from-scratch/rag_from_scratch_15_to_18.ipynb`
- `ResearchPaper/ColBERT.pdf`

Do this:

1. Retrieve more chunks than needed, such as top 8 or top 10.
2. Rerank them.
3. Pass only the best 2 to 4 chunks to the LLM.
4. Compare answers before and after reranking.

Checkpoint:

- You should understand the retrieve-wide, rerank-narrow pattern.

## Step 9: Advanced Indexing

Goal: learn when basic chunk embedding is not enough.

Use:

- `rag-from-scratch/rag_from_scratch_12_to_14.ipynb`
- `ResearchPaper/RAPTOR.pdf`
- `ResearchPaper/ColBERT.pdf`

Study:

1. Multi-representation indexing
2. RAPTOR
3. ColBERT

Checkpoint:

- You should know that indexing design changes what the retriever can find.

## Step 10: Read Papers in the Right Order

Goal: connect practical RAG to research ideas without drowning in papers.

Recommended order:

1. `Retrieval_Augmented_Generation_for_Knowledge_Intensive_NLP_Tasks.pdf`
2. `Dense Passage Retrieval for Open-Domain Question Answering.pdf`
3. `REALM.pdf`
4. `Leveraging Passage Retrieval with Generative Models.pdf`
5. `Lost in the Middle.pdf`
6. `ColBERT.pdf`
7. `RAPTOR.pdf`
8. `SELF-RAG.pdf`
9. `Atlas.pdf`
10. `APPROXIMATE NEAREST NEIGHBOR NEGATIVE CONTRASTIVE LEARNING FOR DENSE TEXT RETRIEVAL.pdf`

For each paper, write only:

- the problem it solves
- the main idea
- how it affects RAG design
- one practical lesson

## Step 11: Build a Small Real Project

Goal: move from notebook learning to a usable RAG system.

Project idea:

- Local company-policy assistant

Features:

1. Load documents from a folder.
2. Chunk and embed them.
3. Store them in Chroma.
4. Retrieve with dense search.
5. Add keyword or hybrid search.
6. Generate grounded answers with citations.
7. Show retrieved chunks for debugging.
8. Add a small evaluation set.

Checkpoint:

- You should be able to debug an answer by checking the query, retrieved chunks, prompt, and final response.

## Weekly Rhythm

Use this cycle for every topic:

1. Read the concept in `notes.md`.
2. Run or create a practice notebook.
3. Break it with bad queries.
4. Record what failed.
5. Add 5 Q&A items to `question.md`.
6. Update `study-map.md` status.

## Immediate Next Actions

Do these next:

1. Finish hybrid retrieval concept revision.
2. Run `Practice/rag_local_02_chunking_experiments.ipynb`.
3. Add retrieval evaluation questions to `question.md`.
4. Compare `k=1`, `k=2`, and `k=4` in the current retrieval notebook.
5. Start a simple retrieval evaluation table.
