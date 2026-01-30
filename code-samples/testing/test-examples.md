<div align="center">

# ✅ Test Examples

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Testing+Strategies;Unit+%26+Integration+Tests" alt="Typing"/>

> **Testing strategies and examples from AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🧪 1. Unit Tests for Custom Hooks

### Context

**Feature:** Testing hooks in isolation  
**Complexity:** Medium  
**Technologies:** Jest, React Testing Library

**The Challenge:**  
Test hooks that use browser APIs, async operations, and state updates.

---

### Implementation: Testing useDeviceTier

```typescript
/**
 * DISCLAIMER: Simplified test examples.
 * Actual tests include more edge cases.
 */

import { renderHook, act, waitFor } from '@testing-library/react';
import { useDeviceTier } from '@/lib/hooks/useDeviceTier';

describe('useDeviceTier', () => {
  // Save original navigator
  const originalNavigator = global.navigator;

  afterEach(() => {
    // Restore navigator after each test
    Object.defineProperty(global, 'navigator', {
      value: originalNavigator,
      writable: true,
    });
  });

  it('should return "low" when navigator is undefined (SSR)', () => {
    // Simulate SSR environment
    Object.defineProperty(global, 'navigator', {
      value: undefined,
      writable: true,
    });

    const { result } = renderHook(() => useDeviceTier());
    
    expect(result.current).toBe('low');
  });

  it('should initially return "low" for hydration safety', () => {
    Object.defineProperty(navigator, 'hardwareConcurrency', {
      value: 8,
      configurable: true,
    });

    const { result } = renderHook(() => useDeviceTier());
    
    // Initially low for SSR consistency
    expect(result.current).toBe('low');
  });

  it('should upgrade to "high" after hydration on powerful devices', async () => {
    Object.defineProperty(navigator, 'hardwareConcurrency', {
      value: 8,
      configurable: true,
    });

    const { result } = renderHook(() => useDeviceTier());

    // Wait for useEffect
    await waitFor(() => {
      expect(result.current).toBe('high');
    });
  });

  it('should return "medium" for devices with 2-5 cores', async () => {
    Object.defineProperty(navigator, 'hardwareConcurrency', {
      value: 4,
      configurable: true,
    });

    const { result } = renderHook(() => useDeviceTier());

    await waitFor(() => {
      expect(result.current).toBe('medium');
    });
  });
});
```

### Testing useGalaxyWorker

```typescript
/**
 * Testing Web Worker hook
 */

// Mock Web Worker
class MockWorker {
  onmessage: ((event: MessageEvent) => void) | null = null;
  
  postMessage(data: unknown) {
    // Simulate worker processing
    setTimeout(() => {
      const mockResults = {
        positions: new Float32Array(data.count * 3),
        colors: new Float32Array(data.count * 3),
      };
      
      this.onmessage?.({ data: mockResults } as MessageEvent);
    }, 10);
  }
  
  terminate() {}
}

// Mock the Worker constructor
global.Worker = MockWorker as unknown as typeof Worker;

describe('useGalaxyWorker', () => {
  it('should return null initially while loading', () => {
    const { result } = renderHook(() => 
      useGalaxyWorker({
        count: 1000,
        radius: 10,
        branches: 5,
        spin: 1,
      })
    );

    expect(result.current.results).toBeNull();
    expect(result.current.loading).toBe(true);
  });

  it('should return results after worker completes', async () => {
    const { result } = renderHook(() => 
      useGalaxyWorker({
        count: 1000,
        radius: 10,
        branches: 5,
        spin: 1,
      })
    );

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.results).not.toBeNull();
      expect(result.current.results?.positions).toBeInstanceOf(Float32Array);
      expect(result.current.results?.colors).toBeInstanceOf(Float32Array);
    });
  });

  it('should have correct array lengths', async () => {
    const count = 1000;
    
    const { result } = renderHook(() => 
      useGalaxyWorker({
        count,
        radius: 10,
        branches: 5,
        spin: 1,
      })
    );

    await waitFor(() => {
      expect(result.current.results?.positions.length).toBe(count * 3);
      expect(result.current.results?.colors.length).toBe(count * 3);
    });
  });
});
```

---

## 🔧 2. Component Testing

### Context

**Feature:** Testing React components  
**Complexity:** Medium  
**Technologies:** React Testing Library, Jest

