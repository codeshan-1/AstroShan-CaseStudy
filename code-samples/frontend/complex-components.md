<div align="center">

# 🧩 Complex Components

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Component+Architecture;CSS+3D+%26+Live+Validation" alt="Typing"/>

> **Component architecture patterns from AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🏠 1. CSS 3D Holographic House

### Context

**Feature:** Visual progress indicator that builds with student advancement  
**Complexity:** High  
**Technologies:** CSS 3D Transforms, Framer Motion, React

**The Challenge:**  
Create an impressive 3D visualization without adding WebGL overhead. The house should progressively reveal as students complete lessons.

---

### The Problem

Need a visually striking progress indicator that:
- Looks 3D and premium
- Doesn't require WebGL (already using for galaxy)
- Responds to progress percentage
- Works on all devices including mobile
- Supports different visual states (normal, pending, locked)

**Requirements:**
- True 3D perspective (not just 2D transforms)
- Progress-driven visibility
- Smooth animations
- Performance-optimized

**Constraints:**
- No additional WebGL contexts
- Must work in CSS-only environments
- Mobile performance matters

---

### The Approach

Use CSS `transform-style: preserve-3d` to create genuine 3D space, with Framer Motion for smooth progress-driven animations.

#### Why CSS 3D Instead of Three.js?

| Factor | CSS 3D | Three.js |
|--------|--------|----------|
| GPU Context | Browser compositor | Separate WebGL |
| Main thread | Minimal | Can be heavy |
| Device support | Universal | Requires WebGL |
| Complexity | Moderate | Higher |
| Animation | CSS + requestAnimationFrame | Custom loop |

---

### Implementation (Simplified)

```tsx
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - 512 lines of CSS
 * - 6 house components (base, 4 walls, roof)
 * - Orbiting particle rings
 * - Window details and glows
 * - Theme variants (cyan, yellow)
 */

import { motion, useTransform, MotionValue } from 'framer-motion';
import styles from './BlueprintHouse.module.css';

interface BlueprintHouseProps {
  progress: MotionValue<number>;  // 0 to 100
  isPending?: boolean;             // Yellow theme if true
}

export function BlueprintHouse({ progress, isPending = false }: BlueprintHouseProps) {
  const theme = isPending ? 'pending' : 'active';
  
  return (
    <div className={styles.container}>
      <div className={styles.scene}>
        {/* Foundation: Visible at 0-15% */}
        <HoloBase 
          progress={progress} 
          triggerRange={[0, 0.15]} 
          theme={theme}
        />
        
        {/* Walls: Visible at 15-55% */}
        <HoloFace 
          position="front" 
          progress={progress} 
          triggerRange={[0.15, 0.35]} 
          theme={theme}
        />
        <HoloFace 
          position="back" 
          progress={progress} 
          triggerRange={[0.25, 0.45]} 
          theme={theme}
        />
        <HoloFace 
          position="left" 
          progress={progress} 
          triggerRange={[0.35, 0.50]} 
          theme={theme}
        />
        <HoloFace 
          position="right" 
          progress={progress} 
          triggerRange={[0.40, 0.55]} 
          theme={theme}
        />
        
        {/* Roof: Visible at 55-80% */}
        <HoloRoof 
          progress={progress} 
          triggerRange={[0.55, 0.80]} 
          theme={theme}
        />
        
        {/* Details: Visible at 80-100% */}
        <OrbitingRing 
          progress={progress} 
          triggerRange={[0.80, 1.0]} 
          theme={theme}
        />
      </div>
    </div>
  );
}
```

### Progress-Driven Visibility Component

```tsx
interface HoloComponentProps {
  progress: MotionValue<number>;
  triggerRange: [number, number];
  theme: 'active' | 'pending';
  children?: React.ReactNode;
}

function HoloFace({ progress, triggerRange, theme, position }: HoloComponentProps & { position: string }) {
  // Transform progress to opacity based on trigger range
  const opacity = useTransform(
    progress,
    [triggerRange[0] * 100, triggerRange[1] * 100],
    [0, 1]
  );
  
  // Scale up as it appears
  const scale = useTransform(
    progress,
    [triggerRange[0] * 100, triggerRange[1] * 100],
    [0.8, 1]
  );

  const themeColor = theme === 'pending' ? 'var(--color-warning)' : 'var(--color-primary)';
  
  return (
    <motion.div
      className={`${styles.face} ${styles[position]}`}
      style={{
        opacity,
        scale,
        '--glow-color': themeColor,
      } as React.CSSProperties}
    />
  );
}
```

