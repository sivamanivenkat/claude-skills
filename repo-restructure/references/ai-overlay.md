# AI / RAG Overlay

Read only when the repo uses LLM SDKs, embeddings, vector DBs or prompt files. Place these layers inside the stack's source root (for example `app/` in Python, `<base.package>/ai/` in Java, `src/ai/` in Node).

## Layers
| Layer | Contents |
|---|---|
| retrieval (components) | `hybrid_retriever`, `reranker`: hybrid search plus reranking |
| services | `rag_pipeline`, `semantic_cache`, `conversation` (memory), `query_rewriter`, `query_router` |
| prompts | `templates`, `registry`: versioned, type-specific, hot-swappable. Java/Node may keep prompt text in resource files. |
| agents | `document_grader`, `query_decomposer`, `adaptive_router`: self-correcting retrieval and LLM-driven source selection |
| agents/tools | `vector_search`, `web_search`, `code_search`: pluggable tool definitions |
| security | `input_guard`, `content_filter`, `output_filter`: three guard layers |
| evaluation | `golden_dataset.json`, `offline_eval`, `online_monitor`, `eval_results/` |
| observability | `tracer` (per-stage), `feedback`, `cost_tracker` |
| data | `raw/` → `processed/` → `index_config/` |

## Classification hints
- LLM client calls with prompt strings inline: extract the prompt to `prompts/` only if the user approves a split; otherwise move the file to `services/`.
- Vector DB clients (Pinecone, Qdrant, pgvector, Chroma) go to retrieval or adapters.
- Langfuse, LangSmith or OpenTelemetry hooks go to observability.
- Moderation, PII or prompt-injection checks go to security.

## Docs
In `docs/architecture.md`, the Mermaid request flow should show: input guard → query rewriter/router → retrieval → reranker → LLM → output filter, with cache and tracing alongside.
