# 04 - Key Features

> **Deep dive into AstroShan's major features and their technical implementation**

---

## 🌌 1. Adaptive 3D Galaxy Engine

### Overview

The landing page features a stunning 3D galaxy with up to 100,000 animated particles. This creates immediate visual impact while demonstrating the platform's technical sophistication.

### User Story

> As a visitor, I want to see an impressive visual experience that hints at the platform's quality, so that I'm motivated to explore further.

### Technical Implementation

**Architecture:**

```
┌─────────────────────────────────────────────────────────────────┐
│                 ADAPTIVE RENDERING PIPELINE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌────────────────┐                                            │
│   │ useDeviceTier  │  Detects hardware via                      │
│   │                │  navigator.hardwareConcurrency             │
│   └───────┬────────┘                                            │
│           │                                                      │
│    ┌──────┴──────┬──────────────┐                               │
│    │             │              │                                │
│   HIGH         MEDIUM         LOW                                │
│    │             │              │                                │
│    ▼             ▼              ▼                                │
│  ┌────────┐  ┌────────┐   ┌───────────────┐                     │
│  │ WebGL  │  │ WebGL  │   │ Canvas 2D     │                     │
│  │ 100k   │  │ 25k    │   │ StaticBackground│                   │
│  │particles│ │particles│  │ requestIdleCallback│                │
│  └────────┘  └────────┘   └───────────────┘                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Key Technologies:**
- `Three.js` via `@react-three/fiber`: 3D rendering
- `Web Workers`: Off-thread particle calculation
- `useDeviceTier` hook: Hardware detection
- `Canvas 2D API`: Fallback renderer

**Data Flow:**

1. Page loads → `useDeviceTier` runs
2. Hook checks `navigator.hardwareConcurrency`
3. Based on tier, either `InteractiveGalaxy` or `StaticBackground` renders
4. If WebGL, `useGalaxyWorker` spawns Web Worker
5. Worker calculates positions/colors for all particles
6. Main thread receives `Float32Array` (zero-copy transfer)
7. Three.js `BufferAttribute` updates with new data

**Challenges:**
- Initial implementation caused 14s TBT on mobile
- WebGL detection could be unreliable
- Memory usage with 100k particles

**Solution:**
- Web Worker for all GPU-bound calculation
- Robust WebGL detection with fallback
- Adaptive particle count based on screen width
- `requestIdleCallback` for deferred loading

### Code Highlights

```typescript
// Device tier detection - simplified
export function useDeviceTier(): DeviceTier {
  const [tier, setTier] = useState<DeviceTier>("low");

  useEffect(() => {
    const timer = setTimeout(() => {
      const cores = navigator.hardwareConcurrency || 4;
      if (cores >= 6) setTier("high");
      else if (cores >= 2) setTier("medium");
    }, 0);
    return () => clearTimeout(timer);
  }, []);

  return tier;
}
```

### Performance Considerations

| Metric | Before Optimization | After Optimization |
|--------|--------------------|--------------------|
| TBT (Mobile) | 14,000ms | 2,500ms |
| Particle generation | Main thread | Web Worker |
| Memory usage | Fixed 100k | Adaptive 12k-100k |

**Visual Diagram:**

![Web Worker Galaxy Engine](../diagrams/webworker_galaxy_fixed_1768603947661.webp)

## 🏠 2. CSS 3D Holographic House

### Overview

The BlueprintHouse is a visual progress indicator built entirely with CSS 3D transforms. As students complete lessons, the house builds itself – from foundation to roof to decorative details.

### User Story

> As a student, I want to see my progress visualized in an engaging way, so that I feel motivated to continue learning.

### Technical Implementation

**Architecture:**

The house uses CSS `transform-style: preserve-3d` to create genuine 3D perspective without WebGL.

```
BlueprintHouse (512 lines)
├── Container (perspective: 1000px)
│   ├── HoloBase (Foundation)
│   │   └── Visible at 0-15% progress
│   │
│   ├── HoloFace × 4 (Walls)
│   │   └── Visible at 15-55% progress
│   │
│   ├── HoloRoofFace × 2 (Roof panels)
│   │   └── Visible at 55-80% progress
│   │
│   └── Details + OrbitingRing
│       └── Visible at 80-100% progress
```

**Key Technologies:**
- `CSS transform-style: preserve-3d`: 3D space preservation
- `Framer Motion`: Opacity/scale transitions
- `MotionValue`: Progress-driven visibility
- `useTransform`: Smooth interpolation

**Why CSS 3D Instead of Three.js?**

| Consideration | CSS 3D | Three.js |
|---------------|--------|----------|
| Main thread load | Minimal | Heavy |
| GPU usage | Browser compositing | WebGL context |
| Device compatibility | Universal | Requires WebGL |
| Implementation complexity | Moderate | Higher |
| Visual quality | Good enough | Superior |

**Progress-Driven Visibility:**

```typescript
// Each component uses progress-driven opacity
const opacity = useTransform(
  progress,
  [triggerRange[0], triggerRange[1]],  // e.g., [0.15, 0.55]
  [0, 1]                                // fade in
);
```

### States

**Normal State:** Cyan/blue glow (#1eb8e4)
**Pending State:** Yellow/warning theme (student needs prerequisites)

### Performance Considerations

- Uses CSS compositing layer promotion (`will-change`)
- Animation pauses when lesson modal is open
- OrbitingRing uses CSS animations (not JS)

---

## ✅ 3. Live Code Validation Engine

### Overview

TaskChecklist analyzes student code in real-time, checking DOM structure and content against lesson requirements.

### User Story

> As a student writing code, I want immediate feedback on whether my code meets the requirements, so that I can correct mistakes while the context is fresh.

### Technical Implementation

**Architecture:**

```
┌────────────────────────────────────────────────────────────────┐
│                    VALIDATION PIPELINE                          │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Student Code         DOMParser            Validators          │
│   (string)       →     (parse)      →      (check)      →      │
│                                                                 │
│   "<h1>Hello</h1>"  →  Document  →  [                          │
│                                       { type: "exists",         │
│                                         selector: "h1",         │
│                                         passed: true },         │
│                                       { type: "content",        │
│                                         regex: /Hello/,        │
│                                         passed: true }          │
│                                     ]  →   UI Update            │
└────────────────────────────────────────────────────────────────┘
```

**Key Technologies:**
- `DOMParser API`: Safe HTML parsing (no XSS risk)
- `Regex validators`: Content checking
- `React state`: Live UI updates

**Validation Types:**

| Type | Description | Example |
|------|-------------|---------|
| `elementExists` | Check if element exists | `<header>` present? |
| `elementContent` | Check text content | `<title>` contains "Portfolio"? |
| `attributeExists` | Check for attribute | `<img>` has `alt`? |
| `attributeValue` | Check attribute value | `lang="ar"`? |

**Progressive Hints:**

When validation fails, hints reveal progressively:
1. First attempt: General hint
2. After 30 seconds: More specific hint
3. After 60 seconds: Nearly complete solution

### Gatekeeper Integration

```typescript
// Tasks lock if student isn't authorized
<TaskChecklist
  requirements={taskRequirements}
  code={studentCode}
  onValidationChange={setIsValid}
  isAuthorized={isAuthorized}  // from useGatekeeper
