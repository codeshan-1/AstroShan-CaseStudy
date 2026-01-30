<div align="center">

# ⚡ 07 - Performance Optimization

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=82%25+TBT+Reduction;Performance+Engineering;Optimization+Strategies" alt="Typing"/>

> **How we achieved 82% Total Blocking Time reduction**

<br/>

[![Prev](https://img.shields.io/badge/←_Challenges-1eb8e4?style=for-the-badge)](06-challenges-solutions.md)
[![Next](https://img.shields.io/badge/Testing_→-7565e3?style=for-the-badge)](08-testing-quality.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 📊 Initial Baseline

### Before Optimization

| Metric | Value | Target | Status |
|:------:|:-----:|:------:|:------:|
| TBT | 14,000ms | < 3,000ms | ❌ Failed |
| FCP | 1.8s | < 1.0s | ⚠️ Warning |
| LCP | 2.4s | < 2.5s | ⚠️ Borderline |
| TTI | 16s | < 5s | ❌ Failed |

### 🔍 Bottleneck Identification

Chrome DevTools revealed:
- **72% TBT**: `generateParticles()` function (galaxy)
- **15% TBT**: Framer Motion bundle
- **8% TBT**: Monaco Editor initialization
- **5% TBT**: Other

---

## ⚡ Optimization 1: Web Worker Galaxy

### Problem
100k particle generation blocked main thread for 14 seconds.

### Solution

```typescript
// useGalaxyWorker.ts
const worker = useMemo(() => 
  new Worker(new URL("../workers/galaxy.worker.ts", import.meta.url)),
[]);

useEffect(() => {
  worker.postMessage(params);
  worker.onmessage = (e) => {
    // Zero-copy transfer with Transferable
    updatePositions(e.data);
  };
}, []);
```

### Results

| Before | After | Improvement |
|:------:|:-----:|:-----------:|
| 14,000ms | 2,500ms | **-82%** |

---

## 🎬 Optimization 2: LazyMotion

### Problem
Framer Motion adds 30KB+ to bundle.

### Solution

```typescript
// layout.tsx
import { LazyMotion, domAnimation } from "framer-motion";

<LazyMotion features={domAnimation} strict>
  {children}
</LazyMotion>
```

```typescript
// Components
import { m } from "framer-motion";
<m.div animate={{ opacity: 1 }} /> // Not `motion.div`
```

### Results
- Tree-shaking enabled
- Only used features loaded
- Strict mode catches violations

---

## 📦 Optimization 3: Dynamic Imports

### Problem
Heavy components loaded on initial render.

### Solution

```typescript
// Lazy load heavy components
const Scene = dynamic(() => import("@/components/landing/Scene"), {
  ssr: false,
  loading: () => <SceneSkeleton />
});

const Monaco = dynamic(() => import("@monaco-editor/react"), {
  ssr: false
});
```

### next.config.ts

```typescript
experimental: {
  optimizePackageImports: ["@react-three/drei", "react-icons", "framer-motion"]
}
```

---

## 🖥️ Optimization 4: SSR for LCP

### Problem
Hero text waited for JavaScript, hurting LCP.

### Solution

```typescript
// Hero text renders on server
export default function Hero() {
  return (
    <h1>Start Your Coding Journey</h1> // Available immediately
  );
}
```

```typescript
// Client-side personalization after hydration
useEffect(() => {
  const session = await getSession();
  if (session.name) setTitle(`Welcome, ${session.name}`);
}, []);
```

### Results
- LCP: ~0ms for text
- Progressive enhancement

---

## 🎨 Optimization 5: Critical CSS

### Critters Integration

```javascript
// next.config.ts
const withCritters = require("critters-webpack-plugin");

module.exports = withCritters({
  preload: "swap",
  pruneSource: false
});
```

### Benefits
- Above-fold CSS inlined
- Remaining CSS loaded async
- Faster FCP

---

## 📊 Final Results

| Metric | Before | After | Improvement |
|:------:|:------:|:-----:|:-----------:|
| **TBT** | 14,000ms | 2,500ms | **-82%** ✅ |
| **FCP** | 1.8s | 1.0s | **-44%** ✅ |
| **LCP** | 2.4s | ~0s (SSR) | **-100%** ✅ |
| **TTI** | 16s | 4s | **-75%** ✅ |

---

## 🎓 Key Learnings

1. **Measure before optimizing** - Profiling revealed surprising bottlenecks
2. **Web Workers are underutilized** - Any CPU-bound work benefits
3. **Progressive enhancement works** - Server-render first, enhance on client
4. **Bundle analysis is essential** - Know what you're shipping

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 Related Documents

| Navigation |
|:----------:|
| [💡 ← Challenges](06-challenges-solutions.md) |
| [🧪 Testing & Quality →](08-testing-quality.md) |

---

*Next: [08 - Testing & Quality](08-testing-quality.md)*
