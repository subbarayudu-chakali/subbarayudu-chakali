# RAG (Retrieval-Augmented Generation) Interview Questions & Answers

An interview-ready reference for **Retrieval-Augmented Generation** — the architecture that
grounds LLM answers in your own data. Covers the pipeline end to end (chunking, embeddings,
vector stores, retrieval, generation), advanced techniques, evaluation, and production
concerns. Grouped by theme, with answers concise enough to say aloud.

---

## Fundamentals

**1. What is RAG?**

Retrieval-Augmented Generation is a technique that **retrieves relevant documents from an
external knowledge source at query time** and injects them into the LLM's prompt as context,
so the model generates answers grounded in that data rather than only its parametric
knowledge.

**2. Why use RAG?**

To ground responses in **up-to-date, private, or domain-specific** data the model wasn't
trained on; reduce hallucinations; provide **citations/sources**; avoid the cost of
retraining/fine-tuning; and control what knowledge the model uses. It's the standard way to
build Q&A over your own documents.

**3. RAG vs. fine-tuning — when do you choose which?**

**RAG** injects *knowledge* dynamically — best for changing facts, large/private corpora, and
traceability. **Fine-tuning** adjusts *behavior/style/format* and can teach narrow skills, but
is costly to update and doesn't easily add fresh facts. They're complementary: fine-tune for
form, RAG for facts. Often used together.

**4. What are the two main phases of RAG?**

**Indexing (offline)** — load, chunk, embed, and store documents in a vector index.
**Retrieval + generation (online)** — embed the query, retrieve relevant chunks, and pass
them with the query to the LLM to generate a grounded answer.

**5. Walk through the RAG pipeline end to end.**

Ingest documents → **chunk** them → **embed** chunks into vectors → store in a **vector
database**. At query time: **embed the query** → **retrieve** the top-k similar chunks →
optionally **rerank** → assemble a prompt with the chunks as context → **generate** an
answer, ideally with citations.

**6. How does RAG reduce hallucinations?**

By supplying the model authoritative context and instructing it to answer **only from that
context** (and say "I don't know" otherwise), it constrains generation to grounded facts and
enables source citation. It reduces — but doesn't eliminate — hallucination.

---

## Chunking

**7. What is chunking and why is it needed?**

Splitting documents into smaller passages ("chunks") before embedding. Needed because
embeddings and context windows have size limits, and smaller, focused chunks retrieve more
precisely than whole documents. Chunking quality strongly affects retrieval quality.

**8. What is chunk size and overlap, and how do they trade off?**

**Chunk size** is how much text per chunk; **overlap** repeats some tokens between adjacent
chunks to preserve context across boundaries. Small chunks → precise but may lose context;
large chunks → more context but noisier retrieval and more tokens. Overlap avoids splitting
key info at a boundary.

**9. What chunking strategies exist?**

Fixed-size (by tokens/characters), **recursive** (split on structure: paragraphs → sentences),
**semantic** (split where meaning shifts, using embeddings), document-structure-aware
(headings, markdown, code blocks), and sentence/paragraph-based. Choose based on document type.

**10. How do you decide chunk size?**

Empirically, via evaluation on your data and query types. Consider the embedding model's
optimal input length, the granularity of questions, and the downstream context budget.
Typical starting points are a few hundred tokens with modest overlap, then tune.

**11. What is contextual/late chunking or adding metadata to chunks?**

Enriching chunks with context — document title, section headers, a summary, or a
model-generated context sentence — and storing **metadata** (source, date, author, page) so
retrieval is more accurate and answers can cite and filter. It combats the loss of context
that naive chunking causes.

---

## Embeddings

**12. What is an embedding?**

A dense numeric vector representation of text that captures semantic meaning, such that
semantically similar texts have vectors that are close in the vector space. Embeddings are
what make **semantic** (meaning-based) search possible.

**13. What is an embedding model?**

A model that converts text into embedding vectors (e.g. OpenAI `text-embedding-3`, Cohere,
`sentence-transformers`/BGE/E5). The same model must embed both documents and queries so
they share a vector space.

**14. How do you choose an embedding model?**

By domain fit, supported language(s), vector dimensionality (cost/quality trade-off), max
input length, cost/latency, and benchmark performance (e.g. MTEB) — validated on your own
data. Match the model to the query type (short queries vs. long documents).

**15. What similarity metrics are used?**

**Cosine similarity** (most common — angle between vectors, magnitude-invariant), **dot
product**, and **Euclidean (L2) distance**. Cosine is standard for normalized text
embeddings; use the metric the embedding model was trained/recommended for.

**16. Why must the same embedding model be used for queries and documents?**

Because similarity is only meaningful within the **same vector space**. Different models
produce incompatible embeddings, so query and document vectors wouldn't be comparable and
retrieval would be meaningless.

**17. What is the "curse of dimensionality" in embeddings?**

In very high-dimensional spaces, distances between points become less discriminative and
computation costs rise. It motivates dimensionality choices, efficient ANN indexes, and
sometimes dimensionality reduction — while balancing against loss of semantic detail.

