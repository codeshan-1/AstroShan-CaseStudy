<div align="center">

# 📦 State Management

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=State+Handling+Patterns;Zustand+%26+3-Way+Merge" alt="Typing"/>

> **State handling approaches used in AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🔐 1. AuthStore - Singleton Authentication State

### Context

**Feature:** Anonymous-first authentication with cloud sync  
**Complexity:** High  
**Technologies:** TypeScript, localStorage, JWT, Singleton Pattern

**The Challenge:**  
Manage authentication state that:
- Works without registration (anonymous users)
- Persists sessions securely
- Syncs with cloud when users authenticate
- Handles token refresh silently

---

### The Problem

Need a centralized auth state that:
- Provides instant access (no registration required)
- Stores tokens securely (XSS/CSRF prevention)
- Handles session persistence across reloads
- Merges anonymous → authenticated transitions

**Requirements:**
- UUID for every visitor (anonymous identity)
- Google OAuth integration
- Automatic token refresh
- Cross-tab sync

**Constraints:**
- Access tokens can't be in localStorage (XSS risk)
- Refresh tokens need HttpOnly cookies (CSRF prevention)
- Must work with SSR

---

### The Approach

Singleton store with in-memory access tokens and localStorage session metadata.

#### Why Singleton?

| Need | Singleton Solution |
|------|-------------------|
| Single source of truth | One instance across app |
| Token refresh coordination | No race conditions |
| Cross-component access | Direct import, no props |
| SSR compatibility | Lazy initialization |

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - Full OAuth flow handling
 * - Token validation and refresh
 * - Profile management
 * - Error recovery
 */

interface Session {
  user: {
    id: string;
    displayName?: string;
    email?: string;
    avatarUrl?: string;
  };
  isAuthenticated: boolean;
}

interface AuthStore {
  getSession(): Promise<Session>;
  loginAnonymously(): Promise<void>;
  loginWithGoogle(): Promise<void>;
  logout(): Promise<void>;
  getAccessToken(): Promise<string | null>;
}

const STORAGE_KEY = 'auth_session';

class LocalStorageAuthStore implements AuthStore {
  private session: Session | null = null;
  private accessToken: string | null = null;  // IN-MEMORY ONLY
  private refreshPromise: Promise<string | null> | null = null;
  private subscribers: Set<() => void> = new Set();

  async init(): Promise<void> {
    if (typeof window === 'undefined') return;

    try {
      const stored = localStorage.getItem(STORAGE_KEY);
      if (stored) {
        this.session = JSON.parse(stored);
        
        // Verify stored authenticated session
        if (this.session?.isAuthenticated) {
          const token = await this.refreshAccessToken();
          if (!token) {
            // Session invalid, revert to anonymous
            await this.loginAnonymously();
          }
        }
      } else {
        // First visit - create anonymous session
        await this.loginAnonymously();
      }
    } catch (e) {
      console.error('Auth init failed:', e);
      await this.loginAnonymously();
    }

    this.notifySubscribers();
  }

  async loginAnonymously(): Promise<void> {
    const anonymousId = crypto.randomUUID();
    
    this.session = {
      user: { id: anonymousId },
      isAuthenticated: false,
    };
    this.accessToken = null;
    
    this.persistSession();
  }

  async loginWithGoogle(): Promise<void> {
    // Redirect to OAuth endpoint
    window.location.href = '/api/auth/google/signin';
  }

  async handleOAuthCallback(code: string): Promise<void> {
    const response = await fetch('/api/auth/google/callback', {
      method: 'POST',
      body: JSON.stringify({ code }),
      credentials: 'include',  // Include cookies
    });
    
    const data = await response.json();
    
    // Store access token IN MEMORY ONLY
    this.accessToken = data.accessToken;
    
    // Store session metadata (no tokens) in localStorage
    this.session = {
      user: data.user,
      isAuthenticated: true,
    };
    
    this.persistSession();
    this.notifySubscribers();
  }

  async getAccessToken(): Promise<string | null> {
    // Return if valid
    if (this.accessToken && !this.isTokenExpired(this.accessToken)) {
      return this.accessToken;
    }
    
    // Refresh if we have a session
    if (this.session?.isAuthenticated) {
      return this.refreshAccessToken();
    }
    
    return null;
  }

  async refreshAccessToken(): Promise<string | null> {
    // Dedupe concurrent refresh calls
    if (this.refreshPromise) {
      return this.refreshPromise;
    }
    
    this.refreshPromise = this.doRefresh();
    const result = await this.refreshPromise;
    this.refreshPromise = null;
    
    return result;
  }

  private async doRefresh(): Promise<string | null> {
    try {
      const response = await fetch('/api/auth/token/refresh', {
        method: 'POST',
        credentials: 'include',  // Send HttpOnly cookie
      });
      
      if (!response.ok) {
        throw new Error('Refresh failed');
      }
      
      const { accessToken } = await response.json();
      this.accessToken = accessToken;  // Store in memory
      
      return accessToken;
    } catch (e) {
      this.accessToken = null;
      return null;
    }
  }

