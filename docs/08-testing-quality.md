<div align="center">

# 🧪 08 - Testing & Quality

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Quality+Assurance;Testing+Strategies;Code+Standards" alt="Typing"/>

> **Quality assurance approach and testing strategies for AstroShan**

<br/>

[![Prev](https://img.shields.io/badge/←_Performance-1eb8e4?style=for-the-badge)](07-performance.md)
[![Next](https://img.shields.io/badge/Deployment_→-7565e3?style=for-the-badge)](09-deployment.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🎯 Testing Philosophy

AstroShan follows the **Testing Trophy** model:

```
         ▲
        /║\       E2E Tests
       / ║ \      (Few, critical flows)
      /──║──\
     /   ║   \    Integration Tests
    /    ║    \   (Key feature workflows)
   /─────║─────\
  /      ║      \ Static Analysis
 ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀ (TypeScript, ESLint)
```

### Priority Order

1. **Static Analysis** - Catch errors at compile time
2. **Integration Tests** - Test feature workflows
3. **E2E Tests** - Verify critical user journeys
4. **Unit Tests** - For complex utility functions

---

## 📊 Static Analysis

### TypeScript Strict Mode

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUncheckedIndexedAccess": true
  }
}
```

### Impact

| Metric | Value |
|:------:|:-----:|
| TypeScript Coverage | 100% |
| `.js` files | 0 |
| `any` types | 0 in production |

### ESLint Configuration

```javascript
{
  rules: {
    "no-console": ["warn", { allow: ["warn", "error"] }],
    "react-hooks/exhaustive-deps": "error",
    "@typescript-eslint/no-unused-vars": "error"
  }
}
```

---

## 🧪 Testing Categories

### 1. Integration Tests

```typescript
describe("Progress Sync", () => {
  it("should merge anonymous progress on login", async () => {
    // Setup anonymous progress
    localStorage.setItem("htmlLessonsProgress", JSON.stringify({
      savedCompleted: [1, 2, 3]
    }));
    
    // Mock cloud progress
    mockMongoAdapter.fetchProgress.mockResolvedValue({
      html: { completedLessons: [1, 5, 7] }
    });
    
    // User logs in
    await progressStore.syncUser("user-123");
    
    // Merged result
    const local = await progressStore.getHtmlProgress();
    expect(local.completedLessons).toEqual(
      expect.arrayContaining([1, 2, 3, 5, 7])
    );
  });
});
```

### 2. API Route Testing

```typescript
describe("POST /api/auth/token/refresh", () => {
  it("should issue new access token", async () => {
    const validRefreshToken = createTestToken({ userId: "user-123" });
    
    const response = await POST(new NextRequest("/api/auth/token/refresh", {
      headers: { Cookie: `refreshToken=${validRefreshToken}` }
    }));
    
    expect(response.status).toBe(200);
    expect((await response.json()).accessToken).toBeDefined();
  });
});
```

---

## ♿ Accessibility Testing

### Automated Checks (Jest-axe)

```typescript
import { axe, toHaveNoViolations } from "jest-axe";
expect.extend(toHaveNoViolations);

describe("Interactive Widgets", () => {
  it("should have no accessibility violations", async () => {
    const { container } = render(<WidgetContainer />);
    expect(await axe(container)).toHaveNoViolations();
  });
});
```

### Keyboard Navigation

```typescript
it("should navigate with arrow keys", async () => {
  render(<HtmlLessonsList />);
  
  const firstLesson = screen.getByRole("button", { name: /lesson 1/i });
  firstLesson.focus();
  await userEvent.keyboard("{ArrowDown}");
  
  expect(screen.getByRole("button", { name: /lesson 2/i })).toHaveFocus();
});
```

---

## 🔄 CI/CD Integration

### GitHub Actions

```yaml
name: Quality Checks

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: treosh/lighthouse-ci-action@v10
        with:
          configPath: "./lighthouserc.js"
```

### Lighthouse Thresholds

```javascript
{
  assertions: {
    "categories:performance": ["error", { minScore: 0.75 }],
    "categories:accessibility": ["error", { minScore: 0.90 }],
    "first-contentful-paint": ["error", { maxNumericValue: 2000 }]
  }
}
```

---

## 📋 Quality Metrics

| Category | Metric |
|:---------|:------:|
| TypeScript Coverage | 100% |
| ESLint Compliance | 0 errors |
| A11y Violations | 0 critical |
| Lighthouse Performance | > 75 |
| Lighthouse Accessibility | > 90 |

---

## 🎓 Quality Learnings

1. **Static analysis catches most bugs** - TypeScript strict mode is worth it
2. **Integration > Unit tests** - Test workflows, not isolated functions
3. **Accessibility must be automated** - Manual testing always misses something
4. **Performance is a feature** - Treat regressions like bugs

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 Related Documents

| Navigation |
|:----------:|
| [⚡ ← Performance](07-performance.md) |
| [🚀 Deployment →](09-deployment.md) |

---

*Next: [09 - Deployment & DevOps](09-deployment.md)*