### CSS 3D Structure

```css
/* BlueprintHouse.module.css - Simplified */

.container {
  perspective: 1000px;
  perspective-origin: center center;
}

.scene {
  transform-style: preserve-3d;
  transform: rotateX(-15deg) rotateY(25deg);
  animation: gentleRotate 20s ease-in-out infinite;
}

@keyframes gentleRotate {
  0%, 100% { transform: rotateX(-15deg) rotateY(25deg); }
  50% { transform: rotateX(-15deg) rotateY(35deg); }
}

.face {
  position: absolute;
  transform-style: preserve-3d;
  backface-visibility: hidden;
  
  /* Holographic effect */
  background: linear-gradient(
    135deg,
    rgba(var(--glow-color-rgb), 0.1) 0%,
    rgba(var(--glow-color-rgb), 0.05) 100%
  );
  border: 1px solid rgba(var(--glow-color-rgb), 0.3);
  box-shadow: 
    0 0 20px rgba(var(--glow-color-rgb), 0.2),
    inset 0 0 20px rgba(var(--glow-color-rgb), 0.1);
}

.front {
  transform: translateZ(50px);
}

.back {
  transform: translateZ(-50px) rotateY(180deg);
}

.left {
  transform: rotateY(-90deg) translateZ(50px);
}

.right {
  transform: rotateY(90deg) translateZ(50px);
}

.roof {
  transform: rotateX(-45deg) translateY(-40px);
}
```

---

### Key Techniques Used

#### 1. preserve-3d for True 3D Space

```css
.scene {
  transform-style: preserve-3d;
}
```

*Why it matters:* Children maintain their 3D positions relative to parent, creating genuine depth.

#### 2. MotionValue for Reactive Transforms

```tsx
const opacity = useTransform(progress, [0, 50], [0, 1]);
```

*Why it matters:* Framer Motion's MotionValues update without re-renders.

#### 3. CSS Custom Properties for Theming

```tsx
style={{ '--glow-color': themeColor }}
```

*Why it matters:* Theme changes propagate through CSS cascade automatically.

---

### Performance Considerations

| Technique | Impact |
|-----------|--------|
| CSS transforms | GPU-accelerated, no layout recalculation |
| `will-change: transform` | Promotes to compositor layer |
| `backface-visibility: hidden` | Reduces render complexity |
| MotionValue | Updates bypass React re-renders |

---

## ✅ 2. TaskChecklist - Live Validation Component

### Context

**Feature:** Real-time code validation with visual feedback  
**Complexity:** High  
**Technologies:** React, DOMParser, Regex

**The Challenge:**  
Validate student HTML code in real-time, showing which requirements are met without blocking the UI.

---

### Implementation (Simplified)

