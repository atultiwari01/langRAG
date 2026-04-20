# Notes

This file stores concept notes, structured explanations, and references to practice work.

## How this file will be used

- Save topic-wise notes here.
- Keep explanations concise but useful for revision.
- Add links or references to notebooks inside `Practice/` when relevant.
- Prefer updating existing sections instead of creating duplicate notes.

## Practice Reference Convention

When a notebook is created in `Practice/`, add it under the relevant topic like this:

- Practice: `Practice/<notebook-name>.ipynb`
- Purpose: one-line summary of what was practiced

## Topics

### RAG

#### What is RAG

RAG stands for Retrieval-Augmented Generation. It improves an LLM by retrieving relevant external information and using that information during answer generation.

#### Why RAG is used

- gives access to private or domain-specific data
- helps ground answers in relevant context
- reduces hallucination compared to using only model memory
- is easier to update than retraining or fine-tuning for changing documents

#### Main pipeline

1. Indexing
2. Retrieval
3. Generation

Each stage has a different responsibility:

- Indexing prepares knowledge so it can be searched efficiently.
- Retrieval finds the most relevant pieces for the current query.
- Generation uses the retrieved context to produce the final answer.

Important practical idea:

- indexing affects searchability
- retrieval affects relevance
- generation affects answer quality

In many RAG systems, retrieval quality is the biggest bottleneck. If the wrong context is retrieved, the LLM usually cannot recover well.

#### Indexing includes

- loading data
- chunking documents
- creating embeddings
- storing vectors in a vector database

Key point:

- indexing is mostly an offline or preprocessing step
- bad chunking or poor document preparation can weaken the whole pipeline

#### Retrieval includes

- converting the user query into a searchable representation
- fetching relevant chunks
- optionally reranking or filtering them

Key point:

- retrieval decides what the model is allowed to see
- better retrieval usually improves groundedness more than prompt tweaks alone

#### What retrieval is

Retrieval is the search stage of RAG. It finds the most relevant information from the indexed knowledge base for the current user query and passes that information to the LLM.

In practice:

1. the user asks a question
2. the system converts the question into a searchable form
3. relevant chunks are retrieved from the knowledge base
4. those chunks are passed to the LLM for final answer generation

Important practical idea:

- retrieval is often the biggest bottleneck in a RAG system
- if retrieval returns the wrong chunks, even a strong LLM usually gives a weak answer

#### Sparse retrieval

Sparse retrieval is keyword-based retrieval. It matches the actual words or terms in the query with the words present in documents or chunks.

Common example:

- BM25

Why it is called sparse:

- the representation uses a very large vocabulary space
- only a small number of terms are present in a document or query
- most values are zero, so the representation is sparse

How it works:

- the query is broken into terms
- the retriever checks which chunks contain those terms
- chunks are scored based on term matching and relevance

Strengths:

- strong when exact wording matters
- good for product names, IDs, codes, laws, error messages, and exact technical terms
- interpretable and efficient

Weaknesses:

- weak when query wording differs from document wording
- does not understand semantic similarity well

Example:

- query: `refund policy for annual plan`
- sparse retrieval works well if the chunk contains similar exact words

Failure case:

- query: `how do I get my money back?`
- chunk: `customers may request reimbursement within 30 days`
- sparse retrieval may miss it because the wording is different

#### Dense retrieval

Dense retrieval is embedding-based retrieval. It converts queries and text chunks into dense vectors and retrieves chunks whose vectors are semantically similar to the query vector.

Why it is called dense:

- the vector representation contains many meaningful values instead of mostly zeros

How it works:

1. split documents into chunks
2. convert each chunk into an embedding
3. store embeddings in a vector database
4. convert the query into an embedding
5. compare the query embedding with stored chunk embeddings
6. retrieve the most similar chunks

Usually the vector database stores:

- the embedding vector
- the original chunk text
- metadata such as source, page, title, tags, or document id

Strengths:

- understands meaning, not only exact words
- handles paraphrased questions better
- useful when user wording differs from document wording

Weaknesses:

- may miss exact keyword matches
- depends heavily on embedding quality
- can retrieve related but not exact chunks

Example:

- query: `how do I get my money back?`
- chunk: `customers may request reimbursement within 30 days`
- dense retrieval can match this because the meaning is similar

#### Sparse vs dense retrieval

Sparse retrieval is like searching by exact words.

Dense retrieval is like searching by meaning.

When to prefer sparse retrieval:

- exact terms matter
- you need reliable matching for IDs, error codes, legal clauses, product names, or specific tokens

When to prefer dense retrieval:

- natural language questions are common
- users and documents may use different wording
- semantic meaning matters more than exact phrasing

