## Component Diagram

```mermaid
graph TD
  subgraph User
    Browser[User Browser / SPA]
  end

  subgraph Frontend
    SPA[React + TypeScript SPA]
    CDN[CDN / Static Hosting]
  end

  subgraph Backend
    API[FastAPI (REST / WebSocket) API]
    Auth[Auth Service / JWT / OAuth]
    API_GW[API Gateway / Ingress]
  end

  subgraph Workers
    Queue[Redis / Task Queue]
    Worker[Worker Pool (task executor)]
    Scheduler[Scheduler / Cron Jobs]
  end

  subgraph Data
    Postgres[(PostgreSQL)]
    Redis[(Redis - cache/queue)]
    ObjectStore[(Object Storage - logs & artifacts)]
  end

  subgraph Integrations
    LLM[LLM Providers (OpenAI, Anthropic, etc.)]
    Webhooks[External Webhooks / Tooling]
    Email[Email Provider]
  end

  subgraph Observability
    Metrics[Prometheus / Cloud Metrics]
    Traces[OpenTelemetry / Tracing]
    Errors[Sentry]
  end

  Browser -->|HTTPS| CDN
  CDN --> SPA
  SPA -->|HTTPS| API_GW
  API_GW --> API
  API --> Auth
  API --> Postgres
  API --> Redis
  API --> ObjectStore
  API -->|enqueue| Queue
  Queue --> Worker
  Worker -->|call| LLM
  Worker --> Postgres
  Worker --> ObjectStore
  Worker --> Webhooks
  API --> Metrics
  API --> Traces
  Worker --> Traces
  API --> Errors
  Worker --> Errors

  classDef infra fill:#f9f,stroke:#333,stroke-width:1px;
  class Postgres,Redis,ObjectStore infra;

```

Notes:
- The SPA communicates with the API via an API Gateway for routing, authentication, and rate-limiting.
- The API persists configuration, user and agent metadata to Postgres and uses Redis for caching and the task queue.
- Workers dequeue tasks and call LLM providers; results and execution traces are stored in Object Storage and Postgres.