  async logout(): Promise<void> {
    // Revoke refresh token on server
    await fetch('/api/auth/token/revoke', {
      method: 'POST',
      credentials: 'include',
    });
    
    // Clear in-memory token
    this.accessToken = null;
    
    // Create new anonymous session
    await this.loginAnonymously();
    this.notifySubscribers();
  }

  async getSession(): Promise<Session> {
    if (!this.session) {
      await this.init();
    }
    return this.session!;
  }

  subscribe(callback: () => void): () => void {
    this.subscribers.add(callback);
    return () => this.subscribers.delete(callback);
  }

  private persistSession(): void {
    // NOTE: Only session metadata, NEVER tokens
    localStorage.setItem(STORAGE_KEY, JSON.stringify(this.session));
  }

  private notifySubscribers(): void {
    this.subscribers.forEach(cb => cb());
  }

  private isTokenExpired(token: string): boolean {
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      return Date.now() >= payload.exp * 1000;
    } catch {
      return true;
    }
  }
}

// Singleton export
export const authStore = new LocalStorageAuthStore();
```

### React Hook Wrapper

```typescript
import { useSyncExternalStore } from 'react';

export function useAuth() {
  const session = useSyncExternalStore(
    authStore.subscribe.bind(authStore),
    () => authStore.getSession(),
    () => ({ user: { id: '' }, isAuthenticated: false })  // SSR fallback
  );

  return {
    session,
    login: authStore.loginWithGoogle.bind(authStore),
    logout: authStore.logout.bind(authStore),
    isAuthenticated: session?.isAuthenticated ?? false,
  };
}
```

---

### Key Techniques Used

#### 1. In-Memory Token Storage

```typescript
private accessToken: string | null = null;  // Never in localStorage
```

*Why it matters:* XSS attacks can't access in-memory variables like they can localStorage.

#### 2. Promise Deduplication

```typescript
if (this.refreshPromise) {
  return this.refreshPromise;
}
```

*Why it matters:* Prevents multiple concurrent refresh requests that would fail.

#### 3. useSyncExternalStore for React Integration

```typescript
useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
```

*Why it matters:* React 18+ way to subscribe to external stores with SSR support.

---

## 📊 2. ProgressStore - 3-Way Merge Sync

### Context

**Feature:** Progress persistence with local-cloud sync  
**Complexity:** High  
**Technologies:** localStorage, MongoDB, Event-driven updates

**The Challenge:**  
Sync user progress between localStorage and MongoDB without data loss, handling the anonymous → authenticated transition.

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - HTML and Internet module progress
 * - Quiz scores and attempts
 * - Code task persistence
 * - Widget interaction tracking
 */

interface ModuleProgress {
  completedLessons: number[];
  quizScores: Record<number, number>;
  lastUpdated: number;
}

interface ProgressStore {
  getProgress(module: string): Promise<ModuleProgress>;
  saveProgress(module: string, progress: ModuleProgress): Promise<void>;
  syncWithCloud(userId: string): Promise<void>;
}

const STORAGE_PREFIX = 'progress_';

class LocalProgressStore implements ProgressStore {
  async getProgress(module: string): Promise<ModuleProgress> {
    const stored = localStorage.getItem(`${STORAGE_PREFIX}${module}`);
    if (stored) {
      return JSON.parse(stored);
    }
    return { completedLessons: [], quizScores: {}, lastUpdated: 0 };
  }

  async saveProgress(
    module: string, 
    progress: ModuleProgress,
    skipSync = false  // Prevent sync loops
  ): Promise<void> {
    // Add timestamp
    const withTimestamp = {
      ...progress,
      lastUpdated: Date.now(),
    };
    
    // Save to localStorage
    localStorage.setItem(
      `${STORAGE_PREFIX}${module}`,
      JSON.stringify(withTimestamp)
    );
    
    // Notify other components
    window.dispatchEvent(new Event('progress-update'));
    
    // Sync to cloud if authenticated and not in a sync loop
    if (!skipSync) {
      const session = await authStore.getSession();
      if (session.isAuthenticated) {
        await this.syncToCloud(session.user.id, module, withTimestamp);
      }
    }
  }

  async syncWithCloud(userId: string): Promise<void> {
    // Fetch cloud progress
    const cloudProgress = await this.fetchCloudProgress(userId);
    
    // Get local progress
    const localProgress = await this.getProgress('html');
    
    // 3-way merge
    const merged = this.mergeProgress(localProgress, cloudProgress);
    
    // Save merged locally (skipSync to prevent loop)
    await this.saveProgress('html', merged, true);
    
    // Save merged to cloud
    await this.saveToCloud(userId, 'html', merged);
  }

  private mergeProgress(
    local: ModuleProgress, 
    cloud: ModuleProgress
  ): ModuleProgress {
    return {
      // Union of completed lessons
      completedLessons: [...new Set([
        ...local.completedLessons,
        ...cloud.completedLessons,
      ])],
      
      // Max of each quiz score
      quizScores: this.mergeQuizScores(
        local.quizScores, 
        cloud.quizScores
      ),
      
      // Keep latest timestamp
      lastUpdated: Math.max(local.lastUpdated, cloud.lastUpdated),
    };
  }

  private mergeQuizScores(
    local: Record<number, number>,
    cloud: Record<number, number>
  ): Record<number, number> {
    const allKeys = new Set([
      ...Object.keys(local).map(Number),
      ...Object.keys(cloud).map(Number),
    ]);
    
    const merged: Record<number, number> = {};
    for (const key of allKeys) {
      merged[key] = Math.max(local[key] ?? 0, cloud[key] ?? 0);
    }
    
    return merged;
  }

  private async fetchCloudProgress(userId: string): Promise<ModuleProgress> {
    const token = await authStore.getAccessToken();
    const response = await fetch(`/api/progress?userId=${userId}`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    return response.json();
  }

  private async saveToCloud(
    userId: string, 
    module: string, 
    progress: ModuleProgress
  ): Promise<void> {
    const token = await authStore.getAccessToken();
    await fetch('/api/progress', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({ userId, module, progress }),
    });
  }
}

export const progressStore = new LocalProgressStore();
```