Important caution:

- dense retrieval is not always better
- sparse retrieval can be more reliable when exact terms are critical

#### Hybrid retrieval

Hybrid retrieval combines sparse and dense retrieval.

Why it is useful:

- sparse retrieval is good for exact matches
- dense retrieval is good for semantic matches
- together they often perform better than either method alone

Industry note:

- many production RAG systems prefer hybrid retrieval because real queries need both exact and semantic matching

How it works in practice:

1. the user asks a query
2. sparse retrieval searches by keywords
3. dense retrieval searches by embeddings
4. results from both are combined
5. duplicates are removed
6. ranking or reranking selects the final chunks
7. the final chunks are passed to the LLM

Common ways to combine results:

- score fusion
- rank fusion
- reranking after merging candidate results

Important idea:

- sparse retrieval answers: which chunks contain the right words
- dense retrieval answers: which chunks carry the right meaning
- hybrid retrieval tries to use both signals together

Why teams use it in production:

- real user queries are messy
- some cases need exact keyword matching
- some cases need semantic matching
- hybrid retrieval usually improves robustness

#### Where sparse and dense retrieval can run

Sparse retrieval is not limited to SQL.

Dense retrieval is not limited to vector databases.

What matters is whether the system supports the needed indexing and search method well.

Sparse retrieval is commonly done in:

- search engines like Elasticsearch, OpenSearch, or Solr
- systems with inverted indexes
- some databases that support keyword search

Dense retrieval is commonly done in:

- vector databases
- vector-enabled databases such as PostgreSQL with vector support

Some modern systems support both sparse and dense retrieval for hybrid search.

#### Text chunks and retrieval

In RAG, both sparse and dense retrieval are usually applied to text chunks rather than full large documents.

Why chunks are used:

- retrieval becomes more precise
- only the most relevant text is sent to the LLM
- context noise is reduced
- answer grounding improves

Sparse retrieval works mostly on text chunks because it matches query terms against the actual words or tokens in chunk text.

Dense retrieval works on chunk embeddings, which are vector representations of chunk meaning.

#### Reranking

Reranking is the step where the system takes retrieved candidate chunks and reorders them so the most relevant chunks come first.

In simple words:

- retrieval finds possible matches
- reranking chooses the best ones

Why reranking is used:

- first-stage retrieval is fast but not always precise
- retrieved chunks may be only partly relevant
- reranking improves the final context quality before generation

Typical flow:

1. retrieve top candidate chunks
2. score them more carefully with a reranker
3. keep only the best few chunks
4. pass those chunks to the LLM

Retriever vs reranker:

- retriever is optimized for recall
- reranker is optimized for precision

Practical idea:

- retrieval searches large collections quickly
- reranking works on a smaller candidate set more accurately

Why reranking matters:

- reduces noisy context
- improves grounding
- often improves answer quality more than prompt changes alone

Common production pattern:

- retrieve top 20 to 100 chunks
- rerank them
- send only top 3 to 10 chunks to the LLM

Tradeoffs:

- improves quality
- adds latency
- adds compute cost
- adds pipeline complexity

Important limitation:

- reranking cannot recover chunks that retrieval failed to find

#### Bi-encoder vs cross-encoder

Bi-encoder:

- query and chunk are encoded separately
- similarity is computed between their vectors
- fast and scalable
- commonly used for dense retrieval

Why bi-encoders are used in dense retrieval:

- large collections need fast search
- document embeddings can be precomputed and stored
- only the query embedding needs to be computed at runtime
- vector similarity search is efficient for millions of chunks

Cross-encoder:

- query and chunk are encoded together as one input
- the model directly predicts relevance for that query-chunk pair
- slower but more accurate
- commonly used for reranking

Why cross-encoders are used in reranking:

- they read query and chunk jointly
- they model deeper interaction between the two texts
- they improve precision on a small candidate set
- they are too expensive for full collection search

Best mental model:

- bi-encoder is for fast candidate retrieval
- cross-encoder is for careful final relevance judgment

#### Metadata filtering

Metadata filtering means narrowing retrieval using structured information attached to documents or chunks.

Examples of metadata:

- document type
- source
- department
- date
- author
- access level
- product name
- language
- page number
- region

Why metadata filtering is useful:

- it restricts retrieval to the correct search space
- it improves precision
- it helps compliance and access control
- it reduces noise from unrelated sources

How it works:

- chunks are stored with text, embeddings, and metadata
- retrieval is limited to chunks that match the filter conditions

Example:

- query: `What is the data retention policy for EU customers?`
- filter by `region = EU`, `document_type = policy`, `domain = privacy`

Industry note:

