<div align="center">

# 🔧 Custom Hooks

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Reusable+Logic+Patterns;Web+Workers+%26+Device+Detection" alt="Typing"/>

> **Reusable logic patterns extracted from AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🌌 1. useDeviceTier - Adaptive Hardware Detection

### Context

**Feature:** Adaptive 3D rendering based on device capabilities  
**Complexity:** Medium  
**Technologies:** React, Navigator API, SSR-safe patterns

**The Challenge:**  
Detect device hardware capabilities to automatically select the appropriate rendering tier (WebGL full, WebGL reduced, or Canvas 2D fallback) without causing hydration mismatches.

---

### The Problem

Need to adapt visual experience based on device power:
- High-end: Full WebGL with 100k particles
- Mid-range: Reduced particle count
- Low-end: Canvas 2D fallback

**Requirements:**
- Detect hardware capabilities accurately
- Work with SSR (no `navigator` on server)
- Avoid React hydration mismatches
- Update tier after initial render without layout shift

**Constraints:**
- `navigator.hardwareConcurrency` not available on server
- Initial render must match server for hydration
- State update shouldn't cause visible flicker

---

### The Approach

Initialize with the safest tier ("low") for SSR consistency, then upgrade after hydration based on actual hardware.

#### Why This Pattern?

1. **SSR Safety:** Always starts with "low" - matches server
2. **No Hydration Mismatch:** State upgrades after mount
3. **Progressive Enhancement:** Low-end devices get immediate content, high-end get enhanced after

#### Alternatives Considered

| Approach | Pros | Cons |
|----------|------|------|
| Server-side detection via UA | Works on first render | Unreliable, doesn't reflect actual capability |
| Client-only rendering | No hydration issues | Blocks SSR benefits |
| **Post-hydration upgrade** ✅ | SSR + accurate detection | Brief delay before upgrade |

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration purposes.
 * Actual implementation includes:
 * - GPU vendor string analysis
 * - WebGL capability probing
 * - Memory estimation
 * - Battery status consideration
 */

import { useState, useEffect } from 'react';

type DeviceTier = 'low' | 'medium' | 'high';

function getHardwareTier(): DeviceTier {
  if (typeof navigator === 'undefined') return 'low';
  
  const cores = navigator.hardwareConcurrency || 4;
  
  // Simple heuristic based on CPU cores
  if (cores >= 6) return 'high';    // 100k particles, full WebGL
  if (cores >= 2) return 'medium';  // 25k particles
  return 'low';                     // Canvas 2D fallback
}