---

## Vector stores & retrieval

**18. What is a vector database?**

A database optimized to store embeddings and perform fast **similarity search** over them,
typically using approximate nearest neighbor (ANN) indexes, with metadata filtering.
Examples: Pinecone, Weaviate, Qdrant, Milvus, Chroma, pgvector, Elasticsearch/OpenSearch.

**19. What is Approximate Nearest Neighbor (ANN) search and why use it?**

Algorithms (e.g. **HNSW**, IVF, PQ) that find *approximately* the closest vectors much
faster than exact search, trading a little recall for large speed/scale gains. Essential when
searching millions+ of vectors under latency constraints.

**20. What is HNSW?**

Hierarchical Navigable Small World — a graph-based ANN index that navigates layered proximity
graphs to find nearest neighbors quickly with high recall. A widely used default in vector
databases.

**21. What is top-k retrieval?**

Returning the **k** most similar chunks to the query. `k` balances recall vs. noise/context
budget — too small misses relevant info, too large adds distractors and cost. Tune it, and
consider reranking a larger candidate set down to a smaller final set.

**22. What is semantic search vs. keyword search?**

**Keyword (lexical/BM25)** search matches exact terms — great for names, codes, rare tokens,
exact phrases. **Semantic** search matches meaning via embeddings — great for paraphrases and
concepts. Each misses cases the other catches.

**23. What is hybrid search?**

Combining **semantic (vector)** and **keyword (BM25/sparse)** retrieval and fusing the
results (e.g. **Reciprocal Rank Fusion**). It captures both meaning and exact-term matches,
usually outperforming either alone — especially for queries with specific terms.

**24. What is metadata filtering?**

Restricting retrieval by structured attributes stored with chunks (date, source, department,
access level) — e.g. "only 2025 docs the user may read." Crucial for correctness, freshness,
and access control in production RAG.

---

## Advanced RAG techniques

**25. What is reranking?**

A second stage where a **cross-encoder reranker** rescoring retrieved candidates by deeper
query-document relevance, reordering (and trimming) them before generation. Retrieve a larger
candidate set with fast ANN, then rerank to a precise top few. It notably improves precision.

**26. Bi-encoder vs. cross-encoder — what's the difference?**

A **bi-encoder** embeds query and document separately (fast, precomputable — used for
retrieval). A **cross-encoder** processes query+document *together* for a relevance score
(accurate but slow — used for reranking a shortlist). RAG often uses both in sequence.

**27. What is query transformation / rewriting?**

Reformulating the user query to improve retrieval: **query expansion** (add synonyms/terms),
**decomposition** (split multi-part questions), **step-back** prompting (ask a more general
question), and conversational rewriting (resolve pronouns using chat history). It bridges the
gap between how users ask and how documents are written.

**28. What is HyDE (Hypothetical Document Embeddings)?**

Generate a **hypothetical answer** to the query with an LLM, embed *that*, and retrieve
documents similar to it. Because a hypothetical answer resembles real documents more than a
short query does, retrieval improves — especially for sparse or vague queries.

**29. What is multi-query retrieval / RAG-Fusion?**

Generate several paraphrased variants of the query, retrieve for each, and fuse the results
(e.g. RRF). It broadens coverage and reduces sensitivity to a single query phrasing.

**30. What is a parent-document / small-to-big retriever?**

Retrieve on small, precise chunks for matching accuracy, but return the **larger parent
context** (surrounding section/document) to the LLM for generation — getting both precise
retrieval and sufficient context.

**31. What is sentence-window retrieval?**

Embed and match single sentences, then expand to include a window of surrounding sentences
when passing context to the model — precision on retrieval, coherence for generation.

**32. What is contextual compression?**

Post-retrieval filtering/summarizing of chunks to keep only the parts relevant to the query
before sending to the LLM — reducing noise and token cost, improving answer focus.

**33. What is agentic RAG?**

Wrapping retrieval in an **agent** that decides *whether*, *what*, and *how many times* to
retrieve, can call multiple sources/tools, reformulate queries, and iterate — rather than a
single fixed retrieve-then-generate step. More powerful for complex, multi-hop questions.

**34. What is GraphRAG?**

RAG over a **knowledge graph** (entities and relationships) rather than (or alongside) flat
text chunks, enabling multi-hop reasoning and global/summary questions that flat vector
retrieval handles poorly.

**35. How do you handle multi-hop questions?**

Use query decomposition, iterative/agentic retrieval (retrieve, reason, retrieve again),
multi-query, or GraphRAG. A single retrieval rarely surfaces all pieces needed to chain facts
across documents.

---

## Generation & prompting

**36. How do you construct the generation prompt in RAG?**

Include a clear instruction to **answer only from the provided context**, the retrieved
chunks (clearly delimited, often with source labels), the user question, and instructions to
**cite sources** and say "I don't know" if the answer isn't present. Order and formatting matter.

