<div align="center">

# 🚀 Deployment Pipeline

[![Back](https://img.shields.io/badge/⬅️_Back_to_Deployment-1eb8e4?style=for-the-badge)](../docs/09-deployment.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

```mermaid
graph LR
    START((Code Push)) --> LINT[ESLint & Prettier]
    LINT --> BUILD[Next.js Build]
    BUILD --> TEST[Unit Tests (Jest)]
    TEST --> VERCEL[Deploy to Vercel Preview]
    VERCEL --> E2E[E2E Tests (Playwright)]
    
    E2E -->|Success| PROD[Promote to Production]
    E2E -->|Failure| NOTIFY[Notify Developer]
    
    PROD --> EDGE[Edge Middleware Deploy]
    PROD --> STATIC[Static Assets to CDN]
    PROD --> FUNCS[Serverless Functions]
    
    classDef check fill:#fff3e0,stroke:#ef6c00
    classDef deploy fill:#e3f2fd,stroke:#1565c0
    classDef infra fill:#e8f5e9,stroke:#2e7d32
    
    class LINT,BUILD,TEST,E2E check
    class VERCEL,PROD deploy
    class EDGE,STATIC,FUNCS infra
```

## Pipeline Overview

The deployment pipeline leverages Vercel's integrated CI/CD:

1.  **Validation**: Every commit triggers linting and a production build check to ensure type safety.
2.  **Preview**: Pull requests automatically deploy to a unique preview URL.
3.  **Testing**: E2E tests run against the preview deployment to verify critical flows (Login, Lesson Loading) before merging.
4.  **Production**: Merging to `main` triggers a zero-downtime deployment to the global edge network.
