# CS Fundamentals Notes

> Personal study notes by **Parth Jain** (B.Tech CSE VIT Bhopal + BS DS IIT Madras)
> Built chapter-by-chapter as ongoing revision material for SDE interviews.

---

## Table of Contents

### 📦 Data Structures (`DS/`)
| File | Topics |
|------|--------|
| [01_arrays_and_strings.md](DS/01_arrays_and_strings.md) | Prefix sum, sliding window, Kadane's |
| [02_linked_lists.md](DS/02_linked_lists.md) | Singly/doubly, Floyd's cycle, reverse |
| [03_stacks_and_queues.md](DS/03_stacks_and_queues.md) | Monotonic stack, NGE, deque |
| [04_trees.md](DS/04_trees.md) | Traversals, BST, LCA, balance |
| [05_heaps.md](DS/05_heaps.md) | Heapify, heap sort, top-K, median |
| [06_hash_tables.md](DS/06_hash_tables.md) | Chaining, open addressing, LRU |
| [07_graphs.md](DS/07_graphs.md) | BFS/DFS, topo sort, Dijkstra, MST |
| [08_tries.md](DS/08_tries.md) | Insert/search, XOR trie |

### ⚙️ Algorithms (`Algo/`)
| File | Topics |
|------|--------|
| [01_sorting.md](Algo/01_sorting.md) | All sorts, stability, complexity table |
| [02_searching.md](Algo/02_searching.md) | Binary search variants, C++ STL |
| [03_dynamic_programming.md](Algo/03_dynamic_programming.md) | 7 DP patterns, full code |
| [04_greedy.md](Algo/04_greedy.md) | Exchange argument, interval scheduling |
| [05_backtracking.md](Algo/05_backtracking.md) | Template, subsets, N-queens |

### 💻 Operating Systems (`OS/`)
| File | Topics |
|------|--------|
| [01_processes_and_threads.md](OS/01_processes_and_threads.md) | PCB, mutex, semaphore |
| [02_cpu_scheduling.md](OS/02_cpu_scheduling.md) | FCFS/SJF/RR, Gantt, formulas |
| [03_memory_management.md](OS/03_memory_management.md) | Paging, TLB, page replacement |
| [04_deadlocks.md](OS/04_deadlocks.md) | Coffman, Banker's algorithm |

### 🗄️ DBMS (`DBMS/`)
| File | Topics |
|------|--------|
| [01_er_and_relational_model.md](DBMS/01_er_and_relational_model.md) | ER, normalization to BCNF |
| [02_sql.md](DBMS/02_sql.md) | Joins, window functions, 10 problems |
| [03_transactions.md](DBMS/03_transactions.md) | ACID, isolation levels, 2PL |
| [04_indexing.md](DBMS/04_indexing.md) | B+ tree, covering index |

### 🌐 Computer Networks (`CN/`)
| File | Topics |
|------|--------|
| [01_osi_and_tcp_ip.md](CN/01_osi_and_tcp_ip.md) | OSI layers, TCP/IP, PDUs |
| [02_http_and_https.md](CN/02_http_and_https.md) | HTTP versions, TLS handshake |
| [03_tcp_vs_udp.md](CN/03_tcp_vs_udp.md) | 3-way handshake, flow control |
| [04_dns_and_routing.md](CN/04_dns_and_routing.md) | DNS resolution, BGP, NAT |

### 🏗️ OOP (`OOP/`)
| File | Topics |
|------|--------|
| [01_pillars.md](OOP/01_pillars.md) | Encapsulation, Inheritance, Polymorphism, Abstraction |
| [02_solid_principles.md](OOP/02_solid_principles.md) | SRP, OCP, LSP, ISP, DIP |