- metadata filtering is very important in enterprise, legal, healthcare, and multi-tenant systems

Common mistakes:

- not storing useful metadata at indexing time
- relying only on semantic search when structured constraints matter
- over-filtering and removing useful results

Best mental model:

- metadata filtering decides where retrieval is allowed to search

#### Query rewriting

Query rewriting means reformulating the user query into a better query for retrieval without changing the intended meaning.

Why it is useful:

- users often ask vague, short, informal, or ambiguous questions
- documents may use more formal or domain-specific language
- better retrieval queries improve the chance of finding the right chunks

Example:

- original query: `Can temp staff roll over leave?`
- rewritten query: `Can temporary staff carry forward unused leave days according to company leave policy?`

What query rewriting improves:

- keyword mismatch
- vague wording
- missing context
- conversational phrasing
- misalignment between user language and document language

Common types of query rewriting:

1. clarifying rewrite
2. expansion rewrite
3. standardization rewrite
4. multi-query rewriting

How query rewriting is achieved:

- rule-based rewriting
- LLM-based rewriting
- query expansion
- multi-query generation
- context-aware rewriting using conversation history

Rule-based rewriting:

- uses predefined logic such as synonym replacement, abbreviation expansion, spelling correction, or domain normalization

LLM-based rewriting:

- uses a language model to rewrite the query into a clearer and more retrieval-friendly form

Query expansion:

- adds related keywords or terms without fully changing the query

Multi-query generation:

- creates several alternative rewritten versions of the same question
- retrieval is run on all of them and results are combined

Context-aware rewriting:

- uses earlier conversation turns to restore missing context in follow-up questions

Important caution:

- query rewriting should improve searchability without changing the actual meaning of the user question

Common mistakes:

- over-rewriting and changing intent
- adding too much detail and making the query too narrow
- assuming rewriting is always necessary
- not evaluating whether rewritten queries actually improve retrieval

Best mental model:

- metadata filtering improves where to search
- query rewriting improves how to search

#### Chunking

Chunking means splitting large documents into smaller pieces before retrieval.

Why chunking is needed:

- large documents are too broad for precise retrieval
- smaller chunks improve retrieval precision
- the LLM receives more focused context
- answer grounding becomes better

Example:

- a large HR document may contain leave policy, salary policy, reimbursement rules, and work-from-home rules
- if stored as one large block, retrieval may return too much irrelevant text
- if chunked, retrieval can return only the leave-policy section

What makes a good chunk:

- small enough to be precise
- large enough to preserve meaning
- self-contained enough to be understandable

Chunking methods:

1. fixed-size chunking
2. recursive chunking
3. semantic chunking
4. document-aware chunking

Fixed-size chunking:

- splits by character count, token count, or sentence count
- simple and common
- may break meaning at awkward boundaries

Recursive chunking:

- tries natural boundaries first, such as headings, paragraphs, or sentences
- preserves structure better than simple fixed splitting

Semantic chunking:

- tries to keep logically related meaning together
- more advanced than simple size-based splitting

Document-aware chunking:

- uses document structure such as headers, sections, tables, pages, markdown, or HTML structure
- useful in real enterprise and structured documents

Chunk overlap:

- repeats a small portion of text between neighboring chunks
- helps preserve context when meaning spans boundaries

Why overlap helps:

- avoids cutting off important context
- improves retrieval when related information sits across two chunks

Chunking tradeoff:

- very large chunks reduce precision
- very small chunks lose context
- too much overlap increases duplication and cost

Why chunking matters:

- affects retrieval precision
- affects reranking quality
- affects answer grounding
- affects cost and latency

Common mistakes:

- chunks too large
- chunks too small
- ignoring document structure
- using no overlap when context crosses chunk boundaries
- using one chunking strategy for every document type

Best mental model:

- chunking defines the basic unit of knowledge that retrieval can return

#### Generation includes

- sending the question plus retrieved context to the LLM
- generating a grounded final answer

Key point:

- the model should answer from retrieved context, not from unsupported memory
- prompt design and context ordering can strongly affect answer quality

#### Why RAG can be better than fine-tuning for document QA

- documents can be updated without retraining
- cheaper and faster to maintain
- better for factual lookup over changing knowledge
- useful for private internal knowledge bases

#### Security note

RAG itself does not automatically prevent data leaks. Data safety depends on deployment choices, access controls, storage policy, and whether the model provider trains on submitted data.

RAG can reduce data exposure risk when retrieval and generation are run in a local or controlled environment.

#### Real-world uses

- internal company knowledge assistant
- support chatbot over documentation
- legal or policy assistant
- enterprise search
- PDF and report question answering

#### Practice references

- None yet