export function useDeviceTier(): DeviceTier {
  // Initialize with "low" for SSR consistency
  const [tier, setTier] = useState<DeviceTier>('low');

  useEffect(() => {
    // Defer tier detection to avoid blocking first paint
    const timer = setTimeout(() => {
      const actualTier = getHardwareTier();
      if (actualTier !== 'low') {
        setTier(actualTier);
      }
    }, 0);
    
    return () => clearTimeout(timer);
  }, []);

  return tier;
}
```

### Usage Example

```typescript
function GalaxyBackground() {
  const tier = useDeviceTier();
  
  // Render appropriate component based on tier
  if (tier === 'high') {
    return <InteractiveGalaxy particleCount={100000} />;
  }
  
  if (tier === 'medium') {
    return <InteractiveGalaxy particleCount={25000} />;
  }
  
  return <StaticBackground />; // Canvas 2D fallback
}
```

---

### Key Techniques Used

#### 1. SSR-Safe Initial State

```typescript
const [tier, setTier] = useState<DeviceTier>('low');
```

*Why it matters:* Server renders with "low", client hydrates with same value - no mismatch.

#### 2. Deferred Detection

```typescript
const timer = setTimeout(() => { ... }, 0);
```

*Why it matters:* Pushes tier detection to next tick, ensuring hydration completes first.

#### 3. Guard Clause for SSR

```typescript
if (typeof navigator === 'undefined') return 'low';
```

*Why it matters:* Prevents crashes during server-side rendering.

---

### Performance Considerations

| Aspect | Impact |
|--------|--------|
| Initial render | Immediate (uses default "low") |
| Tier upgrade | Next tick (~0ms delay) |
| Re-renders | Only on tier change |
| Memory | Negligible (single state value) |

---

## 🔮 2. useGalaxyWorker - Web Worker Particle Generation

### Context

**Feature:** 100,000 particle galaxy background  
**Complexity:** High  
**Technologies:** Web Workers, React, Three.js, Typed Arrays

**The Challenge:**  
Generate 100,000 particles with position and color calculations without blocking the main thread. Initial implementation caused 14-second Total Blocking Time.

---

### The Problem

Galaxy rendering requires:
- 100,000 position calculations (x, y, z)
- 100,000 color interpolations
- Trigonometric operations for spiral arms
- Real-time update capability

**Requirements:**
- Zero main thread blocking
- Smooth 60fps animation
- Memory-efficient data transfer
- Single-shot generation (not continuous)

**Constraints:**
- Web Workers can't access DOM
- Data transfer can be expensive
- Worker creation has overhead

---

### The Approach

Offload all calculations to a dedicated Web Worker with zero-copy array transfer.

#### Why This Pattern?

1. **True Parallelism:** Worker runs on separate thread
2. **Zero-Copy Transfer:** `Transferable` arrays move, don't copy
3. **Single-Shot:** Worker terminates after generation, freeing resources

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration purposes.
 * Actual implementation includes:
 * - Spiral arm mathematics
 * - Randomness distribution curves
 * - Color temperature gradients
 * - Multiple galaxy types
 */

import { useState, useEffect, useRef } from 'react';

interface GalaxyParams {
  count: number;
  radius: number;
  branches: number;
  spin: number;
  insideColor: string;
  outsideColor: string;
}

interface GalaxyResult {
  positions: Float32Array;
  colors: Float32Array;
}

export function useGalaxyWorker(params: GalaxyParams) {
  const [results, setResults] = useState<GalaxyResult | null>(null);
  const [loading, setLoading] = useState(true);
  const workerRef = useRef<Worker | null>(null);

  useEffect(() => {
    // Create worker from same-origin URL
    const worker = new Worker(
      new URL('./galaxyWorker.ts', import.meta.url)
    );
    workerRef.current = worker;

    worker.onmessage = (event: MessageEvent<GalaxyResult>) => {
      setResults(event.data);
      setLoading(false);
      
      // Single-shot: terminate after generation
      worker.terminate();
    };

    // Send parameters to worker
    worker.postMessage({
      count: params.count,
      radius: params.radius,
      branches: params.branches,
      spin: params.spin,
      insideColor: params.insideColor,
      outsideColor: params.outsideColor,
    });

    return () => {
      worker.terminate();
    };
  }, [params.count, params.radius, params.branches, params.spin]);

  return { results, loading };
}
```

### Worker Implementation (Simplified)

```typescript
// galaxyWorker.ts
/**
 * DISCLAIMER: Simplified spiral mathematics.
 * Actual implementation uses more complex distribution curves.
 */

self.onmessage = (event) => {
  const { count, radius, branches, spin } = event.data;
  
  // Create typed arrays for efficiency
  const positions = new Float32Array(count * 3);
  const colors = new Float32Array(count * 3);
  
  for (let i = 0; i < count; i++) {
    const i3 = i * 3;
    
    // Simplified spiral arm calculation
    const radiusValue = Math.random() * radius;
    const branchAngle = ((i % branches) / branches) * Math.PI * 2;
    const spinAngle = radiusValue * spin;
    
    positions[i3] = Math.cos(branchAngle + spinAngle) * radiusValue;
    positions[i3 + 1] = (Math.random() - 0.5) * 0.5;
    positions[i3 + 2] = Math.sin(branchAngle + spinAngle) * radiusValue;
    
    // Simplified color interpolation
    const colorMix = radiusValue / radius;
    colors[i3] = 1 - colorMix;     // R
    colors[i3 + 1] = 0.5;          // G
    colors[i3 + 2] = colorMix;     // B
  }
  
  // Transfer ownership (zero-copy)
  self.postMessage(
    { positions, colors },
    [positions.buffer, colors.buffer]
  );
};
```

