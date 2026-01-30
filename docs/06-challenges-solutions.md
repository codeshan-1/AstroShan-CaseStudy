<div align="center">

# 💡 06 - Challenges & Solutions

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Engineering+Problem+Solving;Real-World+Challenges;Innovative+Solutions" alt="Typing"/>

> **Engineering stories of problems encountered and creative solutions implemented**

<br/>

[![Prev](https://img.shields.io/badge/←_Decisions-1eb8e4?style=for-the-badge)](05-technical-decisions.md)
[![Next](https://img.shields.io/badge/Performance_→-7565e3?style=for-the-badge)](07-performance.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🔥 Challenge 1: The 14-Second Freeze

### 📋 Problem
Galaxy animation caused 14-second Total Blocking Time (TBT) on mobile.

### 🔍 Investigation
Chrome DevTools Performance tab revealed:
- 72% of TBT from single function: `generateParticles()`
- 100k iterations × position + color calculations
- All on main thread, blocking user interaction

### ✅ Solution
**Web Worker Architecture**

```
Main Thread              Web Worker
     │                       │
     │  postMessage(params)  │
     ├──────────────────────►│
     │                       │ [Heavy calculation]
     │  onmessage(data)      │
     │◄──────────────────────┤
     ▼                       ▼
```

### 📊 Results

| Metric | Before | After | Improvement |
|:------:|:------:|:-----:|:-----------:|
| TBT | 14,000ms | 2,500ms | **-82%** |
| User perception | Frozen | Responsive | ✅ |

---

## 💻 Challenge 2: Monaco on Low-End Devices

### 📋 Problem
Monaco Editor consumed too much memory on budget smartphones (2GB RAM).

### 🔍 Investigation
- Monaco loads entire editor engine on first render
- TypeScript integration adds significant overhead
- No built-in lazy loading mechanism

### ✅ Solution

1. **Lazy load Monaco only on lesson pages**
```typescript
const Monaco = dynamic(() => import("@monaco-editor/react"), {
  ssr: false,
  loading: () => <EditorSkeleton />
});
```

2. **Disable unnecessary features**
```typescript
options={{
  minimap: { enabled: false },
  lineNumbers: screenWidth > 768 ? "on" : "off",
  suggest: { showIcons: false }
}}
```

3. **Memory cleanup on unmount**

### 📊 Results
- ✅ 40% reduced memory footprint
- ✅ Faster initial page load
- ✅ Works on budget devices

---

## 🔄 Challenge 3: RTL/LTR Bidirectional Content

### 📋 Problem
Arabic UI (RTL) with English code examples (LTR) caused rendering issues.

### 🔍 Symptoms
- Code snippets displayed backwards
- Cursor jumping in editor
- Variable names corrupted

### ✅ Solution

**Multi-layer approach:**

1. **CSS isolation**
```css
.code-block {
  direction: ltr !important;
  unicode-bidi: embed;
}
```

2. **Editor configuration**
```typescript
<Editor dir="ltr" options={{ rtlAlign: false }} />
```

3. **Component boundaries**
```typescript
<div dir="rtl"> {/* Arabic content */}
  <div dir="ltr"> {/* Code section */}
    <MonacoEditor />
  </div>
</div>
```

### 🎯 Key Learning
> Bidirectional content requires explicit direction at each boundary.

---

## 🔐 Challenge 4: Progress Sync Race Conditions

### 📋 Problem
Simultaneous localStorage and MongoDB updates caused data loss.

### 🔍 Scenario
```
Tab 1: Save lesson 3 → localStorage
Tab 2: Save lesson 5 → localStorage (overwrites)
Cloud: Receives lesson 5 only
```

### ✅ Solution

**Event-based synchronization:**

```typescript
// On any save, dispatch event
localStorage.setItem(key, data);
window.dispatchEvent(new Event("local-storage-update"));

// Other tabs listen
window.addEventListener("storage", handleCrossTabSync);
```

**MongoDB atomic operations:**
```typescript
await collection.updateOne(
  { userId },
  { $addToSet: { completedLessons: lessonId } } // Never loses data
);
```

### 📊 Results
- ✅ Zero data loss across tabs
- ✅ Eventual consistency with cloud
- ✅ Atomic MongoDB operations

---

## 🎭 Challenge 5: Hydration Mismatch

### 📋 Problem
Server-rendered content didn't match client state, causing React errors.

### 🔍 Cause
Auth state only available on client:
```
Server: No session → renders "Guest" hero
Client: Session exists → expects "Welcome, [Name]"
Mismatch!
```

### ✅ Solution

**Two-phase rendering:**
```typescript
function HeroSection() {
  const [variant, setVariant] = useState<HeroVariant>("guest");
  const [hydrated, setHydrated] = useState(false);

  useEffect(() => {
    setHydrated(true);
    // Determine actual variant based on auth
    determineVariant().then(setVariant);
  }, []);

  // Server and initial client render: consistent "guest"
  // After hydration: update to actual variant with transition
}
```

### 🎯 Key Learning
> Always render server-safe default, then update on client.

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 Related Documents

| Navigation |
|:----------:|
| [🔧 ← Technical Decisions](05-technical-decisions.md) |
| [⚡ Performance →](07-performance.md) |

---

*Next: [07 - Performance Optimization](07-performance.md)*