### 🏛️ High Level Design (`HLD/`)
| File | Topics |
|------|--------|
| [01_scalability.md](HLD/01_scalability.md) | Horizontal/vertical, consistent hashing, CAP |
| [02_caching.md](HLD/02_caching.md) | Redis, CDN, write strategies, stampede |
| [03_databases_at_scale.md](HLD/03_databases_at_scale.md) | SQL vs NoSQL, sharding, WAL |
| [04_message_queues.md](HLD/04_message_queues.md) | Kafka, RabbitMQ, outbox pattern |
| [05_system_design_template.md](HLD/05_system_design_template.md) | Full template + URL shortener |
| [06_distributed_patterns.md](HLD/06_distributed_patterns.md) | Idempotency, OCC, saga, outbox, rate limiting, resilience |
| [07_system_design_interview_qa.md](HLD/07_system_design_interview_qa.md) | Framework + worked designs (chat, feed, booking, RAG, ML serving) |

### 🔧 Low Level Design + Design Patterns (`LLD/`)
| File | Topics |
|------|--------|
| [01_design_patterns_overview.md](LLD/01_design_patterns_overview.md) | All 23 GoF patterns table |
| [02_creational_patterns.md](LLD/02_creational_patterns.md) | Singleton, Factory, Builder, Prototype |
| [03_structural_patterns.md](LLD/03_structural_patterns.md) | Adapter, Decorator, Facade, Proxy |
| [04_behavioral_patterns.md](LLD/04_behavioral_patterns.md) | Observer, Strategy, Command, State |
| [05_lld_problems.md](LLD/05_lld_problems.md) | 10 LLD problems with class designs |

### 🐍 Python Interview Notes (`Python/`)
| File | Topics |
|------|--------|
| [01_core_language.md](Python/01_core_language.md) | Data model, mutability, scope, generators, decorators, GIL, memory |
| [02_oop_and_diamond_problem.md](Python/02_oop_and_diamond_problem.md) | OOP, `super()`, MRO and C3, diamond problem, dunders, dataclasses |
| [03_concurrency_and_async.md](Python/03_concurrency_and_async.md) | Threads, processes, asyncio, performance, testing |
| [04_python_coding_questions.md](Python/04_python_coding_questions.md) | Output questions, classic problems, pandas/NumPy one-liners |

### 🗃️ SQL Interview Notes (`SQL/`)
| File | Topics |
|------|--------|
| [01_sql_fundamentals.md](SQL/01_sql_fundamentals.md) | DDL/DML, constraints, NULLs, aggregates, subqueries, upsert |
| [02_joins_window_functions_cte.md](SQL/02_joins_window_functions_cte.md) | Joins, CTEs, recursive CTEs, window functions |
| [03_indexes_transactions_optimization.md](SQL/03_indexes_transactions_optimization.md) | Indexes, EXPLAIN, ACID, isolation, locking, normalization |
| [04_sql_interview_problems.md](SQL/04_sql_interview_problems.md) | 28 solved problems (top-N, gaps and islands, retention, job queues) |

### 🌐 Backend (`Backend/`)
| File | Topics |
|------|--------|
| [01_http_rest_api_design.md](Backend/01_http_rest_api_design.md) | HTTP, REST, status codes, idempotency, pagination |
| [02_flask.md](Backend/02_flask.md) | Flask, app factory, blueprints, SQLAlchemy, auth, deployment |
| [03_fastapi.md](Backend/03_fastapi.md) | FastAPI, Pydantic, Depends, async vs sync, WebSockets |
| [04_auth_jwt_security.md](Backend/04_auth_jwt_security.md) | JWT, sessions, OAuth, OWASP, rate limiting |
| [05_redis.md](Backend/05_redis.md) | Data types, caching patterns, persistence, locks, rate limits |
| [06_celery_background_jobs.md](Backend/06_celery_background_jobs.md) | Celery, retries, idempotency, outbox, Beat |