---

### Testing TaskChecklist

```typescript
/**
 * Testing live validation component
 */

import { render, screen } from '@testing-library/react';
import { TaskChecklist } from '@/components/TaskChecklist';

describe('TaskChecklist', () => {
  const requirements = [
    {
      id: 'has-header',
      label: 'Add a <header> element',
      type: 'elementExists' as const,
      selector: 'header',
    },
    {
      id: 'has-nav',
      label: 'Add navigation inside header',
      type: 'elementExists' as const,
      selector: 'header nav',
    },
  ];

  it('should show all requirements as pending initially', () => {
    render(
      <TaskChecklist
        code="<html></html>"
        requirements={requirements}
        onAllComplete={() => {}}
      />
    );

    expect(screen.getByText('Add a <header> element')).toBeInTheDocument();
    expect(screen.getByText('Add navigation inside header')).toBeInTheDocument();
    
    // Both should have pending icons
    expect(screen.getAllByText('⬜')).toHaveLength(2);
  });

  it('should mark requirement as passed when code satisfies it', () => {
    render(
      <TaskChecklist
        code="<html><header></header></html>"
        requirements={requirements}
        onAllComplete={() => {}}
      />
    );

    // Header requirement passed
    expect(screen.getByText('✅')).toBeInTheDocument();
    // Nav requirement still pending
    expect(screen.getByText('⬜')).toBeInTheDocument();
  });

  it('should mark all passed when all requirements met', () => {
    render(
      <TaskChecklist
        code="<html><header><nav></nav></header></html>"
        requirements={requirements}
        onAllComplete={() => {}}
      />
    );

    expect(screen.getAllByText('✅')).toHaveLength(2);
    expect(screen.queryByText('⬜')).not.toBeInTheDocument();
  });

  it('should call onAllComplete when all requirements pass', () => {
    const onComplete = jest.fn();
    
    render(
      <TaskChecklist
        code="<html><header><nav></nav></header></html>"
        requirements={requirements}
        onAllComplete={onComplete}
      />
    );

    expect(onComplete).toHaveBeenCalled();
  });

  it('should not call onAllComplete when not all pass', () => {
    const onComplete = jest.fn();
    
    render(
      <TaskChecklist
        code="<html><header></header></html>"
        requirements={requirements}
        onAllComplete={onComplete}
      />
    );

    expect(onComplete).not.toHaveBeenCalled();
  });
});
```

### Testing Context-Aware Hero

```typescript
/**
 * Testing component with auth state
 */

import { render, screen, waitFor } from '@testing-library/react';
import { ContextAwareHero } from '@/components/ContextAwareHero';
import { authStore } from '@/lib/stores/authStore';
import { progressStore } from '@/lib/stores/progressStore';

// Mock stores
jest.mock('@/lib/stores/authStore', () => ({
  authStore: {
    getSession: jest.fn(),
  },
}));

jest.mock('@/lib/stores/progressStore', () => ({
  progressStore: {
    getProgress: jest.fn(),
  },
}));

describe('ContextAwareHero', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should render guest variant for unauthenticated user', async () => {
    (authStore.getSession as jest.Mock).mockResolvedValue({
      isAuthenticated: false,
      user: { id: 'anon-123' },
    });
    (progressStore.getProgress as jest.Mock).mockResolvedValue({
      completedLessons: [],
    });

    render(<ContextAwareHero />);

    await waitFor(() => {
      expect(screen.getByText('Start Your Space Journey')).toBeInTheDocument();
      expect(screen.getByText('Begin Exploration')).toBeInTheDocument();
    });
  });

  it('should render inProgress variant for authenticated user with progress', async () => {
    (authStore.getSession as jest.Mock).mockResolvedValue({
      isAuthenticated: true,
      user: { id: 'user-123', displayName: 'Ahmed' },
    });
    (progressStore.getProgress as jest.Mock).mockResolvedValue({
      completedLessons: [1, 2, 3],
    });

    render(<ContextAwareHero />);

    await waitFor(() => {
      expect(screen.getByText('Journey in Progress')).toBeInTheDocument();
      expect(screen.getByText('Continue')).toBeInTheDocument();
    });
  });

  it('should have SSR-safe initial render', () => {
    (authStore.getSession as jest.Mock).mockResolvedValue({
      isAuthenticated: false,
      user: { id: 'anon' },
    });
    (progressStore.getProgress as jest.Mock).mockResolvedValue({
      completedLessons: [],
    });

    render(<ContextAwareHero />);

    // Initial render should be guest (SSR-safe)
    expect(screen.getByRole('heading', { level: 1 })).toBeInTheDocument();
  });
});
```

