<div align="center">

# 🍃 Database Schema

[![Back](https://img.shields.io/badge/⬅️_Back_to_Architecture-1eb8e4?style=for-the-badge)](../docs/03-solution-architecture.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

> ⚠️ **Note:** This is a simplified representation. Actual schema utilizes MongoDB specifically, but represented here as ERD for clarity.

```mermaid
erDiagram
    USER ||--|| PROGRESS : has
    USER ||--o{ SESSION : maintains
    
    USER {
        ObjectId _id PK
        string email
        string displayName
        string avatarUrl
        string provider
        string providerId
        timestamp createdAt
        timestamp lastLogin
    }
    
    PROGRESS {
        ObjectId _id PK
        ObjectId userId FK
        object html_module
        object internet_module
        timestamp updatedAt
    }
    
    SESSION {
        string refreshToken
        ObjectId userId FK
        timestamp expiresAt
        string device
    }
```

## Schema Design Decisions

-   **Embedded Progress**: Instead of a separate `LessonCompletion` table, progress is embedded within a per-module structure in the `Progress` document. This allows fetching all progress for a module in a single query.
-   **Atomic Updates**: Because progress is structured as arrays (`completedLessons: [1, 2, 3]`), we can use MongoDB's `$addToSet` operator to safely merge progress from multiple devices without read-modify-write race conditions.
