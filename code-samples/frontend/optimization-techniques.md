<div align="center">

# ⚡ Optimization Techniques

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=82%25+TBT+Reduction;Performance+Patterns" alt="Typing"/>

> **Performance patterns that achieved 82% TBT reduction in AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## ⚡ 1. Adaptive Rendering Pipeline

### Context

**Feature:** Device-appropriate 3D experience  
**Complexity:** High  
**Technologies:** React, Web Workers, Canvas API

**The Challenge:**  
Deliver stunning visuals on high-end devices while maintaining usability on low-end devices.

---

### The Problem

**Initial State:**
- 100k particles rendered on all devices
- 14-second Total Blocking Time on mobile
- Users on budget phones couldn't interact

**Goal:**
- Smooth experience across all devices
- No compromise on visual quality for capable devices
- Graceful degradation for constrained devices

---

### The Solution: 3-Tier Rendering

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes more nuanced tier detection.
 */

// Tier selection based on device capabilities
function selectRenderer(tier: DeviceTier): React.ComponentType {
  const renderers = {
    high: () => import('./renderers/WebGLFull'),      // 100k particles
    medium: () => import('./renderers/WebGLReduced'), // 25k particles
    low: () => import('./renderers/Canvas2D'),        // Static fallback
  };
  
  return lazy(renderers[tier]);
}

// Component that selects appropriate renderer
function AdaptiveBackground() {
  const tier = useDeviceTier();
  const Renderer = useMemo(() => selectRenderer(tier), [tier]);
  
  return (
    <Suspense fallback={<StaticGradient />}>
      <Renderer />
    </Suspense>
  );
}
```

### Key Optimization Details

```typescript
// Particle count adapts to screen width
function getParticleCount(tier: DeviceTier): number {
  const baseCount = {
    high: 100000,
    medium: 25000,
    low: 0,  // Canvas fallback uses different approach
  };
  
  // Further reduce on small screens
  if (typeof window !== 'undefined' && window.innerWidth < 768) {
    return Math.floor(baseCount[tier] * 0.5);
  }
  
  return baseCount[tier];
}
```

---

### Results

| Metric | Before | After |
|--------|--------|-------|
| TBT (high-end) | 14,000ms | 2,500ms |
| TBT (low-end) | 14,000ms+ | 500ms |
| User satisfaction | Poor | High |

---

## 🚀 2. Code Splitting & Lazy Loading

### Context

**Feature:** Fast initial load with on-demand features  
**Complexity:** Medium  
**Technologies:** Next.js, React.lazy, dynamic imports

**The Challenge:**  
Heavy dependencies (Monaco Editor, Three.js) shouldn't block initial page load.

---

### Implementation

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes error boundaries and loading states.
 */

import dynamic from 'next/dynamic';

// 1. SSR-disabled dynamic import for browser-only components
const MonacoEditor = dynamic(
  () => import('@monaco-editor/react'),
  { 
    ssr: false,  // Monaco needs browser APIs
    loading: () => <EditorSkeleton />,
  }
);

// 2. Lazy load heavy 3D components
const InteractiveGalaxy = dynamic(
  () => import('./3d/InteractiveGalaxy'),
  { 
    ssr: false,
    loading: () => <StaticBackground />,
  }
);

// 3. Deferred loading with requestIdleCallback
function DeferredComponent({ 
  loader, 
  fallback 
}: { 
  loader: () => Promise<React.ComponentType>;
  fallback: React.ReactNode;
}) {
  const [Component, setComponent] = useState<React.ComponentType | null>(null);

  useEffect(() => {
    const loadComponent = async () => {
      const Loaded = await loader();
      setComponent(() => Loaded);
    };

    // Wait for idle time before loading
    if ('requestIdleCallback' in window) {
      requestIdleCallback(() => loadComponent(), { timeout: 2000 });
    } else {
      // Fallback for Safari
      setTimeout(loadComponent, 200);
    }
  }, [loader]);

  if (!Component) return <>{fallback}</>;
  return <Component />;
}
```

### Usage