---

### Key Techniques Used

#### 1. 3-Way Merge for Union

```typescript
completedLessons: [...new Set([...local, ...cloud])]
```

*Why it matters:* No lessons are ever "un-completed" during merge.

#### 2. skipSync Parameter

```typescript
await this.saveProgress('html', merged, true);  // skipSync = true
```

*Why it matters:* Prevents infinite sync loops when updating after cloud fetch.

#### 3. Event-Driven Updates

```typescript
window.dispatchEvent(new Event('progress-update'));
```

*Why it matters:* Components can subscribe to progress changes without prop drilling.

---

## 🌀 3. ScrollStore - UI State with Zustand

### Context

**Feature:** Scroll-driven animations and visibility  
**Complexity:** Low  
**Technologies:** Zustand

**The Challenge:**  
Efficiently share scroll progress across multiple components without causing unnecessary re-renders.

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - Canvas visibility tracking
 * - Section visibility states
 * - Scroll direction detection
 */

import { create } from 'zustand';

interface ScrollState {
  progress: number;        // 0 to 1
  isCanvasVisible: boolean;
  setProgress: (progress: number) => void;
  setCanvasVisible: (visible: boolean) => void;
}

export const useScrollStore = create<ScrollState>((set) => ({
  progress: 0,
  isCanvasVisible: true,
  
  setProgress: (progress) => set({ progress }),
  setCanvasVisible: (visible) => set({ isCanvasVisible: visible }),
}));
```

### Usage with Scroll Listener

```typescript
function ScrollTracker() {
  const setProgress = useScrollStore(state => state.setProgress);
  
  useEffect(() => {
    const handleScroll = () => {
      const scrollHeight = document.documentElement.scrollHeight - window.innerHeight;
      const progress = window.scrollY / scrollHeight;
      setProgress(Math.min(1, Math.max(0, progress)));
    };
    
    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, [setProgress]);
  
  return null;
}
```

### Usage in Components

```typescript
function ProgressIndicator() {
  // Only re-renders when progress changes
  const progress = useScrollStore(state => state.progress);
  
  return (
    <div 
      className="progress-bar" 
      style={{ width: `${progress * 100}%` }} 
    />
  );
}

function GalaxyCanvas() {
  // Only re-renders when visibility changes
  const isVisible = useScrollStore(state => state.isCanvasVisible);
  
  if (!isVisible) return null;
  
  return <Canvas>...</Canvas>;
}
```

---

### Key Techniques Used

#### 1. Selector Pattern for Minimal Re-renders

```typescript
const progress = useScrollStore(state => state.progress);
```

*Why it matters:* Component only re-renders when selected slice changes.

#### 2. Passive Scroll Listener

```typescript
{ passive: true }
```

*Why it matters:* Browser can optimize scrolling, no waiting for JS.

---

## 💡 Lessons Learned

### What Worked Well

- **Singleton stores** provide clear, centralized state
- **3-way merge** eliminates data loss concerns
- **Zustand selectors** minimize re-renders

### What I'd Do Differently

- Add optimistic updates for better UX
- Implement offline queue for sync failures
- Use IndexedDB for larger data (code history)

### Key Takeaways

1. In-memory tokens are worth the complexity for security
2. Merge strategies should be explicit and documented
3. Zustand's simplicity is a feature, not a limitation

---

## 🔗 Related Patterns

- [Custom Hooks](./custom-hooks.md) - Hooks that consume these stores
- [API Design](../backend/api-design.md) - Server-side token handling
- [Design Patterns](../patterns/design-patterns.md) - Singleton pattern details

---

## 📚 Further Reading

- [Zustand Documentation](https://docs.pmnd.rs/zustand)
- [React useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)
- [JWT Best Practices](https://auth0.com/docs/secure/tokens/json-web-tokens)
- [3-Way Merge Explained](https://en.wikipedia.org/wiki/Merge_(version_control)#Three-way_merge)
