# 🎓 Campus Multi-Agent AI Assistant
### Powered by LangGraph, RAG, FAISS & Semantic Caching

---

## 📌 1. Project Overview

The **Campus Multi-Agent AI Assistant** is an intelligent, production-style conversational system built to answer the full spectrum of college-related student queries — placements, exam preparation, project ideas, campus events, academic/college knowledge, and transport — through a single unified chat interface.

Instead of relying on one generic chatbot model, the system is architected as a **team of specialized AI agents**, each an expert in one domain, coordinated by an intelligent routing layer built on **LangGraph**. Every query is enhanced, checked against a semantic cache for instant reuse, routed to the correct expert agent, grounded in real data through **Retrieval-Augmented Generation (RAG)**, and polished before being returned to the user.

This design mirrors how a real campus help-desk works — a front office that understands your question and forwards it to the right department, except here it happens in milliseconds and is powered by LLMs.

---

## 🚀 2. Key Highlights

| Capability | Description |
|---|---|
| **Multi-Agent Architecture** | 6 independent, domain-specialized agents working under one orchestrator |
| **LangGraph Workflow Orchestration** | A stateful graph controls the entire query lifecycle from input to final answer |
| **Query Enhancement** | An LLM rewrites vague/short user queries into clear, keyword-rich, routable queries |
| **Semantic Answer Caching** | Previously answered (or near-duplicate) questions are served instantly from a FAISS-backed cache, skipping redundant computation |
| **Hybrid Intent Router** | A 3-stage cascade (TF-IDF → Sentence Embeddings → LLM fallback) picks the right agent with minimal latency and LLM cost |
| **Retrieval-Augmented Generation (RAG)** | Agents ground their answers in real documents (PDFs, CSVs) instead of hallucinating |
| **FAISS Vector Database** | High-speed semantic search over embedded document chunks |
| **OCR-Ready PDF Ingestion** | PyMuPDF-based parsing supports scanned/structured academic PDFs |
| **Groq LLM Integration** | Ultra-low-latency inference using Llama-3.3-70B-Versatile via Groq |
| **Answer Enhancement Layer** | A dedicated post-processing agent restructures and clarifies raw answers without altering facts |
| **Interactive Gradio UI** | A clean, custom-themed "Reading Room" interface for real-time interaction |
| **Modular & Extensible Design** | Every agent is a plug-and-play node — new domains can be added without touching existing logic |

---

## 🧩 3. System Architecture

The entire pipeline is modeled as a **stateful directed graph** using LangGraph, where a shared `RagState` object flows through nodes, accumulating context at each step.

```
                     ┌────────────────────┐
                     │        START        │
                     └──────────┬───────────┘
                                │
                     ┌──────────▼───────────┐
                     │   Query Enhancer      │  (LLM rewrites the query)
                     └──────────┬───────────┘
                                │
                     ┌──────────▼───────────┐
                     │  Semantic Cache Check  │  (FAISS similarity ≥ 0.95?)
                     └──────────┬───────────┘
                          Hit  │   Miss
                     ┌─────────┴─────────┐
                     ▼                   ▼
                 ┌───────┐      ┌────────────────┐
                 │  END   │      │ Hybrid Router   │  (TF-IDF → Embeddings → LLM)
                 └───────┘      └───────┬────────┘
                                        │
        ┌──────────────┬───────────────┼──────────────┬───────────────┬──────────────┐
        ▼              ▼               ▼              ▼               ▼              ▼
   Placement     Important Q's    Project Idea     Event Agent   College Knowledge  Transport
     Agent           Agent           Agent          (Web Search)      Agent          Agent
        │              │               │              │               │              │
        └──────────────┴───────────────┴──────────────┴───────────────┴──────────────┘
                                        │
                             ┌──────────▼───────────┐
                             │   Answer Enhancer      │  (LLM polishes final answer)
                             └──────────┬───────────┘
                                        │
                             ┌──────────▼───────────┐
                             │  Store in Cache        │
                             └──────────┬───────────┘
                                        │
                                     ┌──▼──┐
                                     │ END  │
                                     └─────┘
```

---

## 🤖 4. The Six Specialized Agents

### 1️⃣ Placement Information Agent
Answers questions about hiring companies, job roles, required skills, minimum CGPA, and internship opportunities by performing RAG over a structured `placement_data.csv` dataset.

### 2️⃣ Important Questions Generation Agent
Analyzes lecture notes, textbooks, and study PDFs to generate high-quality, exam-oriented questions — chapter-wise, topic-wise, and viva/interview style — strictly grounded in the retrieved material.

### 3️⃣ Project Idea Generation Agent
Generates complete, structured project recommendations (objective, tech stack, features, dataset, roadmap, future scope) across AI, ML, IoT, Cybersecurity, Web, Cloud, and Data Science domains — ideal for mini/major/hackathon projects.

### 4️⃣ College Events Information Agent
Performs **real-time web search** (via DuckDuckGo) to discover the latest hackathons, workshops, seminars, and competitions, then uses an LLM to summarize scattered web content into clean, structured event details.

### 5️⃣ College Knowledge Agent
Answers questions about faculty, curriculum, and academic structure by running RAG over the official college syllabus PDF (e.g., `AR22-CSE-SYLLABUS`).

