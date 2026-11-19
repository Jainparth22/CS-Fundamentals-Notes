# Generative AI, LLMs and RAG: 12 Most-Asked Questions

**Q1. Explain the main parts of a RAG system.**
Ingestion (parse, chunk, embed, index into a vector store with metadata), retrieval (embed the query, search, optionally rerank), and generation (build a prompt with the retrieved context and have the LLM answer with citations), plus evaluation and guardrails.

**Q2. Why use RAG instead of relying on the LLM alone, and RAG vs fine-tuning?**
RAG gives up-to-date, private, citable knowledge without retraining and reduces hallucination. Fine-tuning changes behaviour, style or skills. Rule of thumb: RAG for knowledge, fine-tuning for behaviour, and often both.

**Q3. What is the role of a vector database, and do you always need one?**
It stores embeddings and does fast approximate nearest-neighbour search (HNSW, IVF) with metadata filters. Not always: small corpora can use an in-memory index (FAISS), BM25 or even a long context window.

**Q4. Chunking strategies and size trade-offs?**
Fixed size with overlap, recursive by structure, semantic, parent-child. Small chunks give precise matches but lose context; large chunks keep context but dilute relevance and cost more tokens. Tune on a labelled question set.

**Q5. What is hybrid search?**
Combining lexical (BM25) and dense vector search, usually merged with Reciprocal Rank Fusion, so exact terms (names, numbers) and meaning are both captured.

**Q6. What does a reranker do?**
A cross-encoder rescores the top candidates by reading query and document together, improving precision at the cost of latency. Retrieve many, rerank, keep the best few.

**Q7. How do you evaluate a RAG system?**
Measure retrieval (recall@k, MRR) and generation quality: faithfulness, answer relevancy, context precision and recall (RAGAS), plus a golden question set, human review and regression tests on every change.

**Q8. What causes hallucinations and how do you reduce them?**
The model optimises plausible text, lacks or misreads knowledge, or gets weak context. Reduce with grounded prompts and citations, better retrieval and reranking, low temperature, a verifier/critic step, tool use for facts, refusal when evidence is weak, and human review for high stakes.

**Q9. How do you keep a RAG system reliable in production?**
Timeouts, retries, provider fallbacks, caching, input and output guardrails (prompt injection, PII), logging of retrieved chunks and prompts, monitoring latency and cost, scheduled re-indexing, access control per document.

**Q10. How do you handle multi-turn conversations and fresh data?**
Rewrite follow-up questions into standalone queries using chat history, keep a trimmed or summarised memory, re-index on a schedule or on change events, use timestamps and recency boosts, and delete stale documents.

**Q11. Prompting vs fine-tuning vs RAG: how do you choose, and what is LoRA?**
Start with prompting, add RAG for knowledge, fine-tune only when behaviour or format needs changing and you have data. LoRA trains small low-rank adapters on frozen weights, making fine-tuning cheap (QLoRA adds 4-bit quantisation).

**Q12. What are agents, tool calling and LangGraph?**
An agent loops: the LLM decides on a tool, your code runs it with validated arguments, and results feed back until done. Tool calling returns structured calls. LangGraph models this as a stateful graph with branching, loops, checkpoints and human-in-the-loop steps; limit steps and permissions for safety.