### Usage with Three.js

```typescript
function Galaxy() {
  const { results, loading } = useGalaxyWorker({
    count: 100000,
    radius: 10,
    branches: 5,
    spin: 1,
    insideColor: '#ff6030',
    outsideColor: '#1eb8e4',
  });

  if (loading || !results) {
    return <FallbackLoader />;
  }

  return (
    <points>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          count={results.positions.length / 3}
          array={results.positions}
          itemSize={3}
        />
        <bufferAttribute
          attach="attributes-color"
          count={results.colors.length / 3}
          array={results.colors}
          itemSize={3}
        />
      </bufferGeometry>
      <pointsMaterial size={0.01} vertexColors />
    </points>
  );
}
```

---

### Key Techniques Used

#### 1. Transferable Arrays

```typescript
self.postMessage(
  { positions, colors },
  [positions.buffer, colors.buffer]  // Transfer list
);
```

*Why it matters:* Arrays are **moved**, not copied. Zero memory overhead for transfer.

#### 2. Worker URL Pattern

```typescript
new Worker(new URL('./galaxyWorker.ts', import.meta.url))
```

*Why it matters:* Works with bundlers (Webpack, Vite) that need to resolve worker files.

#### 3. Single-Shot Termination

```typescript
worker.terminate();
```

*Why it matters:* Frees worker thread resources after one-time generation.

---

### Performance Considerations

| Metric | Before (Main Thread) | After (Web Worker) |
|--------|---------------------|-------------------|
| Total Blocking Time | 14,000ms | 0ms |
| Time to Interactive | 15s+ | ~2.5s |
| Main thread free | No | Yes |
| User can interact during load | No | Yes |

---

## 🚪 3. useGatekeeper - Access Control Hook

### Context

**Feature:** Course progression enforcement  
**Complexity:** Medium  
**Technologies:** React, localStorage, Zustand

**The Challenge:**  
Enforce learning path order (Internet → Bridge Exam → HTML) while providing visual feedback for locked content.

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - Multiple gate types
 * - Admin override capabilities
 * - Time-based unlocking
 * - Partial completion states
 */

import { useState, useEffect, useCallback } from 'react';

interface GatekeeperState {
  isAuthorized: boolean;
  isInternetComplete: boolean;
  isPending: boolean;
}