---

## 🔗 3. Integration Tests

### Context

**Feature:** Testing API routes end-to-end  
**Complexity:** High  
**Technologies:** Jest, Supertest pattern, MongoDB Memory Server

---

### Testing Progress API

```typescript
/**
 * Integration tests for progress API
 */

import { NextRequest } from 'next/server';
import { GET, POST } from '@/app/api/progress/route';
import { MongoMemoryServer } from 'mongodb-memory-server';
import { MongoClient } from 'mongodb';

describe('Progress API', () => {
  let mongoServer: MongoMemoryServer;
  let client: MongoClient;
  
  beforeAll(async () => {
    // Start in-memory MongoDB
    mongoServer = await MongoMemoryServer.create();
    const uri = mongoServer.getUri();
    
    // Override env
    process.env.MONGODB_URI = uri;
    process.env.MONGODB_DB = 'test';
    
    client = new MongoClient(uri);
    await client.connect();
  });
  
  afterAll(async () => {
    await client.close();
    await mongoServer.stop();
  });
  
  beforeEach(async () => {
    // Clear database before each test
    const db = client.db('test');
    await db.collection('user_progress').deleteMany({});
  });

  describe('GET /api/progress', () => {
    it('should return empty progress for new user', async () => {
      const request = createAuthenticatedRequest('GET', {
        userId: 'new-user-123',
      });
      
      const response = await GET(request);
      const data = await response.json();
      
      expect(response.status).toBe(200);
      expect(data).toEqual({ html: {}, internet: {} });
    });

    it('should return saved progress', async () => {
      // Seed data
      const db = client.db('test');
      await db.collection('user_progress').insertOne({
        userId: 'user-123',
        html: { completedLessons: [1, 2, 3] },
      });
      
      const request = createAuthenticatedRequest('GET', {
        userId: 'user-123',
      });
      
      const response = await GET(request);
      const data = await response.json();
      
      expect(response.status).toBe(200);
      expect(data.html.completedLessons).toEqual([1, 2, 3]);
    });

    it('should return 401 without auth token', async () => {
      const request = new NextRequest('http://localhost/api/progress');
      
      const response = await GET(request);
      
      expect(response.status).toBe(401);
    });
  });

  describe('POST /api/progress', () => {
    it('should save new progress', async () => {
      const request = createAuthenticatedRequest('POST', {
        userId: 'user-123',
        body: {
          module: 'html',
          completedLessons: [1, 2],
        },
      });
      
      const response = await POST(request);
      
      expect(response.status).toBe(200);
      
      // Verify in database
      const db = client.db('test');
      const saved = await db.collection('user_progress').findOne({ 
        userId: 'user-123' 
      });
      expect(saved?.html.completedLessons).toContain(1);
      expect(saved?.html.completedLessons).toContain(2);
    });

    it('should merge with existing progress', async () => {
      const db = client.db('test');
      await db.collection('user_progress').insertOne({
        userId: 'user-123',
        html: { completedLessons: [1, 2] },
      });
      
      const request = createAuthenticatedRequest('POST', {
        userId: 'user-123',
        body: {
          module: 'html',
          completedLessons: [3, 4],
        },
      });
      
      await POST(request);
      
      const saved = await db.collection('user_progress').findOne({ 
        userId: 'user-123' 
      });
      expect(saved?.html.completedLessons).toEqual(
        expect.arrayContaining([1, 2, 3, 4])
      );
    });
  });
});

// Helper to create authenticated request
function createAuthenticatedRequest(
  method: string,
  options: { userId: string; body?: unknown }
) {
  const url = 'http://localhost/api/progress';
  const token = createTestToken(options.userId);
  
  return new NextRequest(url, {
    method,
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: options.body ? JSON.stringify(options.body) : undefined,
  });
}

function createTestToken(userId: string): string {
  // Simple test token (not for production)
  const payload = { userId, exp: Date.now() + 3600000 };
  return Buffer.from(JSON.stringify(payload)).toString('base64');
}
```

