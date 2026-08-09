# vwnexus
VW NEXUS / VW RAG-MCP For Use With Liminal Lore
<img width="1024" height="1024" alt="vwnexus" src="https://github.com/user-attachments/assets/18a771fb-f2a9-4034-91fc-e36f1a61d594" />
# VW Nexus

**Universal orchestration layer for multi-agent systems.**

VW Nexus is the central nervous system of the Liminal Lore suite. It provides agent registration, task scheduling with DAG dependencies, inter-agent event messaging, file locking, RAG integration, and a full HTTP + WebSocket API. Agents register with capabilities, claim tasks, and report results — all through a REST interface.

---

## What It Does

- **Agent Registry** — Tracks all active agents with heartbeats, capabilities, state, and provider info. Stale agents are pruned automatically.
- **Task Queue** — DAG-dependency-aware task scheduling. Tasks can't be claimed until dependencies are satisfied. Agents claim tasks matching their capabilities.
- **Event Bus** — Pub/sub messaging between agents. Broadcast events, subscribe to topics.
- **MCP Server** — Model Context Protocol tool exposure for IDE integration (VS Code, Cascade, etc.).
- **State Store** — SQLite-backed persistent state for cycles, sessions, telemetry, metrics.
- **Lock Manager** — File-level locking to prevent concurrent agent conflicts. TTL-based with automatic expiry.
- **Session Persistence** — Resumable agent sessions with checkpoint/restore.
- **WebSocket Server** — Real-time push notifications for deck UI and live monitoring.
- **Remote Bridge** — Connects to remote Nexus instances for distributed work across machines.
- **Agent Auto-Registration** — Declarative agent manifests (`agent_manifest.json`) for bulk registration.
- **Reasoning Map** — Structured knowledge graph mapping project components, blockers, milestones.
- **Governor** — Rate limiting and governance policies for agent behavior.
- **Committee** — Multi-agent consensus protocol for collaborative decision-making.
- **Telemetry** — Full audit trail with export and ingestion pipelines.

---

## Modules

| File | Purpose |
|------|---------|
| `api.py` | HTTP API server (84K) — all REST endpoints, CORS, auth |
| `__main__.py` | CLI entry point with port/project-root args |
| `agent_registry.py` | Thread-safe agent tracking with heartbeats |
| `agent_auto_register.py` | Bulk agent registration from JSON manifest |
| `agent_identity.py` | Agent identity, self-model prompts, heartbeat loops |
| `agent_identity_tools.py` | MCP tool for self-model generation |
| `task_queue.py` | DAG dependency task scheduling with parallel execution |
| `mcp_server.py` | MCP tool protocol server |
| `event_bus.py` | Inter-agent pub/sub messaging |
| `state_store.py` | SQLite persistent state management |
| `lock_manager.py` | File-level locking with TTL |
| `websocket_server.py` | Real-time WebSocket push notifications |
| `remote_bridge.py` | Remote Nexus instance bridging |
| `session_persistence.py` | Resumable agent sessions |
| `session_registry.py` | Session tracking |
| `reasoning_map.py` | Structured knowledge graph |
| `governor.py` | Rate limiting and governance |
| `committee.py` | Multi-agent consensus protocol |
| `metrics_db.py` | Metrics storage |
| `telemetry_export.py` | Telemetry export pipeline |
| `telemetry_ingest.py` | Telemetry ingestion from agents |
| `alerting.py` | Alert system for anomalies |
| `master_clock.py` | Synchronized clock for distributed agents |
| `models.py` | Data models / schemas |
| `provider_nexus.py` | Provider integration layer |
| `internal_mcp_client.py` | Internal MCP client for tool calls |
| `yaml_tool_loader.py` | YAML tool definition loader |
| `warp_mcp_tools.py` | Warp MCP tool registrations |
| `neural_link_logger.py` | Neural link logging |

---

## API

### Core Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/agents/register` | Register a new agent |
| `POST` | `/agents/{id}/heartbeat` | Update agent heartbeat |
| `GET` | `/agents` | List all active agents |
| `POST` | `/tasks` | Create a new task |
| `GET` | `/tasks` | List tasks (filterable) |
| `POST` | `/tasks/{id}/claim` | Claim a task for execution |
| `POST` | `/tasks/{id}/done` | Mark task complete with result |
| `POST` | `/tasks/{id}/fail` | Mark task failed |
| `POST` | `/locks/request` | Request a file lock |
| `POST` | `/events/broadcast` | Broadcast an event |
| `GET` | `/events` | Get recent events |
| `POST` | `/rag` | Query the RAG engine |
| `POST` | `/rag/index` | Index files into RAG |
| `GET` | `/rag/status` | RAG index status |

### Authentication

Optional API key via `NEXUS_API_KEY` environment variable. If set, all requests must include `Authorization: Bearer <key>` header.

---

## CLI

```powershell
# Start with defaults
python -m vw_nexus

# Custom ports
python -m vw_nexus --port 8651 --ws-port 8652

# With project root
python -m vw_nexus --project-root /path/to/project
```

---

## Agent Manifest

Agents are declared in `data/agent_manifest.json`:

```json
{
  "agents": [
    {
      "agent_id": "ollama_local",
      "name": "Ollama Local LLM",
      "provider": "ollama",
      "model": "qwen2.5-coder:7b",
      "type": "llm",
      "capabilities": ["code_review", "analysis", "architecture"],
      "metadata": { "session_id": "default" }
    }
  ]
}
```

Auto-registered on Nexus startup via `AgentAutoRegistrar`.

---

## Data Directory

