<div align="center">

# 🏗️ System Architecture

[![Back](https://img.shields.io/badge/⬅️_Back_to_Diagrams-1eb8e4?style=for-the-badge)](../docs/03-solution-architecture.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web Application<br/>Next.js 16 + React]
        MOBILE[Mobile View<br/>Responsive Design]
    end
    
    subgraph "CDN & Edge"
        CDN[Vercel Edge Network<br/>Static Assets]
        EDGE[Edge Middleware<br/>Auth & Routing]
    end
    
    subgraph "Application Layer"
        API[Next.js API Routes<br/>Serverless Functions]
        AUTH[Auth Service<br/>JWT + Google OAuth]
        WORKER[Web Workers<br/>Galaxy Generation]
    end
    
    subgraph "Data Layer"
        DB[(MongoDB Atlas<br/>Primary Database)]
        LOCAL[LocalStorage<br/>Offline Progress]
    end
    
    subgraph "Storage"
        CLOUDINARY[Cloudinary<br/>Media Assets]
    end
    
    WEB --> EDGE
    MOBILE --> EDGE
    EDGE --> CDN
    EDGE --> API
    WEB --> WORKER
    API --> AUTH
    API --> DB
    WEB --> LOCAL
    LOCAL -.->|Sync| API
    WEB --> CLOUDINARY
    
    classDef client fill:#e1f5ff,stroke:#01579b
    classDef api fill:#fff9c4,stroke:#f57f17
    classDef data fill:#f3e5f5,stroke:#4a148c
    classDef infra fill:#e8f5e9,stroke:#1b5e20
    
    class WEB,MOBILE,WORKER client
    class API,AUTH api
    class DB,LOCAL data
    class CDN,EDGE,CLOUDINARY infra
```

## Architecture Overview

AstroShan utilizes a modern **Jamstack architecture** built on **Next.js**, leveraging serverless functions and edge middleware for scalability and performance. The system is designed to be **offline-first** for learning progress, syncing to the cloud when connectivity allows.

### Key Components

1.  **Client Layer**: A responsive Next.js application that adapts rendering strategies (WebGL vs. Canvas) based on device capabilities detected at runtime.
2.  **Edge Middleware**: Handles authentication checks and routing decisions at the network edge, reducing latency for global users.
3.  **Application Layer**: Serverless API routes handle business logic, progress synchronization, and profile management.
4.  **Data Layer**: A hybrid approach using MongoDB Atlas for persistent cloud storage and LocalStorage for immediate, offline access to course progress.
5.  **Web Workers**: Offloads heavy computational tasks (like galaxy particle generation) to background threads to keep the main UI thread responsive.

### Scalability & Performance

-   **Serverless**: Automatic scaling of API routes based on demand.
-   **Static Generation (SSG)**: Core lesson content is pre-rendered for maximum speed.
-   **CDN Caching**: Static assets and media serve from the edge.
-   **Adaptive Rendering**: Reduces GPU load on lower-end devices.

### Security

-   **Dual-Token Auth**: Secure, short-lived access tokens (in-memory) and HttpOnly refresh cookies.
-   **Edge Protection**: Middleware verifies tokens before requests reach origin functions.