```typescript
function LandingPage() {
  return (
    <>
      {/* Critical content loads immediately */}
      <Hero />
      
      {/* Heavy 3D loads after idle */}
      <DeferredComponent
        loader={() => import('./3d/Galaxy').then(m => m.Galaxy)}
        fallback={<StaticGradient />}
      />
      
      {/* Below-fold content lazy loaded */}
      <Suspense fallback={<SectionSkeleton />}>
        <AboutSection />
      </Suspense>
    </>
  );
}
```

---

### Key Techniques

#### 1. Route-based Code Splitting

```typescript
// Next.js App Router automatically splits by route
// app/html/page.tsx → separate chunk
// app/internet/page.tsx → separate chunk
```

#### 2. Component-level Splitting

```typescript
const HeavyWidget = dynamic(() => import('./widgets/HeavyWidget'));
```

#### 3. Package-level Optimization

```typescript
// next.config.ts
export default {
  experimental: {
    optimizePackageImports: [
      '@react-three/drei',
      '@react-three/fiber',
      'react-icons',
      'framer-motion',
    ],
  },
};
```

---

## 💨 3. LazyMotion for Framer Motion

### Context

**Feature:** Smooth animations with minimal bundle impact  
**Complexity:** Low  
**Technologies:** Framer Motion

**The Challenge:**  
Framer Motion is powerful but adds significant bundle size. Need animations without the weight.

---

### Implementation

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 */

import { LazyMotion, domAnimation, m } from 'framer-motion';

// 1. Root layout wraps with LazyMotion
function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <LazyMotion features={domAnimation} strict>
      {children}
    </LazyMotion>
  );
}

// 2. Use `m` instead of `motion` for lightweight components
function AnimatedCard({ children }: { children: React.ReactNode }) {
  return (
    <m.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      transition={{ duration: 0.3 }}
    >
      {children}
    </m.div>
  );
}
```

### Bundle Impact

| Approach | Bundle Size |
|----------|-------------|
| Full `motion` | ~50kb |
| `LazyMotion` + `m` | ~15kb |
| Savings | **~70%** |

---

## 🎯 4. Critical CSS & LCP Optimization

### Context

**Feature:** Fast First Contentful Paint  
**Complexity:** Medium  
**Technologies:** Next.js, Critters

**The Challenge:**  
Ensure above-fold content renders immediately without waiting for full CSS.

---

### Implementation

```typescript
// next.config.ts
export default {
  experimental: {
    optimizeCss: true,  // Enables Critters for critical CSS extraction
  },
};
```

### Server-Side LCP Text

```typescript
// Hero renders with server-provided text
function Hero() {
  return (
    <>
      {/* Server-rendered for LCP */}
      <h1 id="hero-headline">
        Start Your Space Journey
      </h1>
      
      {/* Enhanced on client */}
      <ClientEnhancements />
    </>
  );
}
```

### CSS Containment

```css
/* Improve rendering performance */
.section {
  content-visibility: auto;
  contain-intrinsic-size: auto 500px;
}

/* Force GPU layer for animations */
.animated-element {
  will-change: transform, opacity;
  transform: translateZ(0);
}
```

---

## 🔄 5. Memoization Strategies

### Context

**Feature:** Prevent unnecessary re-renders  
**Complexity:** Medium  
**Technologies:** React, useMemo, useCallback, React.memo

**The Challenge:**  
Complex components (lesson lists, validation results) need careful optimization.

---

### Implementation

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Apply memoization judiciously - not everywhere.
 */

// 1. useMemo for expensive computations
function LessonList({ lessons, progress }: Props) {
  const enrichedLessons = useMemo(() => {
    return lessons.map(lesson => ({
      ...lesson,
      isComplete: progress.completedLessons.includes(lesson.id),
      canAccess: isLessonAccessible(lesson, progress),
    }));
  }, [lessons, progress]);  // Only recalculate when these change

  return <List items={enrichedLessons} />;
}

// 2. useCallback for stable function references
function ParentComponent() {
  const [items, setItems] = useState([]);
  
  const handleItemClick = useCallback((id: number) => {
    setItems(prev => prev.map(item => 
      item.id === id ? { ...item, selected: true } : item
    ));
  }, []);  // Stable reference
  
  return <ItemList items={items} onItemClick={handleItemClick} />;
}

// 3. React.memo for component-level memoization
const LessonCard = memo(function LessonCard({ 
  lesson, 
  onSelect 
}: LessonCardProps) {
  return (
    <div onClick={() => onSelect(lesson.id)}>
      <h3>{lesson.title}</h3>
      <ProgressBadge progress={lesson.progress} />
    </div>
  );
});
```

