<div align="center">

# 🎯 Design Patterns

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Architectural+Patterns;Factory%2C+Strategy%2C+Singleton" alt="Typing"/>

> **Architectural patterns implemented in AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🏭 1. Factory Pattern

### Context

**Feature:** Dynamic component/service creation  
**Complexity:** Low  
**Implementation:** Widget system, Renderer selection

**The Challenge:**  
Create different types of objects (widgets, renderers) based on runtime conditions without coupling consumer code to specific implementations.

---

### Implementation: Widget Factory

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual widget system has more types and configuration.
 */

// Widget types
interface Widget {
  render(): React.ReactNode;
  validate(): boolean;
}

// Concrete implementations
class TagOrderWidget implements Widget {
  private options: TagOrderOptions;
  
  constructor(options: TagOrderOptions) {
    this.options = options;
  }
  
  render() {
    return <TagOrderUI options={this.options} />;
  }
  
  validate() {
    return this.options.isCorrectOrder;
  }
}

class TextInputWidget implements Widget {
  private value: string;
  private expected: string;
  
  constructor(config: TextInputOptions) {
    this.value = '';
    this.expected = config.expected;
  }
  
  render() {
    return <TextInput onChange={(v) => this.value = v} />;
  }
  
  validate() {
    return this.value.toLowerCase() === this.expected.toLowerCase();
  }
}

// Factory
class WidgetFactory {
  static create(type: string, config: WidgetConfig): Widget {
    const widgets: Record<string, new (config: WidgetConfig) => Widget> = {
      'tag-order': TagOrderWidget,
      'text-input': TextInputWidget,
      'code-input': CodeInputWidget,
      'multiple-choice': MultipleChoiceWidget,
      'attribute-input': AttributeInputWidget,
    };
    
    const WidgetClass = widgets[type];
    
    if (!WidgetClass) {
      throw new Error(`Unknown widget type: ${type}`);
    }
    
    return new WidgetClass(config);
  }
}

// Usage
const widget = WidgetFactory.create('tag-order', {
  correctOrder: ['<!DOCTYPE html>', '<html>', '<head>', '<title>'],
});
```

### Why This Pattern?

| Benefit | Explanation |
|---------|-------------|
| Decoupling | Consumer doesn't know concrete classes |
| Extensibility | New widget types without changing consumer |
| Centralized creation | One place to add validation, logging |

---

## 🎯 2. Strategy Pattern

### Context

**Feature:** Interchangeable algorithms  
**Complexity:** Medium  
**Implementation:** Rendering tiers, Validation strategies

**The Challenge:**  
Use different algorithms (rendering approaches, validation rules) that can be swapped at runtime.

---

### Implementation: Rendering Strategy

```typescript
/**
 * Different rendering strategies based on device capability
 */

interface RenderStrategy {
  name: string;
  initialize(): void;
  render(): React.ReactNode;
  cleanup(): void;
}

class WebGLHighStrategy implements RenderStrategy {
  name = 'webgl-high';
  
  initialize() {
    // Set up WebGL context with full features
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.scene = new THREE.Scene();
  }
  
  render() {
    return (
      <Canvas gl={{ antialias: true }}>
        <Galaxy particleCount={100000} />
        <PostProcessing />
      </Canvas>
    );
  }
  
  cleanup() {
    this.renderer.dispose();
  }
}

class WebGLReducedStrategy implements RenderStrategy {
  name = 'webgl-reduced';
  
  initialize() {
    this.renderer = new THREE.WebGLRenderer({ antialias: false });
  }
  
  render() {
    return (
      <Canvas gl={{ antialias: false, powerPreference: 'low-power' }}>
        <Galaxy particleCount={25000} />
      </Canvas>
    );
  }
  
  cleanup() {
    this.renderer.dispose();
  }
}

class CanvasFallbackStrategy implements RenderStrategy {
  name = 'canvas-fallback';
  
  initialize() {
    // Simple 2D canvas setup
  }
  
  render() {
    return <StaticGradientCanvas />;
  }
  
  cleanup() {
    // Minimal cleanup
  }
}

// Context that uses strategies
class RenderingContext {
  private strategy: RenderStrategy;
  
  constructor(tier: DeviceTier) {
    this.strategy = this.selectStrategy(tier);
    this.strategy.initialize();
  }
  
  private selectStrategy(tier: DeviceTier): RenderStrategy {
    const strategies: Record<DeviceTier, RenderStrategy> = {
      high: new WebGLHighStrategy(),
      medium: new WebGLReducedStrategy(),
      low: new CanvasFallbackStrategy(),
    };
    return strategies[tier];
  }
  
  render() {
    return this.strategy.render();
  }
  
  switchStrategy(tier: DeviceTier) {
    this.strategy.cleanup();
    this.strategy = this.selectStrategy(tier);
    this.strategy.initialize();
  }
}
```

### Why This Pattern?

| Benefit | Explanation |
|---------|-------------|
| Open/Closed | New strategies without modifying context |
| Single Responsibility | Each strategy handles one approach |
| Runtime flexibility | Switch strategies dynamically |

---

## 👤 3. Singleton Pattern

### Context

**Feature:** Global state management  
**Complexity:** Low  
**Implementation:** AuthStore, MongoDB connection

**The Challenge:**  
Ensure only one instance of critical resources (auth state, database connection) exists across the application.

---

### Implementation: Auth Store Singleton

```typescript
/**
 * Singleton for authentication state
 */