/>
```

---

## 📝 4. Cumulative Mission System

### Overview

Unlike isolated exercises, AstroShan's missions build on each other. Code from one mission carries forward to the next, culminating in a complete portfolio website.

### User Story

> As a student, I want my code to persist across lessons, so that I build a real project instead of isolated snippets.

### Technical Implementation

**Backward Task Discovery Algorithm:**

```
┌─────────────────────────────────────────────────────────────────┐
│                 CODE LOADING DECISION TREE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   CodeEditorView.loadCode()                                      │
│        │                                                         │
│        ▼                                                         │
│   ┌────────────────────────────┐                                │
│   │ Check saved progress for   │                                │
│   │ current task ID            │                                │
│   └────────────┬───────────────┘                                │
│                │                                                 │
│       ┌────────┴────────┐                                       │
│       │                 │                                        │
│   [Found]           [Not Found]                                  │
│       │                 │                                        │
│       ▼                 ▼                                        │
│   Return saved    ┌────────────────────────────┐                │
│   code            │ Find previous mission      │                │
│                   │ in same module             │                │
│                   └────────────┬───────────────┘                │
│                                │                                 │
│                       ┌────────┴────────┐                       │
│                       │                 │                        │
│                   [Found]           [Not Found]                  │
│                       │                 │                        │
│                       ▼                 ▼                        │
│                   Return            Return                       │
│                   previous          starterCode                  │
│                   code                                           │
└─────────────────────────────────────────────────────────────────┘
```

**Auto-Save:**
- Debounced (500ms delay)
- localStorage primary
- MongoDB sync if authenticated

**Mission Sequence:**
- L24: Foundation layout
- L36: Gallery & Services
- L50: Semantic refactor
- L57: Portfolio launch (capstone)

**Visual Diagram:**

![Backward Task Discovery](../diagrams/backward_task_discovery_bilingual_1768602050087.webp)

## 🔐 5. Dual-Token Authentication System

### Overview

Enterprise-grade JWT security with seamless UX. Uses short-lived access tokens (memory-only) and long-lived refresh tokens (HttpOnly cookies).

### User Story

> As a user, I want my session to persist without interruption, while knowing my account is secure.

### Technical Implementation

**Token Architecture:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    DUAL TOKEN SYSTEM                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ACCESS TOKEN                    REFRESH TOKEN                  │
│   ├── Lifetime: 15 minutes        ├── Lifetime: 7 days          │
│   ├── Storage: In-memory only     ├── Storage: HttpOnly cookie  │
│   ├── Contains: userId, email     ├── Contains: userId          │
│   └── Used for: API calls         └── Used for: Token refresh   │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   SECURITY PROPERTIES:                                           │
│   ✅ XSS Protection: Access token never in localStorage         │
│   ✅ CSRF Protection: Refresh cookie is SameSite=Strict         │
│   ✅ Theft Mitigation: 15min window even if token stolen        │
│   ✅ Silent Rotation: User never sees token refresh             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Silent Refresh Flow:**

```
API Call → 401 Unauthorized
     │
     ▼