### When to Memoize

| Scenario | Use Memoization? | Why |
|----------|-----------------|-----|
| Parent re-renders often | ✅ Yes | Prevent child re-renders |
| Expensive computation | ✅ Yes | Cache result |
| Simple component | ❌ No | Overhead > benefit |
| Props rarely change | ❌ No | Already efficient |

---

## 📦 6. Bundle Analysis & Optimization

### Context

**Feature:** Minimize JavaScript payload  
**Complexity:** Medium  
**Technologies:** Next.js Bundle Analyzer

**The Challenge:**  
Identify and eliminate heavy dependencies that impact load time.

---

### Implementation

```typescript
// next.config.ts
import bundleAnalyzer from '@next/bundle-analyzer';

const withBundleAnalyzer = bundleAnalyzer({
  enabled: process.env.ANALYZE === 'true',
});

export default withBundleAnalyzer({
  // ... other config
});
```

### Analysis Process

```bash
# Run analysis
ANALYZE=true npm run build

# Outputs visual treemap of bundle composition
```

### Optimizations Applied

```typescript
// next.config.ts - Tree shaking for common libraries
export default {
  experimental: {
    optimizePackageImports: [
      '@react-three/drei',     // Only import used helpers
      '@react-three/fiber',
      'react-icons',           // Only import used icons
      'framer-motion',
      'lodash',                // Only import used functions
    ],
  },
};
```

### Import Pattern Changes

```typescript
// ❌ Before: Imports entire library
import { FaGithub, FaTwitter, FaLinkedin } from 'react-icons/fa';

// ✅ After: Direct imports (with optimizePackageImports)
// Same syntax, but bundler tree-shakes unused icons
import { FaGithub, FaTwitter, FaLinkedin } from 'react-icons/fa';
```

---

## 📊 Performance Results Summary

| Optimization | Before | After | Improvement |
|--------------|--------|-------|-------------|
| Total Blocking Time | 14,000ms | 2,500ms | **-82%** |
| Initial JS Bundle | ~800kb | ~400kb | **-50%** |
| Framer Motion | 50kb | 15kb | **-70%** |
| LCP | ~500ms | ~100ms | **-80%** |
| Mobile usability | Poor | Excellent | **Transformed** |

---

## 💡 Lessons Learned

### What Worked Well

- **Adaptive rendering** provides best experience per device
- **Web Workers** unlock true parallelism for heavy computation
- **LazyMotion** significant bundle savings with same features

### What I'd Do Differently

- Start with performance budget from day one
- Implement bundle analysis in CI pipeline
- Add real user monitoring earlier

### Key Takeaways

1. Profile before optimizing - measure, don't guess
2. Adaptive approaches beat one-size-fits-all
3. Bundle size directly impacts user experience

---

## 🔗 Related Patterns

- [Custom Hooks](./custom-hooks.md) - useDeviceTier and useGalaxyWorker
- [Complex Components](./complex-components.md) - Optimized component patterns
- [Design Patterns](../patterns/design-patterns.md) - Architectural patterns

---

## 📚 Further Reading

- [Web Vitals](https://web.dev/vitals/)
- [requestIdleCallback - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback)
- [Next.js Performance](https://nextjs.org/docs/pages/building-your-application/optimizing)
- [React Memo](https://react.dev/reference/react/memo)
- [Framer Motion LazyMotion](https://www.framer.com/motion/lazy-motion/)