```tsx
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - 20+ validator types
 * - Progressive hint system
 * - Accessibility announcements
 * - Animation on completion
 */

import { useMemo, useCallback } from 'react';

interface Requirement {
  id: string;
  label: string;
  type: 'elementExists' | 'hasContent' | 'hasAttribute';
  selector?: string;
  regex?: RegExp;
  attribute?: string;
  hint?: string;
}

interface TaskChecklistProps {
  code: string;
  requirements: Requirement[];
  onAllComplete: () => void;
}

export function TaskChecklist({ code, requirements, onAllComplete }: TaskChecklistProps) {
  // Parse HTML safely (no script execution)
  const parsedDOM = useMemo(() => {
    try {
      const parser = new DOMParser();
      return parser.parseFromString(code, 'text/html');
    } catch {
      return null;
    }
  }, [code]);

  // Validate each requirement
  const validationResults = useMemo(() => {
    if (!parsedDOM) return requirements.map(r => ({ ...r, passed: false }));
    
    return requirements.map(req => {
      let passed = false;
      
      switch (req.type) {
        case 'elementExists':
          passed = parsedDOM.querySelector(req.selector!) !== null;
          break;
          
        case 'hasContent':
          const element = parsedDOM.querySelector(req.selector!);
          passed = element?.textContent?.match(req.regex!) !== null;
          break;
          
        case 'hasAttribute':
          const el = parsedDOM.querySelector(req.selector!);
          passed = el?.hasAttribute(req.attribute!) ?? false;
          break;
      }
      
      return { ...req, passed };
    });
  }, [parsedDOM, requirements]);

  // Check if all passed
  const allPassed = validationResults.every(r => r.passed);
  
  // Notify parent when all complete
  useEffect(() => {
    if (allPassed) {
      onAllComplete();
    }
  }, [allPassed, onAllComplete]);

  return (
    <div className="task-checklist">
      <h3>Requirements</h3>
      <ul>
        {validationResults.map(req => (
          <li key={req.id} className={req.passed ? 'passed' : 'pending'}>
            <span className="icon">{req.passed ? '✅' : '⬜'}</span>
            <span className="label">{req.label}</span>
            {!req.passed && req.hint && (
              <span className="hint">{req.hint}</span>
            )}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Usage Example

```tsx
function LessonTaskView() {
  const [code, setCode] = useState(starterCode);
  const [canProceed, setCanProceed] = useState(false);
  
  const requirements: Requirement[] = [
    {
      id: 'has-header',
      label: 'Add a <header> element',
      type: 'elementExists',
      selector: 'header',
      hint: 'The header element wraps your page header content',
    },
    {
      id: 'has-nav',
      label: 'Add navigation inside header',
      type: 'elementExists',
      selector: 'header nav',
      hint: 'The nav element should be inside the header',
    },
    {
      id: 'has-links',
      label: 'Add at least 3 navigation links',
      type: 'hasContent',
      selector: 'nav',
      regex: /<a[^>]*>.*<\/a>.*<a[^>]*>.*<\/a>.*<a[^>]*>/s,
      hint: 'Use <a> tags for navigation links',
    },
  ];
  
  return (
    <div className="lesson-task">
      <CodeEditor value={code} onChange={setCode} />
      <TaskChecklist
        code={code}
        requirements={requirements}
        onAllComplete={() => setCanProceed(true)}
      />
      <button disabled={!canProceed}>
        Next Lesson →
      </button>
    </div>
  );
}
```

---

### Key Techniques Used

#### 1. DOMParser for Safe HTML Analysis

```typescript
const parser = new DOMParser();
const doc = parser.parseFromString(code, 'text/html');
```

*Why it matters:* DOMParser creates an inert document - no scripts execute.

#### 2. useMemo for Expensive Operations

```typescript
const validationResults = useMemo(() => { ... }, [parsedDOM, requirements]);
```

*Why it matters:* Parsing and validation only recalculate when inputs change.

#### 3. Declarative Requirement Definitions

```typescript
const requirements: Requirement[] = [
  { id: 'has-header', type: 'elementExists', selector: 'header' },
];
```

*Why it matters:* Requirements are data, not code. Easy to add/modify lessons.

---

## 🎭 3. Context-Aware Hero Component

### Context

**Feature:** Dynamic landing page hero based on user state  
**Complexity:** Medium  
**Technologies:** React, Framer Motion, SSR-safe patterns

**The Challenge:**  
Show different hero content based on authentication and progress state, while maintaining Server-Side Rendering for LCP.

---

### Implementation (Simplified)

```tsx
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - 6 distinct variants
 * - Complex progress calculations
 * - Animated transitions between variants
 * - Bilingual content (Arabic/English)
 */

import { useState, useEffect } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

type HeroVariant = 
  | 'guest'
  | 'guestWithProgress'
  | 'newExplorer'
  | 'inProgress'
  | 'bridgeTraveler'
  | 'courseMaster';

interface HeroContent {
  headline: string;
  subheadline: string;
  ctaText: string;
  ctaLink: string;
}

