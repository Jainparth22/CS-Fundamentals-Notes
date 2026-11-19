# RAG, LangChain, LangGraph and Agents: Interview Notes

## 1. What RAG is and why
**Retrieval-Augmented Generation:** retrieve relevant documents at query time and give them to the LLM as context, so answers are grounded in your data, up to date, and citable, without retraining.

```
Ingestion: documents -> parse -> clean -> chunk -> embed -> vector store (+ metadata, + BM25 index)
Query:     question -> (rewrite) -> retrieve top-k -> (rerank) -> build prompt -> LLM -> answer + citations -> (guard)
```

## 2. Ingestion decisions
- **Parsing:** PDFs (layout, tables, OCR for scans), DOCX, HTML, CSV; keep source, page, section as metadata.
- **Chunking:** fixed size (e.g. 300 to 800 tokens) with overlap (10 to 20%), recursive by headings/paragraphs, semantic chunking, parent-child chunks (retrieve small, return larger context). Too small loses context; too large dilutes relevance.
- **Embeddings:** BGE, E5, OpenAI embeddings; normalise vectors; same model for documents and queries (BGE uses a query prefix).
- **Vector databases:** FAISS (library), Chroma, Qdrant, Weaviate, Milvus, pgvector, Pinecone. Index types: flat (exact), HNSW (graph, fast and accurate), IVF (clustered), product quantisation (compression). ANN trades a little recall for big speed.
- **Metadata filters** (tenant, date, doc type) and access control at retrieval time.
- Incremental updates, deduplication, versioned indexes.

## 3. Retrieval quality
| Technique | Why |
|---|---|
| **Hybrid search** (BM25 + dense) | BM25 catches exact terms (numbers, names, IDs); dense catches meaning |
| **Reciprocal Rank Fusion (RRF)** | merge rankings: score = sum 1/(k + rank_i) |
| **Reranking (cross-encoder)** | rescore top 20 to 50 candidates for precision |
| **Query rewriting / HyDE / multi-query** | reformulate vague questions; embed a hypothetical answer |
| **MMR** | diversify results |
| **Parent-document / sentence-window retrieval** | precise match, richer context |
| **Metadata filtering / self-query** | narrow by structured fields |
| **Contextual compression** | trim retrieved text to relevant parts |
Measure retrieval separately: recall@k, MRR, hit rate on a labelled question set.

## 4. Generation
- Prompt template: instructions + context (with source ids) + question; "answer only from the context; cite sources; say you do not know otherwise".
- Context budget: pick top-k that fits the window; order matters ("lost in the middle").
- Citations with chunk ids/page numbers; refuse when retrieval is weak (similarity threshold).
- Low temperature for factual answers.

## 5. Evaluation (RAGAS and beyond)
- **Faithfulness:** claims supported by context. **Answer relevancy:** addresses the question. **Context precision:** retrieved chunks are relevant and ranked well. **Context recall:** needed info was retrieved.
- Build a golden set (question, reference answer, source), run it on every change (chunking, embedder, reranker, prompt), track regressions. Add human review and LLM-judge with care for bias.

## 6. Failure modes and fixes
| Failure | Fix |
|---|---|
| Right doc not retrieved | better chunking, hybrid search, query rewriting, more k then rerank, metadata |
| Retrieved but ignored | prompt instructions, reorder, shorter context, stronger model |
| Hallucinated details | grounding prompt, citations, Critic/verifier step, lower temperature |
| Outdated answers | re-index schedule, timestamps, recency boost |
| Tables / numbers wrong | table-aware parsing, keep tables intact, structured extraction |
| Slow or costly | caching, smaller reranker, ANN index tuning, streaming, model routing |
| Prompt injection from documents | sanitise, treat context as data, output validation |
| Data leakage across users | per-tenant indexes/filters, ACL checks |