AuthStore.refreshAccessToken()
     │
     ├── POST /api/auth/token/refresh
     │   (cookie sent automatically)
     │
     ▼
┌────────────────────────────┐
│ Server verifies refresh    │
│ token, issues new access   │
│ token in response body     │
└────────────┬───────────────┘
             │
             ▼
Store in memory → Retry original request
```

**Deduplication:**
```typescript
// Prevents multiple concurrent refresh attempts
if (this.refreshPromise) return this.refreshPromise;

this.refreshPromise = (async () => {
  // ... refresh logic
})();
```

**Visual Diagram:**

![Anonymous-First Auth](../diagrams/anonymous_first_auth_1768725256945.webp)

## 🧩 6. Interactive Widget Library

### Overview

35+ custom React components that visualize abstract concepts through physics simulations, drag-drop puzzles, and interactive diagrams.

### Widget Categories

**Physics Simulations:**
- `TextMagnetSim`: RTL/LTR magnetic field visualization
- `TLSHandshakeSim`: Security handshake animation

**Visual Explainers:**
- `CommentGoggles`: X-Ray vision for HTML comments
- `SkeletonBuilder`: HTML5 boilerplate puzzle
- `SemanticElevator`: Heading hierarchy skyscraper

**Domain Simulators:**
- `DNSSimulator`: DNS lookup animation
- `ClientServerSim`: Request/Response lifecycle
- `FileExplorerSim`: File path navigation

### Widget UI Kit

Consistent styling and accessibility across all widgets:

```
WidgetContainer
├── WidgetHeader (title, icon, reset button)
├── [Interactive Content]
└── WidgetSuccessState (completion animation)
```

**Accessibility:**
- Full keyboard navigation
- `WidgetToggleGroup` for tab-like selection
- Screen reader compatible labels

---

## 🎭 7. Context-Aware Hero

### Overview

The landing page hero dynamically adapts based on user state – showing different content, CTAs, and animations for each journey phase.

### Variants

| State | Detection Logic | Content |
|-------|-----------------|---------|
| **Guest** | No session | "Start your journey" |
| **Guest + Progress** | Anonymous + localStorage data | "Your progress is saved locally" |
| **New Explorer** | Authenticated + 0 lessons | "Welcome, Captain [Name]!" |
| **In Progress** | Authenticated + some completion | "Continuing journey, X% done" |
| **Bridge Traveler** | Internet done, HTML not started | "Ready for HTML?" |
| **Course Master** | Module completed | "Planet conquered!" |

### Hydration-Safe Implementation

Hero text renders server-side for LCP, then client updates based on auth state:

```typescript
// Server: Render default hero text
// Client: useEffect checks auth state, updates variant
useEffect(() => {
  const determineVariant = async () => {
    const session = await authStore.getSession();
    const progress = await progressStore.getOverallProgress();
    
    if (!session.isAuthenticated) {
      if (progress.totalCompleted > 0) return "guestWithProgress";
      return "guest";
    }
    // ... more logic
  };
}, []);
```

---

## 🔗 Related Documents

- [← Solution Architecture](03-solution-architecture.md)
- [Technical Decisions →](05-technical-decisions.md)
- [Challenges & Solutions →](06-challenges-solutions.md)

---

*Next: [05 - Technical Decisions](05-technical-decisions.md)*
