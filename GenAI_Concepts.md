# Generative AI (GenAI) Concepts — Last-Minute Interview Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Example** to see it concretely, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Covers **core GenAI/LLM concepts** — transformers, prompting, RAG, embeddings, fine-tuning, agents, evaluation, and responsible AI — at the level asked of engineers integrating GenAI into products/pipelines. Built for a 2–4 hour final revision.
>
> *2025/26 reality: interviewers expect more than definitions — they probe **RAG, fine-tuning, evaluation, and production failures**. The strongest answers come from a **real project** (even a small RAG app), so tie concepts to something you've built.*

---

## Table of Contents

1. [What is Generative AI?](#1-what-is-generative-ai)
2. [LLMs & How They Work](#2-llms--how-they-work)
3. [Transformers & Attention](#3-transformers--attention)
4. [Tokenization & Context Window](#4-tokenization--context-window)
5. [Training Stages: Pretraining → Fine-Tuning → RLHF](#5-training-stages)
6. [Decoding & Generation Parameters](#6-decoding--generation-parameters)
7. [Prompt Engineering](#7-prompt-engineering)
8. [Embeddings & Vector Databases](#8-embeddings--vector-databases)
9. [RAG (Retrieval-Augmented Generation)](#9-rag-retrieval-augmented-generation)
10. [Fine-Tuning & PEFT/LoRA](#10-fine-tuning--peftlora)
11. [Prompting vs RAG vs Fine-Tuning (Decision)](#11-prompting-vs-rag-vs-fine-tuning)
12. [AI Agents & Tool Use](#12-ai-agents--tool-use)
13. [Hallucination & Mitigation](#13-hallucination--mitigation)
14. [Evaluation of GenAI Systems](#14-evaluation-of-genai-systems)
15. [Responsible AI, Safety & Cost](#15-responsible-ai-safety--cost)
16. [Common Scenarios (with answers)](#16-common-scenarios)
17. [Tricky / Gotcha Questions](#17-tricky--gotcha-questions)
18. [Rapid-Fire One-Liners](#18-rapid-fire-one-liners)
19. [Last-Minute Checklist](#19-last-minute-checklist)

---

## 1. What is Generative AI?

### Theory

**Generative AI** is a class of AI that **creates new content** — text, code, images, audio, video — by learning patterns from large training data, rather than just classifying or predicting labels.

- **Discriminative models** predict a label/boundary (is this spam?). **Generative models** produce new data (write an email, generate an image).
- **Foundation models** — large models pretrained on broad data, adaptable to many tasks (LLMs like GPT/Claude/Gemini/Llama; image models like diffusion models).
- **Modalities:** text (LLMs), images (diffusion: Stable Diffusion, DALL·E), audio, video, and **multimodal** models that handle multiple at once.

### Interview Q&A

**Q: What is Generative AI and how does it differ from traditional AI?**
**Generative AI creates new content** (text, code, images) by learning the underlying distribution of training data. **Discriminative/traditional ML** instead **predicts labels or boundaries** (classification, regression). Generative models *produce*; discriminative models *categorize*.

**Q: What is a foundation model?**
A **large model pretrained on broad, diverse data** that can be **adapted to many downstream tasks** via prompting, RAG, or fine-tuning — e.g., LLMs (GPT, Claude, Gemini, Llama). It's a reusable "base" rather than a task-specific model.

**Q: What's a multimodal model?**
A model that can **process and/or generate more than one modality** — e.g., take an image + text and answer questions about the image, or generate images from text. Modern frontier models are increasingly multimodal.

---

## 2. LLMs & How They Work

### Theory

A **Large Language Model (LLM)** is a neural network (usually a **Transformer**) trained on massive text to **predict the next token** given the preceding context. From that simple objective emerge fluent generation, reasoning, summarization, translation, and coding.

- **Autoregressive** — generates **one token at a time**, feeding each output back as input.
- **Parameters** — the learned weights (billions); more parameters ≈ more capacity (and cost).
- **Emergent abilities** — capabilities (e.g., in-context learning) that appear at scale.

### Interview Q&A

**Q: How does an LLM fundamentally work?**
It's trained to **predict the next token** given prior context. At inference it generates **autoregressively** — predicting a token, appending it, and repeating. Despite the simple objective, scale + Transformer architecture produce fluent, general language abilities.

**Q: What does "autoregressive" mean?**
The model generates output **one token at a time**, where each new token is conditioned on **all previously generated tokens** plus the prompt. That's why generation is sequential.

**Q: What are parameters and why do they matter?**
Parameters are the model's **learned weights**. More parameters generally mean **greater capacity** to capture patterns — but also **higher compute, memory, latency, and cost**. Bigger isn't always better; data quality and fine-tuning matter too.

---

## 3. Transformers & Attention

### Theory

The **Transformer** (Vaswani et al., 2017, "Attention Is All You Need") is the architecture behind modern LLMs. Its core is the **self-attention mechanism**, which lets each token **weigh the relevance of every other token** in the sequence — capturing long-range dependencies in parallel (unlike sequential RNNs).

- **Self-attention** uses three projections per token: **Query (Q)**, **Key (K)**, **Value (V)**. Attention score ≈ how much a token's Query matches other tokens' Keys; the output is a **weighted sum of Values**. Formula: `softmax(QKᵀ / √d) · V`.
- **Multi-head attention** — runs several attention "heads" in parallel to capture different relationship types.
- **Positional encoding** — since attention has **no inherent notion of order**, position information is added so the model knows token order.
- Most LLMs are **decoder-only** transformers.

### Interview Q&A

**Q: What is the attention mechanism (Q, K, V)?**
Self-attention lets each token decide **how much to focus on every other token**. Each token produces a **Query, Key, and Value**. The **Query is compared against all Keys** (dot product) to get attention weights, which are used to take a **weighted sum of Values** — so each token's representation incorporates relevant context. Formula: `softmax(QKᵀ/√d)·V`.

**Q: Why is positional encoding needed?**
Because **self-attention is order-agnostic** — it treats input as a set, not a sequence. **Positional encodings** inject information about each token's **position**, so the model can distinguish "dog bites man" from "man bites dog."

**Q: What is multi-head attention?**
Running **multiple attention mechanisms ("heads") in parallel**, each learning to focus on **different kinds of relationships** (syntax, coreference, etc.). Their outputs are concatenated, giving a richer representation than a single head.

**Q: Why did Transformers replace RNNs/LSTMs?**
Transformers **process all tokens in parallel** (RNNs are sequential), capture **long-range dependencies** better via attention, and **scale** far more efficiently on modern hardware — enabling today's large models.

---

## 4. Tokenization & Context Window

### Theory

- **Tokenization** — splitting text into **tokens** (subword units, via algorithms like **Byte-Pair Encoding (BPE)** or SentencePiece). LLMs operate on **token ids**, not raw characters. Roughly **1 token ≈ ¾ of a word** in English.
- **Context window** — the **maximum number of tokens** (prompt + response) the model can consider at once. Exceed it and earlier content is truncated/lost. Larger windows allow more context but cost more compute (attention scales with sequence length).

### Interview Q&A

**Q: What is tokenization and why subword tokens?**
**Tokenization** converts text into the **token units** the model processes (typically **subwords** via BPE/SentencePiece). Subwords balance vocabulary size and coverage — handling rare/unknown words by composing them from pieces, while keeping common words as single tokens.

**Q: What is the context window and why does it matter?**
The **maximum tokens (input + output) the model can attend to at once**. It limits how much document/conversation history fits. Exceeding it causes **truncation/loss of earlier context** — a key constraint for long documents (which is partly why **RAG** exists). Bigger windows cost more (attention is roughly quadratic in length).

**Q: Roughly how many words per token?**
About **1 token ≈ 0.75 words** in English (varies by language/tokenizer). Useful for estimating prompt size and cost, since pricing is usually **per token**.

---

## 5. Training Stages

### Theory

LLMs are typically built in stages:

1. **Pretraining** — self-supervised **next-token prediction** on massive unlabeled text. Builds general language ability. Hugely expensive.
2. **Supervised Fine-Tuning (SFT) / Instruction tuning** — train on **curated instruction-response pairs** so the model follows instructions.
3. **Alignment — RLHF / RLAIF / DPO** — align outputs with human preferences. **RLHF (Reinforcement Learning from Human Feedback)** trains a **reward model** from human preference rankings, then optimizes the LLM against it (often via PPO). **DPO** is a simpler preference-optimization alternative.

### Interview Q&A

**Q: What are the main stages of training an LLM?**
**Pretraining** (self-supervised next-token prediction on huge corpora → general knowledge), **instruction/supervised fine-tuning** (teach it to follow instructions), and **alignment via RLHF/DPO** (align outputs to human preferences for helpfulness and safety).

**Q: What is RLHF?**
**Reinforcement Learning from Human Feedback** — humans rank model outputs, a **reward model** is trained on those preferences, and the LLM is **optimized to maximize that reward** (e.g., with PPO). It makes models more **helpful, harmless, and aligned** with human intent than raw pretraining.

**Q: Pretraining vs fine-tuning — what's the difference?**
**Pretraining** builds broad general capability from massive unlabeled data (done once, very expensive). **Fine-tuning** adapts that pretrained model to a **specific task/domain/style** using a smaller labeled dataset — far cheaper and what most teams actually do.

---

## 6. Decoding & Generation Parameters

### Theory

At inference, parameters control **how the next token is sampled** from the probability distribution:

- **Temperature** — randomness. **Low (≈0)** = deterministic, focused; **high** = creative, diverse. 0 ≈ greedy.
- **Top-k** — sample only from the **k most likely** tokens.
- **Top-p (nucleus)** — sample from the **smallest set of tokens whose cumulative probability ≥ p**.
- **Max tokens** — caps output length.
- **Frequency/presence penalties** — discourage repetition.
- **Greedy vs sampling vs beam search** — strategies trading determinism vs diversity.

### Interview Q&A

**Q: What does temperature do?**
It controls **randomness** in token sampling. **Low temperature (~0)** makes output **deterministic and focused** (good for factual/extraction tasks); **higher temperature** makes it **more diverse/creative** (good for brainstorming) but riskier. It rescales the probability distribution before sampling.

**Q: Top-k vs top-p sampling?**
**Top-k** restricts sampling to the **k highest-probability tokens**. **Top-p (nucleus)** restricts to the **smallest set of tokens whose probabilities sum to p** — adapting the candidate set size to the distribution's shape. Top-p is usually preferred for balanced quality/diversity.

**Q: How would you set parameters for a factual extraction task vs creative writing?**
For **factual/extraction**: **low temperature (~0)**, low top-p — deterministic, consistent. For **creative writing**: **higher temperature** and top-p for variety. Match randomness to how much creativity vs precision the task needs.

---

## 7. Prompt Engineering

### Theory

**Prompt engineering** is crafting inputs to get better LLM outputs — the cheapest way to steer a model (no training).

**Techniques:**
- **Zero-shot** — just ask, no examples.
- **Few-shot** — include **examples** in the prompt (in-context learning).
- **Chain-of-Thought (CoT)** — ask the model to **reason step by step**, improving complex reasoning/math.
- **Role/system prompt** — set persona, rules, format.
- **Structured output** — request JSON/specific format; constrain with schemas.
- **ReAct** — interleave **Reasoning + Acting** (tool calls) — used in agents.
- Good practice: be **specific, give context, show examples, specify format, and define constraints**.

### Example

```text
System: You are a precise data assistant. Answer only from the provided context.
Few-shot:
  Q: ... A: ...
CoT: "Let's think step by step."
Output format: Return JSON {"answer": ..., "confidence": ...}
```

### Interview Q&A

**Q: What is prompt engineering and why does it matter?**
The practice of **designing inputs** to elicit better, more reliable LLM outputs **without changing the model**. It's the **cheapest, fastest** lever — clear instructions, context, examples, and output-format specs can dramatically improve quality.

**Q: Zero-shot vs few-shot prompting?**
**Zero-shot** gives only the instruction, no examples. **Few-shot** includes **a few input-output examples** in the prompt so the model **learns the pattern in-context**, usually improving accuracy and format adherence — at the cost of more tokens.

**Q: What is Chain-of-Thought (CoT) prompting?**
Prompting the model to **reason step by step** before answering (e.g., "Let's think step by step"). It significantly **improves performance on complex reasoning, math, and multi-step tasks** by externalizing intermediate reasoning rather than jumping to an answer.

**Q: What is ReAct prompting?**
**Reasoning + Acting** — the model alternates between **reasoning steps and actions** (like calling tools/searching), observing results and continuing. It's the backbone of many **agents**, letting the model interact with external tools to solve multi-step tasks.

---

## 8. Embeddings & Vector Databases

### Theory

- **Embeddings** — **dense numeric vectors** representing the **semantic meaning** of text (or images). Similar meanings → vectors close together in space. Produced by an **embedding model** (e.g., OpenAI text-embedding, sentence-transformers).
- **Semantic similarity** — measured by **cosine similarity** (or dot product / Euclidean distance) between vectors.
- **Vector database** — stores embeddings and does fast **similarity search** (nearest neighbors) using **ANN (Approximate Nearest Neighbor)** indexes like **HNSW**. Examples: **Pinecone, Weaviate, Milvus, FAISS, pgvector, Elasticsearch (kNN)**.

### Interview Q&A

**Q: What are embeddings?**
**Dense vector representations of meaning** — text (or images) mapped into a high-dimensional space where **semantically similar items are close together**. They power **semantic search, RAG retrieval, clustering, and recommendations** by enabling meaning-based (not keyword-based) comparison.

**Q: How do you measure similarity between embeddings?**
Most commonly **cosine similarity** (angle between vectors), or dot product / Euclidean distance. Higher cosine similarity = more semantically related. Vector DBs use these to find **nearest neighbors**.

**Q: What is a vector database and why ANN?**
A database optimized to **store embeddings and run fast similarity search**. Exact nearest-neighbor search is too slow at scale, so they use **Approximate Nearest Neighbor (ANN)** indexes (e.g., **HNSW**) to find close matches **quickly** with a small accuracy trade-off.

**Q: How do you choose an embedding model for RAG?**
Match it to your **domain and language**, consider **dimension size vs cost/speed**, the **max input length**, retrieval **benchmark performance (e.g., MTEB)**, and whether it's **consistent** (use the **same model for indexing and querying**). Domain-specific data may need a specialized or fine-tuned embedder.

---

## 9. RAG (Retrieval-Augmented Generation)

### Theory

**RAG** combines a **retriever** with an **LLM**: instead of relying only on the model's parametric memory, it **retrieves relevant external knowledge** and **inserts it into the prompt** so the model generates a **grounded answer**. It's the dominant pattern for **factual, up-to-date, domain-specific** GenAI apps.

**Pipeline:**
1. **Ingest** documents → **chunk** them → **embed** each chunk → **store** in a vector DB (indexing).
2. At query time: **embed the user query** → **similarity search** retrieves top-k relevant chunks → **insert chunks into the prompt** as context → **LLM generates** a grounded answer (often with citations).

**Chunking strategies:** **fixed-size** (simple, may split mid-thought), **semantic** (split on meaning/structure), **recursive** (split by separators hierarchically). Chunk size trades **context completeness vs retrieval precision**; overlap preserves continuity.

**Why RAG:** reduces hallucination, uses **current/proprietary** data without retraining, provides **citations**, and is **cheaper than fine-tuning** for knowledge injection.

### Example (pipeline)

```text
[Indexing]  docs -> chunk -> embed -> vector DB
[Query]     question -> embed -> top-k similarity search
            -> build prompt: "Answer using ONLY this context: {chunks}\nQ: {question}"
            -> LLM -> grounded answer (+ citations)
```

### Interview Q&A

**Q: What is RAG and why use it?**
**Retrieval-Augmented Generation** **retrieves relevant external documents** and feeds them into the LLM's prompt so answers are **grounded in real data**. It's used to give the model **current, proprietary, or domain-specific knowledge** without retraining — **reducing hallucination** and enabling **citations**.

**Q: Walk me through a RAG pipeline.**
**Indexing:** ingest documents → **chunk** → **embed** each chunk → store in a **vector DB**. **Querying:** embed the user query → **similarity search** for top-k relevant chunks → **insert them into the prompt** as context → the **LLM generates a grounded answer**. Optionally add **reranking** and **citations**.

**Q: How do you choose a chunk size / chunking strategy?**
Balance **retrieval precision vs context completeness**. **Too small** loses context; **too large** dilutes relevance and wastes tokens. **Fixed-size** is simple; **semantic/recursive** chunking respects natural boundaries (paragraphs/sections) for better coherence. Add **overlap** so ideas spanning chunk boundaries aren't lost. Tune empirically for your data.

**Q: RAG vs just using a bigger context window?**
Even with large windows, stuffing **everything** is **costly, slower, and dilutes relevance** ("lost in the middle"). RAG **retrieves only the relevant pieces**, which is cheaper, often more accurate, and scales to corpora far larger than any context window.

**Q: How do you improve a RAG system that returns poor answers?**
Diagnose **retrieval vs generation**. Improve **retrieval**: better chunking, a stronger **embedding model**, **hybrid search** (keyword + vector), **reranking**, metadata filters, more/fewer top-k. Improve **generation**: better prompt instructing it to use only the context and say "I don't know," and add **citations** to catch ungrounded answers.

---

## 10. Fine-Tuning & PEFT/LoRA

### Theory

**Fine-tuning** adapts a pretrained model to a **specific task/domain/style/format** by training on a curated dataset.

- **Full fine-tuning** — update **all** weights. Powerful but **expensive** (compute, memory, storage per task).
- **PEFT (Parameter-Efficient Fine-Tuning)** — update only a **small fraction** of parameters.
- **LoRA (Low-Rank Adaptation)** — **freezes the pretrained weights** and injects small **trainable low-rank matrices** into transformer layers, cutting trainable parameters by orders of magnitude (~10,000×) — cheap, fast, and you can swap LoRA "adapters" per task. **QLoRA** adds quantization to fine-tune big models on modest hardware.
- **Key hyperparameters:** learning rate, epochs, batch size, **LoRA rank** (and alpha).

### Interview Q&A

**Q: What is fine-tuning and when do you do it?**
Adapting a **pretrained model to a specific task/domain/style** by training on a curated dataset. Use it when you need the model to **reliably follow a format, adopt a tone, or master a narrow task** that prompting/RAG can't achieve — not for injecting facts (that's RAG's job).

**Q: What is LoRA and why is it popular?**
**Low-Rank Adaptation** — it **freezes the original weights** and trains small **low-rank adapter matrices** added to each layer, reducing trainable parameters **by ~10,000×**. This makes fine-tuning **cheap, fast, and storage-efficient**, and lets you keep **swappable per-task adapters** on one base model. **QLoRA** combines it with quantization to fine-tune large models on a single GPU.

**Q: Full fine-tuning vs PEFT/LoRA?**
**Full fine-tuning** updates all weights — maximum flexibility but **huge compute/memory/storage** and a full model copy per task. **PEFT/LoRA** updates a **tiny subset**, achieving most of the benefit at a **fraction of the cost**, which is why it's the default for most teams.

**Q: Key fine-tuning hyperparameters?**
**Learning rate** (most sensitive), **number of epochs** (too many → overfitting), **batch size**, and for LoRA the **rank** and **alpha**. Also data quality/quantity — clean, representative examples matter more than volume.

---

## 11. Prompting vs RAG vs Fine-Tuning

### Theory

The **most-asked decision question.** Choose based on the need:

| Approach | Best for | Cost | Updates knowledge? |
|---|---|---|---|
| **Prompt engineering** | Quick steering, formatting, general tasks | Lowest | No (uses what's in prompt) |
| **RAG** | Injecting **current/proprietary facts**, citations, reducing hallucination | Medium | **Yes** (just update the data) |
| **Fine-tuning** | Teaching **behavior/format/style/skill**, narrow tasks, latency/cost at scale | Highest (training) | Not ideal for facts |

**Rule of thumb:** start with **prompt engineering** → add **RAG** for knowledge/grounding → **fine-tune** for behavior/format that prompting can't reliably achieve. They're **complementary**, often combined (e.g., fine-tuned model + RAG).

### Interview Q&A

**Q: When do you choose prompting vs RAG vs fine-tuning?**
Start with **prompt engineering** (cheapest, fast). Use **RAG** when you need **external, current, or proprietary knowledge** grounded with citations — it updates by changing the data, not the model. Use **fine-tuning** when you need to change **behavior, format, tone, or a specialized skill** that prompting can't reliably produce. **RAG = knowledge; fine-tuning = behavior.** They're often combined.

**Q: Your model gives outdated answers about company data. RAG or fine-tuning?**
**RAG** — it injects the **current/proprietary knowledge** at query time and updates instantly when the data changes, with citations. Fine-tuning bakes in knowledge that **goes stale** and is expensive to refresh, so it's the wrong tool for frequently-changing facts.

**Q: The model won't follow your strict output format despite prompting. What do you try?**
First push **prompt engineering** (few-shot examples, explicit schema, structured-output/JSON mode). If it still fails at scale, **fine-tune** on examples of the desired format — that reliably teaches a consistent behavior/format prompting struggles with.

---

## 12. AI Agents & Tool Use

### Theory

An **AI agent** is an LLM-powered system that can **reason, plan multi-step tasks, use tools, and act** toward a goal — maintaining **state** and adapting based on **feedback** — unlike a single-turn chatbot.

- **Tools/function calling** — the LLM decides to call external functions/APIs (search, code execution, DB query) and uses results.
- **ReAct loop** — Reason → Act (tool call) → Observe → repeat until done.
- **Memory** — short-term (context) and long-term (vector store) to retain information.
- **Planning** — decompose a goal into steps; **multi-agent** systems coordinate specialized agents.
- Frameworks: LangChain, LlamaIndex, LangGraph, CrewAI, AutoGen (mention only if relevant).

### Interview Q&A

**Q: What is an AI agent and how does it differ from a chatbot?**
An **agent** is an LLM that can **plan multi-step tasks, call tools/APIs, maintain state, and adapt** based on observations to achieve a goal. A **chatbot** typically does **single-turn Q&A** without taking actions. Agents *act on the world* (search, run code, update systems); chatbots *respond*.

**Q: What is function calling / tool use?**
The ability of an LLM to **invoke external functions/APIs** in a structured way — the model outputs which tool to call and with what arguments, the system executes it, and the result is fed back. It lets LLMs access **live data, computation, and actions** beyond their training.

**Q: What is the ReAct pattern in agents?**
**Reason + Act** — the agent alternates **reasoning steps** with **actions (tool calls)**, **observing** each result and continuing until the task is solved. It grounds the LLM's reasoning in real tool outputs, reducing hallucination and enabling multi-step problem solving.

**Q: What are the main risks with agents?**
**Compounding errors** over many steps, **infinite loops/cost blowups**, **unsafe actions** (executing harmful operations), and **prompt injection** via tool outputs. Mitigate with **step limits, guardrails, human-in-the-loop approvals, sandboxing, and validation** of tool inputs/outputs.

---

## 13. Hallucination & Mitigation

### Theory

**Hallucination** is when an LLM generates **plausible-sounding but false or unsupported** content. It happens because LLMs are **probabilistic next-token predictors optimized for fluency**, not truth — they have no built-in fact-checking and will "fill in" confidently.

**Mitigations:**
- **RAG** — ground answers in retrieved sources (the biggest lever).
- **Prompting** — instruct "answer only from context; say 'I don't know' if unsure"; require **citations**.
- **Lower temperature** for factual tasks.
- **Verification** — fact-check against sources, use a second model/critic, structured validation.
- **Human-in-the-loop** for high-stakes outputs.

### Interview Q&A

**Q: What is hallucination and why does it happen?**
When an LLM produces **confident but false/unsupported** information. It happens because the model is a **probabilistic text predictor optimized for plausibility/fluency**, not grounded truth — it lacks an internal fact-checker and generates the most likely continuation even when it doesn't "know."

**Q: How do you reduce hallucination in a production app?**
**Ground with RAG** (retrieve real sources and cite them), **prompt the model to use only the provided context and admit uncertainty**, **lower temperature**, add **verification/guardrails** (validate against sources, a critic model), and use **human review** for high-stakes outputs. RAG + citations is usually the highest-impact fix.

**Q: Can you eliminate hallucination entirely?**
No — you can **substantially reduce** it but not guarantee zero. The realistic goal is **grounding, citations, validation, and human oversight** proportional to the risk, plus clear UX that the output may be imperfect.

---

## 14. Evaluation of GenAI Systems

### Theory

Evaluating generative output is hard (no single "correct" answer). Approaches:

- **Automatic metrics:** BLEU/ROUGE (overlap, for translation/summarization — limited), **perplexity** (language modeling), **exact match/F1** (QA).
- **Embedding/semantic similarity** to references.
- **LLM-as-a-judge** — use a strong LLM to **score** outputs against criteria (increasingly common).
- **Human evaluation** — gold standard for quality, helpfulness, safety.
- **RAG-specific:** **faithfulness/groundedness** (is the answer supported by retrieved context?), **answer relevance**, **context precision/recall** (did retrieval fetch the right chunks?). Tools: RAGAS, etc.
- **Task metrics + guardrail/safety evals + A/B tests** in production.

### Interview Q&A

**Q: How do you evaluate an LLM/GenAI system?**
With a **mix**: **automatic metrics** (ROUGE/BLEU/F1/perplexity where applicable), **semantic similarity**, **LLM-as-a-judge** scoring against rubrics, and **human evaluation** for quality/safety. In production, add **A/B testing** and monitoring. The right metric depends on the task — there's rarely one number.

**Q: How do you evaluate a RAG system specifically?**
Separate **retrieval** and **generation** quality. Retrieval: **context precision/recall** (did it fetch the right chunks?). Generation: **faithfulness/groundedness** (is the answer supported by the retrieved context?) and **answer relevance**. Tools like **RAGAS** automate these; combine with human spot-checks.

**Q: What is "LLM-as-a-judge"?**
Using a **capable LLM to score or compare outputs** against defined criteria (helpfulness, correctness, format). It's **scalable and cheaper than human eval**, though it can carry biases — best validated against human judgments and used with clear rubrics.

---

## 15. Responsible AI, Safety & Cost

### Theory

- **Bias & fairness** — models can reflect/amplify biases in training data; test and mitigate.
- **Privacy/security** — don't leak PII or proprietary data; beware sending sensitive data to third-party APIs; **prompt injection** and **data leakage** are real threats.
- **Prompt injection** — malicious input that overrides instructions (especially via retrieved/tool content); mitigate with input/output validation, separation of trusted vs untrusted content, and guardrails.
- **Guardrails** — content filters, output validation, allow/deny lists.
- **Cost/latency** — pricing is **per token**; manage with smaller models where possible, caching, prompt compression, RAG (less context), and batching.
- **Transparency** — citations, disclaimers, human oversight for high-stakes use.

### Interview Q&A

**Q: What is prompt injection and how do you defend against it?**
An attack where **malicious instructions embedded in user input or retrieved content** hijack the model (e.g., "ignore previous instructions"). Defenses: **separate trusted system instructions from untrusted content**, **validate/sanitize** inputs and tool outputs, use **guardrails/filters**, least-privilege tool access, and never blindly execute model-suggested actions.

**Q: How do you control GenAI cost and latency?**
Costs are **per token**, so: use a **smaller/cheaper model** when sufficient, **cache** repeated responses, **compress prompts / trim context** (RAG helps by sending only relevant chunks), **limit max tokens**, **batch** requests, and stream responses for perceived latency. Pick the smallest model that meets quality needs.

**Q: How do you handle PII/sensitive data with LLMs?**
**Minimize and redact** sensitive data before sending it, prefer **private/self-hosted or enterprise** endpoints with no-training guarantees, enforce **access controls and audit logs**, and validate outputs to avoid leaking data. Treat third-party API calls as a data-governance decision.

**Q: How do you address bias and safety?**
**Evaluate for bias** across groups, use **diverse test sets**, apply **guardrails/content filters**, keep **humans in the loop** for high-stakes decisions, and be **transparent** (citations, disclaimers). Safety is an ongoing process of testing, monitoring, and iteration.

---

## 16. Common Scenarios

**Q: Design a chatbot that answers questions over our internal documents.**
"A **RAG system**: **ingest** the docs → **chunk** (semantic/recursive with overlap) → **embed** with a suitable model → store in a **vector DB**. At query time, **embed the question**, **retrieve top-k** relevant chunks (optionally **rerank**), build a prompt instructing the LLM to **answer only from the context and cite sources** (and say 'I don't know' otherwise), and generate. Add **evaluation** (faithfulness, retrieval precision), **guardrails**, **caching** for cost, and **monitoring**. RAG (not fine-tuning) because the knowledge is proprietary and changes."

**Q: The RAG bot gives wrong/irrelevant answers. How do you debug?**
"First isolate **retrieval vs generation**. Inspect the **retrieved chunks** — if they're irrelevant, improve **chunking, the embedding model, hybrid search, reranking, or top-k**. If retrieval is good but the answer is wrong, fix the **prompt** (force grounding, add citations) or check for **context overflow**. Add **faithfulness eval** to quantify grounding."

**Q: Leadership wants to fine-tune a model on company data for a Q&A bot. Do you agree?**
"For **factual Q&A over changing company data, RAG is usually better** — it stays current, is cheaper, and provides citations; fine-tuning bakes in facts that go stale and is costly to refresh. I'd recommend **RAG for the knowledge**, and consider **fine-tuning only if** we need a specific **tone/format/behavior** RAG + prompting can't achieve — possibly both together."

**Q: How would you add a GenAI feature to an existing data pipeline?**
"Identify the task (summarization, classification, extraction, NL-to-SQL). Choose **prompting first**; add **RAG** if it needs context from our data; **fine-tune** only if needed. Wrap the LLM call with **structured output, validation/guardrails, retries, cost controls, and monitoring/eval**, store prompts in version control, and treat it like any pipeline component with **idempotency and observability**."

---

## 17. Tricky / Gotcha Questions

**1. RAG = knowledge; fine-tuning = behavior.** Don't fine-tune to inject changing facts.

**2. LLMs predict tokens, not truth** — fluency ≠ correctness; that's why hallucination happens.

**3. Bigger context window ≠ "just stuff everything"** — costly, slower, "lost in the middle"; RAG retrieves only what's relevant.

**4. Temperature 0 isn't fully deterministic** in practice (hardware/implementation), but it's the most focused setting.

**5. Embeddings must use the same model for indexing and querying** — mixing models breaks similarity.

**6. Fine-tuning needs quality data** — small clean datasets beat large noisy ones; risk of overfitting/catastrophic forgetting.

**7. LoRA freezes base weights** and trains tiny adapters — that's why it's so cheap.

**8. Prompt injection comes via retrieved/tool content too** — not just direct user input.

**9. Agents can loop and rack up cost** — always add step limits and guardrails.

**10. Evaluation has no single metric** — combine automatic, LLM-as-judge, and human eval.

**11. Tokens ≠ words** — pricing/limits are per token (~0.75 words each).

**12. RAG quality is mostly a retrieval problem** — fix retrieval before blaming the LLM.

---

## 18. Rapid-Fire One-Liners

- **GenAI** creates new content; **discriminative AI** classifies.
- **LLM** = transformer trained to predict the **next token**, autoregressively.
- **Transformer** core = **self-attention** (Q, K, V); `softmax(QKᵀ/√d)·V`.
- **Positional encoding** adds order (attention is order-agnostic).
- **Tokenization** = text → subword tokens (BPE); ~0.75 words/token.
- **Context window** = max tokens (input+output) the model can attend to.
- **Training:** pretraining → instruction/SFT → alignment (**RLHF/DPO**).
- **Temperature** = randomness; **top-k/top-p** = candidate set; low temp for facts.
- **Prompt engineering:** zero-shot, few-shot, **CoT** (step-by-step), ReAct.
- **Embeddings** = meaning as vectors; compare via **cosine similarity**.
- **Vector DB** = fast similarity search via **ANN/HNSW** (Pinecone, FAISS, pgvector).
- **RAG** = retrieve relevant chunks → put in prompt → grounded answer.
- **RAG pipeline:** ingest → chunk → embed → store → retrieve → generate (+cite).
- **Chunking:** fixed/semantic/recursive; size = precision vs context; use overlap.
- **Fine-tuning** = adapt behavior/format; **LoRA** = freeze + train tiny adapters (~10,000× fewer params); **QLoRA** adds quantization.
- **Decision:** prompt → RAG (knowledge) → fine-tune (behavior); often combined.
- **Agent** = plans + uses tools + maintains state (vs single-turn chatbot).
- **Function calling** lets LLMs call APIs/tools; **ReAct** = reason+act loop.
- **Hallucination** = confident false output; mitigate with **RAG + citations + low temp + validation**.
- **Evaluation** = automatic + **LLM-as-judge** + human; RAG: faithfulness + context precision/recall.
- **Risks:** prompt injection, PII leakage, bias, cost (per token), agent loops — use guardrails.

---

## 19. Last-Minute Checklist

The hour before:

- [ ] GenAI vs discriminative AI; foundation/multimodal models.
- [ ] How LLMs work: next-token prediction, autoregressive, parameters.
- [ ] **Transformer & attention (Q/K/V)**; positional encoding; multi-head.
- [ ] **Tokenization** and the **context window** (and why RAG exists).
- [ ] Training stages: pretraining → SFT → **RLHF**.
- [ ] Decoding params: **temperature, top-k, top-p**; when to use which.
- [ ] **Prompt engineering**: zero/few-shot, **CoT**, ReAct, structured output.
- [ ] **Embeddings + vector DBs** (cosine similarity, ANN/HNSW).
- [ ] **RAG**: full pipeline, chunking strategies, how to improve it (the #1 topic).
- [ ] **Fine-tuning & LoRA/PEFT/QLoRA**; key hyperparameters.
- [ ] **Prompting vs RAG vs fine-tuning** decision (knowledge vs behavior).
- [ ] **Agents**: tool/function calling, ReAct, memory, risks.
- [ ] **Hallucination** causes and mitigations.
- [ ] **Evaluation** (incl. RAG faithfulness, LLM-as-judge).
- [ ] **Responsible AI**: prompt injection, PII, bias, **cost/latency**.
- [ ] Practice the **scenarios** in §16 (design a doc-QA bot, debug RAG, RAG vs fine-tune).

**Interview tips:** The highest-frequency topics are **RAG, the prompting-vs-RAG-vs-fine-tuning decision, transformers/attention, and hallucination mitigation** — know these cold. In 2025/26, interviewers expect **production awareness** (evaluation, cost, safety, failures), not just definitions. The strongest signal is a **real project** — even a small RAG app — so frame answers around something you've built. Given your data-engineering background, you can connect GenAI to **data pipelines** (embedding pipelines, vector stores, RAG over your data, NL-to-SQL) and your **Elasticsearch** experience (kNN/vector search) — that's a genuine edge.

---

*Good luck — read it top-to-bottom once, then drill RAG (the whole pipeline + how to improve it), the prompting/RAG/fine-tuning decision, attention (Q/K/V), and hallucination mitigation. Those decide GenAI interviews in 2025/26.*