| File | Purpose |
|------|---------|
| `agent_manifest.json` | Agent definitions for auto-registration |
| `corpus_config.json` | RAG corpus configuration |

SQLite databases (`nexus_state.db`, `nexus_metrics.db`, etc.) are created at runtime — not included in the build package.

---

## Integration

VW Nexus is the hub that all other Liminal Lore tools connect to:

- **ZHARK** — Can register as an agent, query RAG, post research findings as tasks
- **FUBBU** — Registers as `fubbu_router`, claims tasks, posts verified results
- **VW RAG** — RAG engine accessed via Nexus API endpoints
- **VW Deck** — Deck UI polls Nexus for agent status, tasks, events, locks
- **MCP Tools** — 16 YAML tool definitions expose Nexus functions to IDEs

  # VW RAG

**Hybrid retrieval-augmented generation engine — FTS5 keyword + optional vector search.**

VW RAG provides the knowledge retrieval layer for the Liminal Lore suite. It indexes project files, source code, documentation, and study materials into a SQLite FTS5 database with BM25 ranking. Optional sentence-transformers vector embeddings add semantic similarity search. Results are fused using reciprocal rank fusion with a recency boost.

---

## What It Does

- **FTS5 Keyword Search** — SQLite full-text search with BM25 ranking. Always available, no GPU needed.
- **Optional Vector Similarity** — sentence-transformers embeddings for semantic search. Graceful fallback to keyword-only if not installed.
- **Hybrid Retrieval** — Reciprocal rank fusion (k=60) merges keyword + vector results with recency boost.
- **Multi-Corpus Support** — Separate indices for different knowledge domains (source code, study docs, translation tools, etc.).
- **Graph Index** — Relationship-aware retrieval across project components, mapping dependencies and constraints.
- **Auto-Tagging** — 15 topic categories (combat, dx11, npc, asset, audio, quest, save, build, toolchain, rigforge, engine, memory, ui, determinism, world_map) for filtered retrieval.
- **Chunking** — Intelligent document chunking with overlap for context preservation.

---

## Modules

| File | Purpose |
|------|---------|
| `__init__.py` | Package exports |
| `__main__.py` | CLI: index, query, status |
| `chunker.py` | Document chunking with size limits and overlap |
| `embedder.py` | Sentence-transformers wrapper with graceful fallback |
| `indexer.py` | SQLite FTS5 indexing engine with batch inserts and WAL mode |
| `retriever.py` | Hybrid retrieval with reciprocal rank fusion + recency boost |
| `graph_index.py` | Relationship graph index for cross-component retrieval |
| `multi_corpus.py` | Multi-corpus management for separate knowledge domains |
| `models.py` | Data models (Chunk, SearchResult, IndexStats) |

---

## CLI

```powershell
# Index project files
python -m vw_rag index

# Index with custom path
python -m vw_rag index --path /path/to/project

# Query the index
python -m vw_rag query "rendering pipeline"

# Query with filters
python -m vw_rag query "memory allocation" --tags memory,engine --limit 10

# Show index statistics
python -m vw_rag status
```

---

## API (via VW Nexus)

VW RAG is accessed through VW Nexus HTTP endpoints:

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/rag` | Query the RAG index |
| `POST` | `/rag/index` | Index files into RAG |
| `GET` | `/rag/status` | Get index statistics |

### Query Request

```json
{
  "query": "Huffman decompression",
  "limit": 10,
  "tags": ["engine", "memory"],
  "corpus": "vw_x64"
}
```

### Query Response

```json
{
  "results": [
    {
      "file": "src/decompress.c",
      "chunk_id": 42,
      "score": 0.89,
      "snippet": "void huffman_decompress(uint8_t *tree, ...)",
      "tags": ["engine", "memory"]
    }
  ],
  "total": 1,
  "index_stats": { "total_chunks": 46700, "total_files": 866 }
}
```

---

## Multi-Corpus

Separate indices for different knowledge domains:

| Corpus | Content |
|--------|---------|
| `vw_x64` | Main project source code |
| `vw_x64_source` | Decompiled source |
| `study_docs` | Research and progress reports |
| `dq_translation_tools` | DQ4 translation toolchain |
| `dq_cybergrime` | Debug and telemetry tools |
| `dqlosttranslation` | Translation project docs |

Configure corpora in `vw_nexus/data/corpus_config.json`.

---

## Optional: Vector Search

For semantic similarity search, install sentence-transformers:

```powershell
pip install sentence-transformers
```

Or via the project optional dependency:

```powershell
pip install -e ".[rag]"
```

Without it, VW RAG falls back to keyword-only FTS5 search — still highly effective.

---

## Performance

- **Indexing:** ~866 files, ~46,700 chunks in ~2 minutes
- **Query latency:** <50ms for keyword, <200ms with vector fusion
- **Storage:** ~340MB for full index (keyword + optional vectors)
- **WAL mode:** Concurrent read/write support

---

## Integration

- **VW Nexus** — RAG endpoints exposed through Nexus API
- **ZHARK** — Can query RAG for context during research cycles
- **FUBBU** — Router queries RAG for task context before delegation
- **VW Deck** — Deck UI has RAG query panel for manual search
- **MCP Tools** — `rag_query.yaml`, `rag_index.yaml`, `rag_status.yaml` expose RAG to IDEs



**LUX AURA**

*Autonomous agentic systems for the curious.*

[🌐 Bandcamp](https://luxaura.bandcamp.com) · [📘 Facebook](https://facebook.com/LuxAuraOfficial) · [💻 GitHub](https://github.com/luxauraofficial777) · [▶️ YouTube](https://youtube.com/c/LuxAuraOfficial)

</div>
