# LLMs and Generative AI: Interview Notes

## 1. How LLMs work (the short version)
- A **language model** predicts the next token given previous tokens: `P(x_t | x_<t)`.
- **Tokenization:** text is split into subword tokens (BPE, WordPiece, SentencePiece). Token count drives cost and context limits; one English word is about 1.3 tokens on average.
- **Architecture:** decoder-only transformer with causal self-attention (GPT, Llama, Qwen, Gemini-style). Embeddings -> N transformer blocks -> logits over vocabulary -> sampling.
- **Training stages:**
  1. **Pre-training:** next-token prediction on huge corpora (knowledge and language).
  2. **Supervised fine-tuning (SFT) / instruction tuning:** follows instructions.
  3. **Alignment:** RLHF, DPO or similar preference optimisation (helpful, harmless, honest).
- **Context window:** maximum tokens (prompt + output). Attention cost grows with length; KV cache stores past keys/values to avoid recomputation.

## 2. Decoding and sampling
| Setting | Effect |
|---|---|
| Temperature | scales logits; 0 is near-deterministic, higher is more random |
| Top-k | sample from the k most likely tokens |
| Top-p (nucleus) | sample from the smallest set with cumulative probability p |
| Greedy / beam search | always best token / keep several candidates |
| Max tokens, stop sequences | limit and terminate output |
| Repetition/frequency penalties | reduce loops |
Use low temperature for extraction/classification/RAG; higher for creative writing. Structured output: JSON mode / function calling / constrained decoding.

## 3. Prompt engineering
- Be explicit: role, task, constraints, output format, examples.
- **Zero-shot, few-shot** (examples in the prompt), **chain-of-thought** (ask for step-by-step reasoning), **self-consistency** (vote over samples), **ReAct** (reason and act with tools), **structured outputs** (schema), **system vs user messages**.
- Put the question and constraints where the model attends best; delimit retrieved context; tell the model to say "I don't know" when context is insufficient.
- Version and test prompts like code (golden test sets).
- **Prompt injection:** untrusted text (web pages, user documents) may contain instructions; separate data from instructions, validate outputs, limit tool permissions, add input and output guards.

## 4. Adaptation techniques
| Technique | When |
|---|---|
| Prompting / few-shot | first try; cheapest |
| RAG | need fresh or private knowledge, citations, no retraining |
| Fine-tuning (full) | change style/format/skills; needs data and GPUs |
| **PEFT: LoRA / QLoRA** | adapt big models cheaply by training small low-rank adapters (QLoRA adds 4-bit quantisation) |
| Distillation | smaller faster student from a larger teacher |
| Tool use / function calling | let the model call APIs, code, search |
Rule of thumb: **RAG for knowledge, fine-tuning for behaviour**. On very small datasets fine-tuning can hurt (your MCQ project: zero-shot LLM logit scoring beat fine-tuned DeBERTa).

## 5. LLM logit scoring (your MCQ approach)
Instead of generating text, feed "question + options + Answer:" and read the next-token logits for the option letters (A..E); rank by probability to produce top-3 predictions. Benefits: no generation parsing errors, direct probabilities for ranking (MAP@3). Few-shot examples improved it (0.626 to 0.705).

## 6. Evaluation
- **Task metrics** (accuracy, F1, MAP@k, exact match), **LLM-as-judge** (with rubrics and checks for bias), human review, golden sets, regression tests per prompt/model change.
- **Text metrics:** BLEU/ROUGE (overlap), BERTScore, perplexity (language modelling only).
- **RAG metrics (RAGAS):** faithfulness (answer supported by context), answer relevancy, context precision, context recall.
- **Safety:** toxicity, PII leakage, jailbreak resistance, hallucination rate.
- Benchmarks (MMLU, GSM8K, HumanEval) measure general ability, not your task.

## 7. Hallucination and mitigation
Causes: the model optimises plausibility, not truth; missing or outdated knowledge; ambiguous prompts. Mitigations: RAG with citations, low temperature, constrained outputs, verification step (Critic agent), tool use for facts/math, "answer only from context", confidence thresholds, human-in-the-loop for high stakes.

## 8. Serving and cost
- Hosted APIs (OpenAI, Anthropic, Gemini, Groq) vs self-hosted (vLLM, TGI, Ollama, llama.cpp).
- **Latency drivers:** model size, input tokens (prefill), output tokens (decode), batching, network. Stream tokens for responsiveness.
- **Cost control:** shorter prompts, caching (prompt/response/semantic caching), smaller models for simple steps (routing), batching, limiting max tokens.
- **Reliability:** timeouts, retries with backoff, provider fallbacks (Groq -> OpenAI -> Ollama), rate-limit handling, deterministic fallbacks (your Healthcare Gemini layer), audit logging of prompt, output and latency.
- **Optimisation:** quantisation (int8/int4), KV-cache, continuous batching (vLLM), speculative decoding, FlashAttention.

## 9. Embeddings and semantic search
- Embedding models map text to vectors where meaning is close in cosine similarity (SBERT, BGE, E5, OpenAI embeddings).
- **BGE** uses a query instruction prefix for retrieval queries (asymmetric search), used in your matcher.
- **Bi-encoder** (fast retrieval, independent embeddings) vs **cross-encoder** (slow, jointly scores query-document pairs; used for reranking).
- Normalise vectors; choose dimension and model by quality/latency; evaluate with recall@k on your data.

## 10. Multimodal and other generative models
Vision-language models (image + text), speech (Whisper), diffusion models for images (Stable Diffusion), code models. Same ideas: tokenisation of modalities, cross-attention, large-scale pre-training.

## 11. Safety, privacy and governance
PII redaction, data retention, access control per document, prompt-injection defences, content filters, audit logs, bias testing, licensing of models and data, compliance (GDPR, DPDP), disclaimers for regulated domains (finance, medical).

## 12. Key vocabulary
Token, context window, embedding, attention, KV cache, logits, temperature, top-p, system prompt, few-shot, chain-of-thought, RAG, agent, tool/function calling, guardrail, hallucination, fine-tuning, LoRA, RLHF, DPO, quantisation, distillation, perplexity, MoE (mixture of experts), RoPE.

## Quick interview Q&A
- **How does an LLM generate text?** Autoregressively predicts the next token from logits and samples, repeating.
- **What is temperature?** Scales logits before softmax to control randomness.
- **RAG vs fine-tuning?** RAG injects external knowledge at query time; fine-tuning changes model weights/behaviour.
- **What is LoRA?** Train small low-rank matrices added to frozen weights; cheap fine-tuning.
- **Why do LLMs hallucinate and how do you reduce it?** Section 7.
- **What is a context window and what are the limits?** Max tokens; longer means more cost and attention dilution.
- **What is function calling?** Model outputs a structured call; your code executes it and returns results.
- **How do you evaluate an LLM application?** Golden set + task metrics + LLM judge + human review + RAG metrics.
- **Bi-encoder vs cross-encoder?** Independent embeddings (fast) vs joint scoring (accurate, slower).
- **How do you defend against prompt injection?** Separate instructions from data, least-privilege tools, validate outputs, guard inputs.
- **Why did zero-shot logit scoring beat fine-tuning in your MCQ project?** Small data (2,000 rows) and a large pretrained model; fine-tuning overfit while the pretrained knowledge was already strong.