class AuthStore {
  private static instance: AuthStore | null = null;
  
  private session: Session | null = null;
  private accessToken: string | null = null;
  
  // Private constructor prevents external instantiation
  private constructor() {
    // Initialize
  }
  
  // Lazy singleton access
  static getInstance(): AuthStore {
    if (!AuthStore.instance) {
      AuthStore.instance = new AuthStore();
    }
    return AuthStore.instance;
  }
  
  // For testing - reset instance
  static resetInstance(): void {
    AuthStore.instance = null;
  }
  
  async getSession(): Promise<Session> {
    // ... implementation
  }
  
  async getAccessToken(): Promise<string | null> {
    // ... implementation
  }
}

// Module-level export for convenience
export const authStore = AuthStore.getInstance();
```

### TypeScript Module Singleton (Simpler)

```typescript
/**
 * Alternative: Module-level singleton
 * Simpler in TypeScript/ES modules
 */

// auth-store.ts
class AuthStoreImpl {
  private session: Session | null = null;
  
  async getSession(): Promise<Session> {
    // ...
  }
}

// Single instance exported from module
export const authStore = new AuthStoreImpl();

// Importing anywhere gets the same instance
import { authStore } from './auth-store';
```

### Why This Pattern?

| Benefit | Explanation |
|---------|-------------|
| Single source of truth | One auth state for entire app |
| Resource efficiency | One DB connection pool |
| State coordination | No race conditions |

---

## 👀 4. Observer Pattern

### Context

**Feature:** Reactive updates across components  
**Complexity:** Medium  
**Implementation:** Progress updates, Auth state changes

**The Challenge:**  
Notify multiple components when state changes without tight coupling.

---

### Implementation: Progress Observer

```typescript
/**
 * Observer pattern for progress updates
 */

type Listener<T> = (data: T) => void;

class Observable<T> {
  private listeners: Set<Listener<T>> = new Set();
  
  subscribe(listener: Listener<T>): () => void {
    this.listeners.add(listener);
    
    // Return unsubscribe function
    return () => {
      this.listeners.delete(listener);
    };
  }
  
  notify(data: T): void {
    this.listeners.forEach(listener => listener(data));
  }
}

// Progress store with observable
class ProgressStore {
  private progressObservable = new Observable<ModuleProgress>();
  
  onProgressChange(listener: (progress: ModuleProgress) => void): () => void {
    return this.progressObservable.subscribe(listener);
  }
  
  async saveProgress(module: string, progress: ModuleProgress): Promise<void> {
    // Save to localStorage
    localStorage.setItem(`progress_${module}`, JSON.stringify(progress));
    
    // Notify all observers
    this.progressObservable.notify(progress);
    
    // Custom event for cross-tab sync
    window.dispatchEvent(new Event('progress-update'));
  }
}

// React hook using observer
function useProgress(module: string) {
  const [progress, setProgress] = useState<ModuleProgress | null>(null);
  
  useEffect(() => {
    // Subscribe to changes
    const unsubscribe = progressStore.onProgressChange(setProgress);
    
    // Load initial
    progressStore.getProgress(module).then(setProgress);
    
    return unsubscribe;
  }, [module]);
  
  return progress;
}
```

### Browser Event Observer

```typescript
/**
 * Cross-tab communication via storage events
 */

