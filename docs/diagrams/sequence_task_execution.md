## Sequence Diagram — Task Execution & LLM Calls

```mermaid
sequenceDiagram
  participant User
  participant SPA
  participant API
  participant Queue
  participant Worker
  participant LLM
  participant Postgres
  participant ObjectStore

  User->>SPA: Click "Run" on agent
  SPA->>API: POST /agents/:id/run
  API->>Postgres: create task record (queued)
  API->>Queue: enqueue task
  API-->>SPA: 202 Accepted {task_id}

  Queue->>Worker: deliver task
  Worker->>Postgres: update task status=running, start_time
  Worker->>LLM: stream prompt (possibly multi-part)
  LLM-->>Worker: stream partial responses
  Worker->>ObjectStore: append logs / partial outputs
  Worker->>API: emit WebSocket events (progress)
  Worker->>LLM: finalize / request completion
  LLM-->>Worker: final output
  Worker->>ObjectStore: store final artifacts
  Worker->>Postgres: update task status=completed, result pointer
  Worker-->>API: notify completion (webhook / event)
  API-->>SPA: push final state (via WS) or client polls

  alt transient LLM error
    Worker->>LLM: retry with backoff
    Worker->>Postgres: update retry_count
  end

  alt permanent failure
    Worker->>Postgres: update status=failed, error metadata
    Worker->>ObjectStore: store error trace
    Worker-->>API: emit failure event
  end

```

Notes:
- Workers stream LLM responses to Object Storage and emit progress via WebSocket or server-sent events for real-time UI updates.
- Retries and circuit breaker patterns protect against flaky LLM providers.