## 7. LangChain essentials
- **LCEL (LangChain Expression Language):** compose with `|`: `prompt | llm | StrOutputParser()`. Components are `Runnable`s with `invoke`, `batch`, `stream`.
- Core pieces: chat models, prompt templates, output parsers (Pydantic/JSON), document loaders, text splitters, embeddings, vector stores, retrievers, memory/chat history, tools, callbacks/tracing (LangSmith).
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
prompt = ChatPromptTemplate.from_template("Answer from context only.\nContext: {context}\nQuestion: {q}")
chain = {"context": retriever | format_docs, "q": lambda x: x} | prompt | llm | StrOutputParser()
chain.invoke("What was FY24 revenue?")
```
- **Chains vs agents:** a chain has a fixed sequence; an agent lets the LLM decide which tools to call in a loop.
- Know when you do not need it: for simple pipelines plain Python plus the provider SDK is clearer; LangChain helps with integrations and tracing.

## 8. Agents and LangGraph
- **Agent loop:** observe -> think (LLM) -> act (tool) -> observe results -> repeat until done or a step limit. Patterns: **ReAct**, plan-and-execute, reflection, multi-agent.
- **Tools / function calling:** the LLM emits a structured call (name + JSON args); your code validates and runs it; results go back to the model. Validate arguments, limit permissions, time out, log everything.
- **LangGraph:** build agents as **stateful graphs**: nodes (functions/LLM calls), edges (including conditional routing), shared state, loops, checkpoints (persistence, resume), human-in-the-loop interrupts, streaming.
- **Your FinSight design:** Researcher (retrieve) -> Responder (draft with citations) -> Critic (check support and hallucination, send back for revision) with a human-review node; hybrid retrieval, reranking, input/output guardrails, RAGAS evaluation, SQLite memory, Groq -> OpenAI -> Ollama fallback.
- **Multi-agent pitfalls:** cost and latency multiply, loops need hard step limits, state grows, errors compound; prefer the simplest graph that works.

## 9. Guardrails
- **Input:** prompt-injection detection, PII detection/redaction, topic and length limits, auth.
- **Output:** schema validation, hallucination score vs sources, banned content, required disclaimers (regulated domains), citation check.
- **Tool safety:** allow-lists, read-only by default, confirmation for destructive actions, budgets and step limits.

## 10. Memory
Short-term (conversation history, trimmed/summarised), long-term (user facts in a DB or vector store), semantic memory, episodic logs. Store with TTL and privacy controls; summarise to control tokens.

## 11. Production concerns
Caching (embedding cache, retrieval cache, semantic response cache), streaming responses, async ingestion queues (Celery), idempotent re-indexing, observability (trace each step: retrieved chunks, prompt, latency, tokens, cost), rate limits and quotas, evaluation in CI, fallback providers and a deterministic fallback, cost dashboards, versioned prompts and indexes, A/B tests.

## 12. Building a minimal RAG (no framework)
```python
chunks = split(documents)                              # 500-token chunks, overlap 50
vecs = embedder.encode(chunks, normalize_embeddings=True)
index = faiss.IndexFlatIP(vecs.shape[1]); index.add(vecs)

def answer(q, k=5):
    qv = embedder.encode(["Represent this sentence for searching relevant passages: " + q], normalize_embeddings=True)
    D, I = index.search(qv, k)
    ctx = "\n\n".join(f"[{i}] {chunks[i]}" for i in I[0])
    return llm(f"Use only the context. Cite ids.\n\nContext:\n{ctx}\n\nQuestion: {q}")
```
Then add BM25 + RRF, a cross-encoder reranker, metadata filters, and evaluation.

## Quick interview Q&A
- **Explain RAG end to end.** Section 1.
- **How do you choose chunk size?** Test on a golden set; start 300 to 800 tokens with overlap; consider semantic or parent-child chunking.
- **Why hybrid retrieval?** Dense handles meaning, BM25 handles exact terms; fuse with RRF.
- **What does a reranker add?** Joint query-document scoring for higher precision on the top candidates.
- **How do you evaluate RAG?** Retrieval metrics (recall@k) plus RAGAS (faithfulness, relevancy, precision, recall) on a golden set.
- **How do you reduce hallucinations in RAG?** Grounded prompt, citations, rerank, Critic/verification, thresholds, lower temperature.
- **What is an agent vs a chain?** Dynamic tool-choosing loop vs fixed sequence.
- **Why LangGraph over a plain agent loop?** Explicit state, branching, cycles, checkpointing, human-in-the-loop, debuggability.
- **What are vector index types?** Flat, HNSW, IVF, PQ; trade-offs between recall, speed, memory.
- **How do you handle multi-tenant data?** Per-tenant filters/indexes and ACL checks at retrieval.
- **How do you keep an LLM app reliable?** Timeouts, retries, provider fallbacks, deterministic fallbacks, logging, evaluation.
- **FAISS vs a managed vector DB?** FAISS is a fast in-process library; managed/DB options add persistence, filtering, scaling and ops.