const variantContent: Record<HeroVariant, HeroContent> = {
  guest: {
    headline: 'Start Your Space Journey',
    subheadline: 'Learn web development through an immersive adventure',
    ctaText: 'Begin Exploration',
    ctaLink: '/path',
  },
  guestWithProgress: {
    headline: 'Continue Your Journey',
    subheadline: 'Your progress is saved locally',
    ctaText: 'Resume Learning',
    ctaLink: '/path',
  },
  newExplorer: {
    headline: 'Welcome, Captain!',
    subheadline: 'Your spaceship awaits. Ready for your first mission?',
    ctaText: 'Launch Mission',
    ctaLink: '/internet',
  },
  inProgress: {
    headline: 'Journey in Progress',
    subheadline: 'You\'re making great progress. Keep going!',
    ctaText: 'Continue',
    ctaLink: '/html',
  },
  bridgeTraveler: {
    headline: 'Bridge Crossed!',
    subheadline: 'HTML planet is now accessible',
    ctaText: 'Enter HTML',
    ctaLink: '/html',
  },
  courseMaster: {
    headline: 'Planet Conquered! 🎉',
    subheadline: 'You\'ve mastered this module',
    ctaText: 'View Certificate',
    ctaLink: '/profile',
  },
};

export function ContextAwareHero() {
  // Start with guest for SSR
  const [variant, setVariant] = useState<HeroVariant>('guest');
  const [isHydrated, setIsHydrated] = useState(false);

  useEffect(() => {
    setIsHydrated(true);
    
    // Determine variant based on state
    const determineVariant = async () => {
      const session = getAuthSession(); // from auth store
      const progress = getProgressData(); // from progress store
      
      if (!session.isAuthenticated) {
        if (progress.totalCompleted > 0) {
          return 'guestWithProgress';
        }
        return 'guest';
      }
      
      if (progress.totalCompleted === 0) {
        return 'newExplorer';
      }
      
      if (progress.currentModule === 'bridge') {
        return 'bridgeTraveler';
      }
      
      if (progress.moduleComplete) {
        return 'courseMaster';
      }
      
      return 'inProgress';
    };
    
    determineVariant().then(setVariant);
  }, []);

  const content = variantContent[variant];

  return (
    <section className="hero">
      {/* Server-rendered headline for LCP */}
      <h1 id="hero-headline" suppressHydrationWarning>
        {content.headline}
      </h1>
      
      {/* Client-only dynamic content */}
      <AnimatePresence mode="wait">
        {isHydrated && (
          <motion.div
            key={variant}
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -20 }}
            transition={{ duration: 0.3 }}
          >
            <p>{content.subheadline}</p>
            <a href={content.ctaLink} className="cta-button">
              {content.ctaText}
            </a>
          </motion.div>
        )}
      </AnimatePresence>
    </section>
  );
}
```

---

### Key Techniques Used

#### 1. SSR-Safe Initial State

```tsx
const [variant, setVariant] = useState<HeroVariant>('guest');
```

*Why it matters:* Server and client render same initial content.

#### 2. suppressHydrationWarning

```tsx
<h1 suppressHydrationWarning>{content.headline}</h1>
```

*Why it matters:* Prevents React warnings when content updates after hydration.

#### 3. AnimatePresence for Transitions

```tsx
<AnimatePresence mode="wait">
  <motion.div key={variant}>...</motion.div>
</AnimatePresence>
```

*Why it matters:* Smooth transitions when variant changes.

---

## 💡 Lessons Learned

### What Worked Well

- CSS 3D provides impressive visuals without WebGL overhead
- DOMParser enables safe real-time HTML validation
- SSR-safe patterns prevent hydration issues

### What I'd Do Differently

- Pre-compute more variant logic on the server
- Use CSS containment for better isolation
- Add more granular validation error messages

### Key Takeaways

1. CSS 3D is underutilized for impressive, performant visuals
2. DOMParser is perfect for safe HTML analysis
3. Always design components with SSR in mind

---

## 🔗 Related Patterns

- [Custom Hooks](./custom-hooks.md) - Hooks used by these components
- [State Management](./state-management.md) - How state flows to components
- [Optimization Techniques](./optimization-techniques.md) - Performance patterns

---

## 📚 Further Reading

- [CSS 3D Transforms - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-style)
- [DOMParser API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/DOMParser)
- [Framer Motion AnimatePresence](https://www.framer.com/motion/animate-presence/)
- [React Hydration](https://react.dev/reference/react-dom/client/hydrateRoot)
