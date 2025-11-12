## Deployment Diagram (Phase 1 - Managed Services)

```mermaid
graph TD
  subgraph Users
    Browser[User Browser]
  end

  subgraph CDN
    Static[Static Assets CDN (Vercel / Cloud CDN)]
  end

  subgraph Managed
    CloudRun[Cloud Run / App Runner / Fargate (FastAPI API)]
    CloudRunJobs[Cloud Run Jobs / Fargate Tasks (Workers)]
    CloudSQL[(Managed Postgres - Cloud SQL / RDS)]
    Memorystore[(Managed Redis - Memorystore / ElastiCache)]
    Storage[(Managed Object Storage - GCS / S3)]
    SecretMgr[Secret Manager]
    Logging[Cloud Logging / CloudWatch]
    Monitoring[Cloud Monitoring / Grafana]
  end

  Browser --> Static
  Static --> CloudRun
  CloudRun --> CloudSQL
  CloudRun --> Memorystore
  CloudRun --> Storage
  CloudRun --> SecretMgr
  CloudRunJobs --> Memorystore
  CloudRunJobs --> CloudSQL
  CloudRunJobs --> Storage
  CloudRunJobs --> SecretMgr
  CloudRun --> Logging
  CloudRunJobs --> Logging
  Logging --> Monitoring

  classDef managed fill:#e6ffed,stroke:#2a8f53;
  class CloudRun,CloudRunJobs,CloudSQL,Memorystore,Storage,SecretMgr,Logging,Monitoring managed;

```

Notes / Rationale:
- Phase 1 uses managed container services (Cloud Run / AWS Fargate) to reduce ops burden and enable autoscaling.
- Use managed Postgres and Redis for reliability and minimal operational overhead.
- Store large logs/artifacts in object storage. Secrets stored in cloud Secret Manager.
- Observability uses cloud logging and monitoring; Sentry can be added for error tracking.
