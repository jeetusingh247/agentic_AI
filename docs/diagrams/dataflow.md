## Data Flow Diagram

```mermaid
flowchart LR
  subgraph User
    U[User Browser / SPA]
  end

  subgraph Frontend
    SPA[React SPA]
  end

  subgraph API
    API[FastAPI]
    Auth[Auth Service]
  end

  subgraph Queue
    RedisQ[Redis / Task Queue]
    Worker[Worker Pool]
  end

  subgraph Storage
    Postgres[(Postgres)]
    ObjectStore[(Object Storage)]
  end

  subgraph External
    LLM[LLM Provider]
    Webhook[External Webhook]
  end

  U -->|interact| SPA
  SPA -->|REST / WS| API
  API -->|auth| Auth
  API -->|CRUD| Postgres
  API -->|enqueue task| RedisQ
  RedisQ --> Worker
  Worker -->|LLM request| LLM
  Worker -->|store logs/results| ObjectStore
  Worker -->|update status| Postgres
  Worker -->|callback| Webhook

  %% Data considerations
  API ---|PII| ObjectStore
  API ---|PII| Postgres

```

Notes:
- User actions (create agent, start task) go through the SPA to the API; the API validates and stores config in Postgres.
- Long-running tasks are enqueued to Redis; workers process them and call LLM providers.
- Results and execution traces are written to Object Storage; summary/metadata stored in Postgres for quick access.
