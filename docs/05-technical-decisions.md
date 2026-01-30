<div align="center">

# 🔧 05 - Technical Decisions

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Architecture+Decision+Records;Design+Rationale;Trade-off+Analysis" alt="Typing"/>

> **Architecture Decision Records (ADRs) documenting major technical choices**

<br/>

[![Prev](https://img.shields.io/badge/←_Key_Features-1eb8e4?style=for-the-badge)](04-key-features.md)
[![Next](https://img.shields.io/badge/Challenges_→-7565e3?style=for-the-badge)](06-challenges-solutions.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 📋 ADR Index

| ID | Decision | Status |
|:--:|:---------|:------:|
| ADR-001 | CSS 3D over Additional WebGL | ✅ Accepted |
| ADR-002 | Web Workers for Galaxy Particles | ✅ Accepted |
| ADR-003 | Anonymous-First Authentication | ✅ Accepted |
| ADR-004 | MongoDB over Other Databases | ✅ Accepted |
| ADR-005 | LazyMotion for Animation Optimization | ✅ Accepted |

---

## 🏠 ADR-001: CSS 3D for BlueprintHouse

### Context
Need a visual progress indicator. Options: WebGL (Three.js) or CSS 3D transforms.

### Decision
**Use CSS 3D** (`transform-style: preserve-3d`)

### Rationale

| Factor | CSS 3D | Three.js |
|:-------|:------:|:--------:|
| Main thread load | ✅ Minimal | ❌ Heavy |
| Device compatibility | ✅ Universal | ⚠️ WebGL required |
| Implementation | ⚠️ Moderate | ❌ Complex |
| Visual quality | ✅ Sufficient | ✅ Superior |

### Consequences
- ✅ Single WebGL context (galaxy only)
- ✅ Better low-end device support
- ⚠️ Limited to simpler 3D geometry

---

## ⚡ ADR-002: Web Workers for Particles

### Context
Generating 100,000 particle positions blocks main thread for 14 seconds on mobile.

### Decision
**Offload particle calculation to Web Worker**

### Implementation
```typescript
// Main thread
const worker = new Worker(new URL("./galaxy.worker.ts", import.meta.url));
worker.postMessage(params);
worker.onmessage = (e) => updateBufferAttribute(e.data);

// Worker thread
self.onmessage = (e) => {
  const positions = generateParticles(e.data);
  self.postMessage(positions, [positions.buffer]); // Zero-copy transfer
};
```

### Consequences
- ✅ 82% TBT reduction (14s → 2.5s)
- ⚠️ Added async complexity
- ⚠️ Worker bundling configuration needed

---

## 🔓 ADR-003: Anonymous-First Authentication

### Context
Traditional platforms require registration before use, creating friction.

### Decision
**Full access without registration**, using UUID-based anonymous sessions with opt-in OAuth.

### Flow
```
First Visit → UUID generated → Full access (localStorage)
     ↓
Optional: Google OAuth → 3-way merge → Cloud sync enabled
```

### 3-Way Merge Strategy

| Data Type | Merge Strategy |
|:----------|:---------------|
| Completed lessons | Union (∪) |
| Quiz scores | Maximum |
| Timestamps | Latest |

### Consequences
- ✅ Zero-friction entry
- ✅ No lost progress on registration
- ⚠️ More complex sync logic

---

## 🍃 ADR-004: MongoDB Over Alternatives

### Context
Need persistent storage for user progress.

### Alternatives Considered

| Option | Pros | Cons |
|:-------|:-----|:-----|
| Firebase | Easy auth | Vendor lock-in |
| PostgreSQL | Relational | Schema rigidity |
| **MongoDB** | Flexible schema | Requires pooling |

### Decision
**MongoDB Atlas (serverless tier)**

### Consequences
- ✅ Flexible schema for evolving data
- ✅ Native JSON operations
- ✅ Generous free tier
- ⚠️ Requires connection pooling singleton

---

## 🎬 ADR-005: LazyMotion Optimization

### Context
Framer Motion adds significant bundle weight (~30KB).

### Decision
**Use LazyMotion with domAnimation features** and `m` components.

### Implementation
```typescript
// app/layout.tsx
<LazyMotion features={domAnimation} strict>
  {children}
</LazyMotion>

// Components use `m` instead of `motion`
import { m } from "framer-motion";
<m.div animate={{ opacity: 1 }} />
```

### Consequences
- ✅ Reduced bundle size
- ✅ Tree-shaking enabled
- ⚠️ Must use `m` consistently

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 Related Documents

| Navigation |
|:----------:|
| [✨ ← Key Features](04-key-features.md) |
| [💡 Challenges & Solutions →](06-challenges-solutions.md) |

---

*Next: [06 - Challenges & Solutions](06-challenges-solutions.md)*