### 6️⃣ Transport Information Agent
Answers transportation-related queries (bus routes, timings, stops) using a hybrid **TF-IDF + positional retrieval** pipeline over the official transport handbook PDF, combined with LLM-based answer generation.

---

## 🧠 5. Hybrid Multi-Agent Intent Router

Rather than always invoking an LLM to decide "which agent should handle this?" — which is slow and costly — the router uses a **three-stage cascade** that escalates only when needed:

1. **Stage 1 — TF-IDF Semantic Matching:** Fast keyword-level matching against domain descriptions.
2. **Stage 2 — Sentence Embedding Similarity:** Deeper semantic comparison using sentence-transformer embeddings when TF-IDF confidence is low.
3. **Stage 3 — Groq LLM Classification (Fallback):** For ambiguous queries, a Groq-hosted LLM makes the final routing decision using softmax-based, temperature-scaled confidence scoring.

This design **minimizes latency and LLM usage** while preserving high routing accuracy — a key optimization for real-world, high-traffic deployment.

---

## ⚡ 6. Semantic Answer Cache

A dedicated **FAISS vector store** (`QA_CACHE`) stores every previously answered query alongside its final answer.

- Before routing any new query, the system checks the cache using `similarity_search_with_relevance_scores`.
- If a semantically similar prior question exists (similarity score ≥ **0.95**), the cached answer is returned **instantly** — bypassing retrieval, generation, and enhancement entirely.
- This dramatically reduces response time and LLM costs for repeated or near-duplicate questions (e.g., "which bus goes to Secunderabad?" vs. "bus route for Secunderabad").

---

## 🛠️ 7. Technology Stack

| Layer | Technology |
|---|---|
| **Orchestration** | LangGraph (`StateGraph`) |
| **LLM Provider** | Groq (`llama-3.3-70b-versatile`) |
| **RAG Framework** | LangChain (loaders, splitters, retriever tools, ReAct agents) |
| **Vector Database** | FAISS (`IndexFlatL2`) |
| **Embeddings** | HuggingFace — `BAAI/bge-small-en-v1.5`, `all-MiniLM-L6-v2` |
| **Document Parsing** | PyMuPDF (`fitz`), `PyMuPDFLoader`, `CSVLoader` |
| **Classical NLP Retrieval** | Scikit-learn `TfidfVectorizer`, Cosine Similarity |
| **Web Search** | DuckDuckGo Search (`ddgs`), BeautifulSoup |
| **Data Modeling** | Pydantic (`RagState`) |
| **UI** | Gradio (custom "Reading Room" theme) |
| **Language** | Python |

---

## 🔄 8. End-to-End Query Lifecycle

1. **User submits a query** through the Gradio interface.
2. **Query Enhancer** rewrites it into a clear, keyword-rich, intent-preserving version.
3. **Semantic Cache Check** looks for a near-identical past query.
   - **Cache Hit:** the stored answer is returned immediately.
   - **Cache Miss:** the pipeline proceeds.
4. **Hybrid Router** determines which of the 6 agents should handle the query.
5. The **selected agent** retrieves relevant data (via RAG/FAISS/TF-IDF/web search) and generates a grounded answer.
6. **Answer Enhancer** restructures the raw answer for clarity, formatting, and readability — without adding or hallucinating facts.
7. The final answer is **stored in the semantic cache** for future reuse.
8. The polished answer is **returned to the user** through the UI.

---

## 🎨 9. User Interface

The assistant is deployed through a custom-designed **Gradio** web app, styled as **"The Reading Room"** — a warm, scholarly interface (paper background, ink-navy text, brass accent) designed to feel like a personal academic advisor rather than a generic chatbot, complete with a "thinking" state shown while the answer resolves.

---

## 🌱 10. Future Scope

- Persistent, database-backed semantic cache (beyond in-memory FAISS)
- Multi-turn conversational memory across sessions
- Voice-based query input and text-to-speech output
- Admin dashboard for updating placement/event/transport data
- Support for additional campus domains (hostel, library, fees)
- Deployment as a WhatsApp/Telegram bot for wider student accessibility

---

## 🏆 11. Why This Project Stands Out

- **Real multi-agent orchestration** — not a single prompt with if/else logic, but a true stateful graph with conditional routing.
- **Cost- and latency-aware design** — the hybrid cascade router and semantic cache show engineering maturity beyond a typical hackathon prototype.
- **Grounded, hallucination-resistant answers** — every domain agent relies on RAG over real documents/data rather than free-form LLM generation.
- **Production-style modularity** — each agent is an independent, swappable node, making the system genuinely extensible.
- **End-to-end completeness** — from raw query to cached, enhanced, UI-delivered answer, the entire pipeline is implemented and functional.

---

## ✅ Hackathon Submission Confirmed

This project — the **Campus Multi-Agent AI Assistant** — is submitted as a complete, functional, end-to-end multi-agent RAG system demonstrating advanced orchestration (LangGraph), grounded retrieval (FAISS/RAG), intelligent routing, and semantic caching for real-world campus assistance use cases.

**🏅 Win Confirmed.**

---

## © Ownership

**All rights and ownership of this project belong to Mohammed Adil Ahmed Shareef.**
