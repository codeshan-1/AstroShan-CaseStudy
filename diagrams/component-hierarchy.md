<div align="center">

# 🧩 Component Hierarchy

[![Back](https://img.shields.io/badge/⬅️_Back_to_Architecture-1eb8e4?style=for-the-badge)](../docs/03-solution-architecture.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

```mermaid
graph TD
    APP[Next.js App]
    APP --> LAYOUT[RootLayout]
    LAYOUT --> AUTH_PROV[AuthProvider]
    AUTH_PROV --> UI_ROOT[UI Root]
    
    UI_ROOT --> NAV[Navigation]
    UI_ROOT --> PAGE_CONTENT[Page Content]
    UI_ROOT --> FOOTER[Footer]
    
    PAGE_CONTENT --> LANDING[Landing Page]
    PAGE_CONTENT --> LESSON_LO[Lesson Layout]
    PAGE_CONTENT --> PROFILE[Profile Page]
    
    LANDING --> HERO[ContextAwareHero]
    LANDING --> GALAXY[GalaxyBackground]
    LANDING --> FEATURES[FeatureGrid]
    
    LESSON_LO --> SIDEBAR[LessonSidebar]
    LESSON_LO --> CONTENT[LessonContent]
    LESSON_LO --> EDITOR[CodeEditor]
    LESSON_LO --> PREVIEW[LivePreview]
    
    SIDEBAR --> PROGRESS[ProgressRing]
    SIDEBAR --> LIST[LessonList]
    
    EDITOR --> MONACO[MonacoWrapper]
    PREVIEW --> CHECKLIST[TaskChecklist]
    PREVIEW --> HOUSE[BlueprintHouse]
    
    PROFILE --> STATS[UserStats]
    PROFILE --> CERT[Certificate]
    
    classDef page fill:#e3f2fd,stroke:#1976d2
    classDef layout fill:#fff3e0,stroke:#e65100
    classDef component fill:#f3e5f5,stroke:#7b1fa2
    classDef complex fill:#e8f5e9,stroke:#2e7d32
    
    class LANDING,PROFILE page
    class LAYOUT,LESSON_LO,SIDEBAR layout
    class NAV,FOOTER,HERO,FEATURES,CONTENT,PROGRESS,LIST,STATS,CERT component
    class GALAXY,EDITOR,PREVIEW,MONACO,CHECKLIST,HOUSE,AUTH_PROV complex
```

## Key Components

-   **GalaxyBackground**: Complex 3D component managed by `useGalaxyWorker` and `useDeviceTier`.
-   **CodeEditor**: Wraps the Monaco Editor engine with custom themes and configuration for mobile optimizations.
-   **BlueprintHouse**: CSS 3D feature that reacts to progress state.
-   **TaskChecklist**: Real-time validation engine component using DOM parsing.