---

## 🎭 4. Mocking Strategies

### Mocking Browser APIs

```typescript
/**
 * Common browser API mocks
 */

// Mock localStorage
const localStorageMock = (() => {
  let store: Record<string, string> = {};
  
  return {
    getItem: (key: string) => store[key] ?? null,
    setItem: (key: string, value: string) => { store[key] = value; },
    removeItem: (key: string) => { delete store[key]; },
    clear: () => { store = {}; },
    get length() { return Object.keys(store).length; },
    key: (index: number) => Object.keys(store)[index] ?? null,
  };
})();

Object.defineProperty(window, 'localStorage', {
  value: localStorageMock,
});

// Mock IntersectionObserver
class MockIntersectionObserver {
  callback: IntersectionObserverCallback;
  
  constructor(callback: IntersectionObserverCallback) {
    this.callback = callback;
  }
  
  observe(_target: Element) {}
  unobserve(_target: Element) {}
  disconnect() {}
  
  // Test helper to trigger intersection
  trigger(isIntersecting: boolean) {
    this.callback([{ isIntersecting }] as IntersectionObserverEntry[], this);
  }
}

global.IntersectionObserver = MockIntersectionObserver as unknown as typeof IntersectionObserver;

// Mock crypto.randomUUID
Object.defineProperty(global.crypto, 'randomUUID', {
  value: () => 'test-uuid-' + Math.random().toString(36).substr(2, 9),
});
```

### Mocking Fetch

```typescript
/**
 * Fetch mocking patterns
 */

// Simple mock
global.fetch = jest.fn();

beforeEach(() => {
  (fetch as jest.Mock).mockClear();
});

it('should handle successful API response', async () => {
  (fetch as jest.Mock).mockResolvedValueOnce({
    ok: true,
    json: async () => ({ data: 'test' }),
  });

  const result = await myApiFunction();
  
  expect(result.data).toBe('test');
  expect(fetch).toHaveBeenCalledWith(
    expect.stringContaining('/api/'),
    expect.any(Object)
  );
});

it('should handle API error', async () => {
  (fetch as jest.Mock).mockResolvedValueOnce({
    ok: false,
    status: 500,
    json: async () => ({ error: 'Server error' }),
  });

  await expect(myApiFunction()).rejects.toThrow('Server error');
});

// MSW for more realistic mocking
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const server = setupServer(
  http.get('/api/progress', () => {
    return HttpResponse.json({
      html: { completedLessons: [1, 2] },
    });
  }),
  
  http.post('/api/progress', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ success: true });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

---

## 📊 5. Test Organization

### Jest Configuration

```typescript
// jest.config.ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss)$': 'identity-obj-proxy',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};

export default config;
```

### Test File Organization

```
__tests__/
├── unit/
│   ├── hooks/
│   │   ├── useDeviceTier.test.ts
│   │   ├── useGalaxyWorker.test.ts
│   │   └── useGatekeeper.test.ts
│   ├── components/
│   │   ├── TaskChecklist.test.tsx
│   │   └── ContextAwareHero.test.tsx
│   └── utils/
│       └── validation.test.ts
├── integration/
│   ├── api/
│   │   ├── progress.test.ts
│   │   └── auth.test.ts
│   └── stores/
│       ├── authStore.test.ts
│       └── progressStore.test.ts
└── e2e/
    └── flows/
        └── lessonCompletion.test.ts
```

---

## 💡 Lessons Learned

### What Worked Well

- **Testing Library** philosophy: test user behavior, not implementation
- **MongoDB Memory Server** for fast, isolated integration tests
- **Mock factories** reduce test setup boilerplate

### What I'd Do Differently

- Add visual regression tests earlier
- Implement contract tests for API consumers
- Set up test fixtures factory

### Key Takeaways

1. Mock browser APIs at the lowest level
2. Integration tests catch bugs unit tests miss
3. Test coverage thresholds keep quality high

---

## 🔗 Related Patterns

- [Custom Hooks](../frontend/custom-hooks.md) - Hooks being tested
- [API Design](../backend/api-design.md) - APIs being tested
- [Design Patterns](../patterns/design-patterns.md) - Testable patterns

---

## 📚 Further Reading

- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [MSW - Mock Service Worker](https://mswjs.io/)
- [Testing Best Practices](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