export function useGatekeeper(): GatekeeperState & {
  authorize: () => void;
} {
  const [state, setState] = useState<GatekeeperState>({
    isAuthorized: false,
    isInternetComplete: false,
    isPending: true,
  });

  useEffect(() => {
    // Check stored authorization
    const checkAuthorization = async () => {
      const bridgeAuth = localStorage.getItem('bridge_authorized');
      const internetProgress = localStorage.getItem('internet_progress');
      
      const internetComplete = internetProgress
        ? JSON.parse(internetProgress).completedLessons?.length >= 10
        : false;
      
      setState({
        isAuthorized: bridgeAuth === 'true' || internetComplete,
        isInternetComplete: internetComplete,
        isPending: false,
      });
    };
    
    checkAuthorization();
  }, []);

  const authorize = useCallback(() => {
    localStorage.setItem('bridge_authorized', 'true');
    setState(prev => ({ ...prev, isAuthorized: true }));
  }, []);

  return { ...state, authorize };
}
```

### Usage with Route Guard

```typescript
function RequireBridgeAuthorized({ children }: { children: React.ReactNode }) {
  const { isAuthorized, isPending } = useGatekeeper();
  const router = useRouter();

  if (isPending) {
    return <LoadingSpinner />;
  }

  if (!isAuthorized) {
    return <LockedContentMessage onTakeExam={() => router.push('/bridge')} />;
  }

  return <>{children}</>;
}
```

---

## 📚 4. useHtmlLessons - Lesson State Management

### Context

**Feature:** 57 lesson curriculum with progress tracking  
**Complexity:** High  
**Technologies:** React, localStorage, MongoDB sync

**The Challenge:**  
Manage lesson state across navigation, handle quiz scoring, and sync progress between local and cloud storage.

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - 57 lesson configurations
 * - Quiz scoring algorithms
 * - Task code persistence
 * - Widget interaction tracking
 */

import { useState, useEffect, useCallback } from 'react';

interface Lesson {
  id: number;
  title: string;
  completed: boolean;
  quizPassed: boolean;
}

interface UseLessonsReturn {
  lessons: Lesson[];
  currentLesson: Lesson | null;
  markComplete: (id: number) => void;
  markQuizPassed: (id: number, score: number) => void;
  goToLesson: (id: number) => void;
  progress: number;
}

export function useLessons(manifest: Lesson[]): UseLessonsReturn {
  const [lessons, setLessons] = useState<Lesson[]>(manifest);
  const [currentLessonId, setCurrentLessonId] = useState<number | null>(null);

  // Load saved progress on mount
  useEffect(() => {
    const saved = localStorage.getItem('lesson_progress');
    if (saved) {
      const completedIds = JSON.parse(saved).completedLessons || [];
      setLessons(prev =>
        prev.map(lesson => ({
          ...lesson,
          completed: completedIds.includes(lesson.id),
        }))
      );
    }
  }, []);

  const markComplete = useCallback((id: number) => {
    setLessons(prev => {
      const updated = prev.map(lesson =>
        lesson.id === id ? { ...lesson, completed: true } : lesson
      );
      
      // Persist to localStorage
      const completedIds = updated
        .filter(l => l.completed)
        .map(l => l.id);
      localStorage.setItem(
        'lesson_progress',
        JSON.stringify({ completedLessons: completedIds })
      );
      
      return updated;
    });
  }, []);

  const progress = (lessons.filter(l => l.completed).length / lessons.length) * 100;

  return {
    lessons,
    currentLesson: lessons.find(l => l.id === currentLessonId) || null,
    markComplete,
    markQuizPassed: (id, score) => { /* ... */ },
    goToLesson: setCurrentLessonId,
    progress,
  };
}
```

---

## 🧪 Testing Approach

```typescript
describe('useDeviceTier', () => {
  it('should return "low" on server (no navigator)', () => {
    const originalNavigator = global.navigator;
    // @ts-ignore
    delete global.navigator;
    
    const { result } = renderHook(() => useDeviceTier());
    expect(result.current).toBe('low');
    
    global.navigator = originalNavigator;
  });

  it('should upgrade tier after hydration', async () => {
    Object.defineProperty(navigator, 'hardwareConcurrency', {
      value: 8,
      configurable: true,
    });
    
    const { result } = renderHook(() => useDeviceTier());
    
    // Initially low
    expect(result.current).toBe('low');
    
    // Wait for useEffect
    await act(async () => {
      await new Promise(resolve => setTimeout(resolve, 10));
    });
    
    // Now upgraded
    expect(result.current).toBe('high');
  });
});
```

---

## 💡 Lessons Learned

### What Worked Well

- **SSR-first thinking:** Starting with the lowest tier prevents hydration issues
- **Web Workers for heavy computation:** Dramatic improvement in user experience
- **Single-responsibility hooks:** Each hook does one thing well

### What I'd Do Differently

- Add more granular GPU detection (WebGL renderer string analysis)
- Implement worker pooling for multiple heavy operations
- Add telemetry to track actual device tier distribution

### Key Takeaways

1. Always consider SSR when using browser APIs
2. Web Workers are underutilized for compute-heavy tasks
3. Custom hooks enable excellent code reuse and testing

---

## 🔗 Related Patterns

- [State Management](./state-management.md) - How hooks integrate with stores
- [Optimization Techniques](./optimization-techniques.md) - Lazy loading patterns
- [Design Patterns](../patterns/design-patterns.md) - Underlying architectural patterns

---

## 📚 Further Reading

- [Web Workers API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)
- [React Hooks Documentation](https://react.dev/reference/react)
- [Transferable Objects](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects)
- [SSR Hydration in React](https://react.dev/reference/react-dom/client/hydrateRoot)