function useCrossTabSync() {
  useEffect(() => {
    const handleStorageChange = (event: StorageEvent) => {
      if (event.key?.startsWith('progress_')) {
        const module = event.key.replace('progress_', '');
        const newProgress = event.newValue 
          ? JSON.parse(event.newValue) 
          : null;
        
        // Update local state
        progressStore.updateLocal(module, newProgress);
      }
    };
    
    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, []);
}
```

### Why This Pattern?

| Benefit | Explanation |
|---------|-------------|
| Decoupling | Publisher doesn't know subscribers |
| Multiple subscribers | Many components react to one event |
| Reactive updates | UI stays in sync automatically |

---

## 🎨 5. Decorator Pattern

### Context

**Feature:** Add functionality without modifying base classes  
**Complexity:** Medium  
**Implementation:** Middleware, logging wrappers

**The Challenge:**  
Extend behavior (logging, caching, validation) without modifying original code.

---

### Implementation: API Handler Decorators

```typescript
/**
 * Functional decorators for API handlers
 */

type APIHandler = (request: NextRequest) => Promise<NextResponse>;

// Logging decorator
function withLogging(handler: APIHandler): APIHandler {
  return async (request: NextRequest) => {
    const start = Date.now();
    const requestId = crypto.randomUUID();
    
    console.log(`[${requestId}] ${request.method} ${request.url}`);
    
    try {
      const response = await handler(request);
      const duration = Date.now() - start;
      console.log(`[${requestId}] ${response.status} - ${duration}ms`);
      return response;
    } catch (error) {
      console.error(`[${requestId}] Error:`, error);
      throw error;
    }
  };
}

// Caching decorator
function withCache(ttlSeconds: number): (handler: APIHandler) => APIHandler {
  return (handler: APIHandler) => {
    const cache = new Map<string, { data: NextResponse; expires: number }>();
    
    return async (request: NextRequest) => {
      const cacheKey = request.url;
      const cached = cache.get(cacheKey);
      
      if (cached && cached.expires > Date.now()) {
        return cached.data;
      }
      
      const response = await handler(request);
      
      cache.set(cacheKey, {
        data: response,
        expires: Date.now() + ttlSeconds * 1000,
      });
      
      return response;
    };
  };
}

// Auth decorator
function withAuth(handler: APIHandler): APIHandler {
  return async (request: NextRequest) => {
    const userId = await validateAuth(request);
    
    if (!userId) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    // Attach userId to request for handler
    request.headers.set('x-user-id', userId);
    return handler(request);
  };
}

// Compose decorators
const protectedHandler = withLogging(
  withAuth(
    async (request: NextRequest) => {
      const userId = request.headers.get('x-user-id');
      // Handle request...
      return NextResponse.json({ success: true });
    }
  )
);

// Or using a compose helper
function compose(...decorators: ((h: APIHandler) => APIHandler)[]) {
  return (handler: APIHandler) =>
    decorators.reduceRight((h, decorator) => decorator(h), handler);
}

const composedHandler = compose(withLogging, withAuth)(baseHandler);
```

### Why This Pattern?

| Benefit | Explanation |
|---------|-------------|
| Open/Closed | Add features without changing handlers |
| Composable | Mix and match decorators |
| Single Responsibility | Each decorator does one thing |

---

## 🔄 6. Command Pattern

### Context

**Feature:** Encapsulate actions as objects  
**Complexity:** Medium  
**Implementation:** Undo/redo, action history

**The Challenge:**  
Allow actions to be queued, logged, or reversed.

---

### Implementation: Editor Commands

```typescript
/**
 * Command pattern for code editor actions
 */

interface Command {
  execute(): void;
  undo(): void;
  description: string;
}

class InsertTextCommand implements Command {
  private editor: EditorState;
  private text: string;
  private position: Position;
  
  constructor(editor: EditorState, text: string, position: Position) {
    this.editor = editor;
    this.text = text;
    this.position = position;
  }
  
  get description() {
    return `Insert "${this.text.substring(0, 20)}..."`;
  }
  
  execute() {
    this.editor.insertAt(this.position, this.text);
  }
  
  undo() {
    this.editor.deleteAt(this.position, this.text.length);
  }
}

class DeleteTextCommand implements Command {
  private editor: EditorState;
  private deletedText: string = '';
  private range: Range;
  
  constructor(editor: EditorState, range: Range) {
    this.editor = editor;
    this.range = range;
  }
  
  get description() {
    return `Delete text`;
  }
  
  execute() {
    this.deletedText = this.editor.getTextInRange(this.range);
    this.editor.deleteRange(this.range);
  }
  
  undo() {
    this.editor.insertAt(this.range.start, this.deletedText);
  }
}

// Command invoker with history
class CommandInvoker {
  private history: Command[] = [];
  private position: number = -1;
  
  execute(command: Command): void {
    // Remove any forward history
    this.history = this.history.slice(0, this.position + 1);
    
    // Execute and add to history
    command.execute();
    this.history.push(command);
    this.position++;
  }
  
  undo(): boolean {
    if (this.position < 0) return false;
    
    this.history[this.position].undo();
    this.position--;
    return true;
  }
  
  redo(): boolean {
    if (this.position >= this.history.length - 1) return false;
    
    this.position++;
    this.history[this.position].execute();
    return true;
  }
  
  getHistory(): string[] {
    return this.history.map(cmd => cmd.description);
  }
}
```

### Why This Pattern?

| Benefit | Explanation |
|---------|-------------|
| Action encapsulation | Each action is self-contained |
| Undo/Redo | Natural support for reversible actions |
| Logging/Audit | Easy to track all actions |

---

## 💡 Lessons Learned

### Pattern Selection Guidelines

| Situation | Recommended Pattern |
|-----------|-------------------|
| Multiple similar objects | Factory |
| Interchangeable algorithms | Strategy |
| Global shared state | Singleton |
| Event notification | Observer |
| Add behavior to functions | Decorator |
| Reversible actions | Command |

### Key Takeaways

1. **Don't over-engineer**: Use patterns when they solve real problems
2. **Composition over inheritance**: TypeScript favors functional approaches
3. **Keep it simple**: The simplest pattern that solves the problem is best

---

## 🔗 Related Patterns

- [Custom Hooks](../frontend/custom-hooks.md) - Hook factories and strategies
- [State Management](../frontend/state-management.md) - Singleton stores
- [Middleware](../backend/middleware-pattern.md) - Decorator pattern

---

## 📚 Further Reading

- [Patterns.dev](https://patterns.dev/)
- [Refactoring Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [TypeScript Design Patterns](https://www.typescriptlang.org/docs/handbook/2/objects.html)