### 🤖 Machine Learning (`ML/`)
| File | Topics |
|------|--------|
| [01_ml_fundamentals.md](ML/01_ml_fundamentals.md) | Bias-variance, metrics, imbalance, algorithms, leakage, MLOps |
| [02_scikit_learn.md](ML/02_scikit_learn.md) | API, pipelines, CV, tuning, stacking, serving |

### 🧠 Deep Learning (`DL/`)
| File | Topics |
|------|--------|
| [01_deep_learning_fundamentals.md](DL/01_deep_learning_fundamentals.md) | Backprop, optimizers, normalization, CNN/RNN/transformers, transfer learning |
| [02_pytorch.md](DL/02_pytorch.md) | Tensors, autograd, training loop, HF transfer learning, debugging |

### ✨ Generative AI (`GenAI/`)
| File | Topics |
|------|--------|
| [01_llm_and_transformers.md](GenAI/01_llm_and_transformers.md) | LLM basics, prompting, fine-tuning/LoRA, evaluation, serving |
| [02_rag_langchain_agents.md](GenAI/02_rag_langchain_agents.md) | RAG, hybrid retrieval, rerank, LangChain, LangGraph, guardrails |

### ❓ Interview Question Bank (`QuestionBank/`)
| File | Topics |
|------|--------|
| [README.md](QuestionBank/README.md) | 150 most-asked questions, sources, and practice links |
| [01_python_25.md](QuestionBank/01_python_25.md) | 25 Python questions |
| [02_sql_25.md](QuestionBank/02_sql_25.md) | 25 SQL questions |
| [03_flask_fastapi_20.md](QuestionBank/03_flask_fastapi_20.md) | 20 Flask and FastAPI questions |
| [04_redis_celery_18.md](QuestionBank/04_redis_celery_18.md) | 18 Redis and Celery questions |
| [05_system_design_20.md](QuestionBank/05_system_design_20.md) | 20 system design questions |
| [06_ml_20.md](QuestionBank/06_ml_20.md) | 20 machine learning questions |
| [07_dl_pytorch_10.md](QuestionBank/07_dl_pytorch_10.md) | 10 deep learning and PyTorch questions |
| [08_genai_rag_12.md](QuestionBank/08_genai_rag_12.md) | 12 GenAI, LLM and RAG questions |

### Study Notes: DSA, Python, SQL (`Study-Notes/`)
| Folder | Topics |
|------|--------|
| [DSA-Fundamentals](Study-Notes/DSA-Fundamentals/) | Complexity, arrays, hashing, sorting, trees, heaps, graphs, DP, interview method |
| [Python-Interview](Study-Notes/Python-Interview/python_notes.md) | Python semantics, OOP and MRO, concurrency, patterns |
| [SQL-Interview](Study-Notes/SQL-Interview/sql_notes.md) | Joins, window functions, indexes, isolation levels, problems |
| [README](Study-Notes/README.md) | How these were made and their limits |

---

## How to Use This Repo

1. **Daily** — open one topic, read the notes, code the examples from scratch without looking
2. **Spaced repetition** — after 1 day, 3 days, 1 week, re-read and quiz yourself on the practice problems at the bottom of each file
3. **Interview prep** — skim the "Quick Reference" tables before each interview
4. **Cross-link with DSA-Practice** — each notes file references relevant LeetCode problems

---

## Progress

- [ ] DS — 0/8 chapters done
- [ ] Algo — 0/5 chapters done
- [ ] OS — 0/4 chapters done
- [ ] DBMS — 0/4 chapters done
- [ ] CN — 0/4 chapters done
- [ ] OOP — 0/2 chapters done
- [ ] HLD — 0/7 chapters done
- [ ] LLD — 0/5 chapters done
- [ ] Python — 0/4 chapters done
- [ ] SQL — 0/4 chapters done
- [ ] Backend — 0/6 chapters done
- [ ] ML — 0/2 chapters done
- [ ] DL — 0/2 chapters done
- [ ] GenAI — 0/2 chapters done
- [ ] Question bank — 0/150 questions revised