**37. How do you get citations/attributions?**

Attach source metadata to each chunk, instruct the model to reference which chunk(s) support
each claim (e.g. `[1]`), and map those back to sources. Some pipelines verify citations
post-hoc against the retrieved text.

**38. How do you keep the model from using outside knowledge?**

Explicitly instruct it to rely solely on the context, forbid speculation, provide an "I don't
know" escape hatch, and optionally validate the answer against retrieved text. It reduces but
doesn't fully prevent parametric leakage.

**39. How do you handle context that exceeds the window?**

Retrieve fewer/better chunks (rerank, compress), summarize or map-reduce across chunks, use a
larger-context model, and prioritize the most relevant passages. Don't just stuff everything —
it raises cost and can trigger "lost in the middle."

---

## Evaluation

**40. How do you evaluate a RAG system?**

Evaluate **retrieval** and **generation** separately and end to end. Retrieval: recall@k,
precision@k, MRR, NDCG (did we fetch the right chunks?). Generation: faithfulness/groundedness,
answer relevance, correctness. Use labeled sets and LLM-as-a-judge; frameworks like RAGAS help.

**41. What is faithfulness/groundedness vs. answer relevance?**

**Faithfulness/groundedness** — is the answer supported by the retrieved context (no
hallucination)? **Answer relevance** — does it actually address the user's question?
**Context relevance/precision** — were the retrieved chunks relevant? These are the core RAG
quality dimensions.

**42. What are common RAG failure modes?**

Missing content (not in the corpus), retrieval miss (relevant chunk not retrieved), poor
chunking splitting key info, top-k too small/large, reranking absent, prompt not enforcing
grounding, context overflow, and stale index. Diagnose by isolating retrieval vs. generation.

**43. If answers are wrong, how do you tell if it's retrieval or generation?**

Inspect the retrieved chunks: if the right context **wasn't retrieved**, it's a retrieval
problem (fix chunking, embeddings, k, hybrid, reranking). If the context **was** retrieved but
the answer is wrong/unsupported, it's a generation/prompt problem (fix grounding instructions,
model, prompt).

---

## Production concerns

**44. How do you keep the index fresh?**

Incremental ingestion pipelines that add/update/delete embeddings as source data changes,
scheduled or event-driven re-indexing, versioning, and TTL/metadata for staleness. Avoid
full re-embeds where you can — embed only changed content.

**45. How do you handle access control / security in RAG?**

Enforce permissions at retrieval via **metadata filters** tied to the requesting user
(document-level ACLs), so users only retrieve what they're authorized to see. Never rely on
the LLM to hide unauthorized content — filter before it reaches the prompt.

**46. What are the cost and latency levers in RAG?**

Embedding cost (dimensionality, corpus size), vector search latency (index type, k),
reranking cost, and generation tokens (context size, model). Optimize with caching,
smaller/quantized embeddings, ANN tuning, retrieving fewer/better chunks, and compression.

**47. What is caching in RAG?**

Caching embeddings (don't re-embed unchanged text), retrieval results for repeated queries,
and even generated answers (semantic caching for similar queries) — reducing latency and cost.

**48. What are the security risks specific to RAG?**

**Indirect prompt injection** via poisoned documents in the corpus, data leakage across
tenants/users without proper access filtering, and retrieval of sensitive data. Treat
retrieved content as untrusted, filter by permissions, and sanitize/validate.

**49. How do you handle multi-tenancy in a vector store?**

Isolate tenants via separate indexes/namespaces or strict metadata filtering on every query,
enforce it server-side, and never mix tenants' vectors in results. Access control must be
guaranteed at the retrieval layer.

---

## Quick-fire round

- **Ground answers in external data?** RAG.
- **Meaning-based search?** Semantic (embeddings); exact terms → keyword/BM25.
- **Best of both?** Hybrid search (+ RRF).
- **Reorder candidates precisely?** Reranking (cross-encoder).
- **Fast large-scale similarity search?** ANN (e.g. HNSW).
- **Fetch top matches?** Top-k retrieval.
- **Generate a fake answer to retrieve better?** HyDE.
- **Precise match, broad context?** Parent-document retriever.
- **Is the answer supported by context?** Faithfulness/groundedness.
- **Restrict by user permissions?** Metadata filtering.
- **RAG over a knowledge graph?** GraphRAG.

---

These questions cover most RAG interviews — from chunking and embeddings through hybrid
retrieval, reranking, advanced patterns, and evaluation. The fastest way to internalize
them is to build a small RAG app over your own docs, measure retrieval recall and answer
faithfulness, then improve it lever by lever: add overlap, try hybrid search, add a reranker,
and watch the metrics move. Once you've diagnosed a wrong answer down to "retrieval miss vs.
generation error," these answers become practice. See also my companion posts on
[Prompt Engineering](post.html?p=2026-08-22-prompt-engineering-interview-questions) and
[Agentic AI](post.html?p=2026-08-22-agentic-ai-interview-questions).
