<div align="center" dir="rtl">

# 🧪 08 - الاختبار والجودة

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Quality+Assurance;Testing+Strategies;Code+Standards" alt="Typing"/>

> **نهج ضمان الجودة واستراتيجيات الاختبار في AstroShan**

<br/>

[![Prev](https://img.shields.io/badge/←_الأداء-1eb8e4?style=for-the-badge)](07-performance.md)
[![Next](https://img.shields.io/badge/النشر_→-7565e3?style=for-the-badge)](09-deployment.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

<div dir="rtl">

---

## 🎯 فلسفة الاختبار (Testing Philosophy)

يتبع AstroShan نموذج **Testing Trophy**:

```
         ▲
        /║\       E2E Tests
       / ║ \      (اختبارات شاملة قليلة وحرجة)
      /──║──\
     /   ║   \    Integration Tests
    /    ║    \   (تدفقات الميزات الرئيسية)
   /─────║─────\
  /      ║      \ Static Analysis
 ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀ (TypeScript, ESLint)
```

### ترتيب الأولوية

1. **التحليل الساكن (Static Analysis)** - التقاط الأخطاء وقت الـ Compile
2. **اختبارات التكامل (Integration Tests)** - اختبار سير عمل الميزات
3. **الاختبارات الشاملة (E2E Tests)** - التحقق من رحلات المستخدم الحرجة
4. **اختبارات الوحدة (Unit Tests)** - للدوال المساعدة المعقدة فقط

---

## 📊 التحليل الساكن (Static Analysis)

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

### الأثر

| المقياس | القيمة |
|:------:|:-----:|
| تغطية TypeScript | 100% |
| ملفات `.js` | 0 |
| أنواع `any` | 0 في الإنتاج |

### تكوين ESLint

```javascript
{
  "rules": {
    "no-console": ["warn", { "allow": ["warn", "error"] }],
    "react-hooks/exhaustive-deps": "error",
    "@typescript-eslint/no-unused-vars": "error"
  }
}
```

---

## 🧪 فئات الاختبار (Testing Categories)

### 1. اختبارات التكامل (Integration Tests)

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

### 2. اختبار مسارات الـ API

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

## ♿ اختبار إمكانية الوصول (Accessibility Testing)

### الفحوصات الآلية (Jest-axe)

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

### التنقل بلوحة المفاتيح

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

## 🔄 تكامل CI/CD

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

---

## 📋 مقاييس الجودة (Quality Metrics)

| الفئة | المقياس |
|:---------|:------:|
| تغطية TypeScript | 100% |
| امتثال ESLint | 0 أخطاء |
| انتهاكات A11y | 0 حرجة |
| أداء Lighthouse | > 75 |
| إمكانية الوصول Lighthouse | > 90 |

---

## 🎓 دروس الجودة المستفادة

1. **التحليل الساكن يمسك بمعظم الأخطاء** - الوضع الصارم في TypeScript يستحق العناء
2. **التكامل > اختبارات الوحدة** - اختبر تدفقات العمل، وليس الدوال المعزولة
3. **يجب أتمتة إمكانية الوصول** - الاختبار اليدوي دائماً ما يفوته شيء
4. **الأداء ميزة** - تعامل مع التراجع في الأداء كخطأ برمجي (Bug)

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 وثائق ذات صلة

<div align="center">

| التنقل |
|:----------:|
| [⚡ ← الأداء](07-performance.md) |
| [🚀 النشر →](09-deployment.md) |

</div>

<div align="center">

---

*التالي: [09 - النشر و DevOps](09-deployment.md)*

</div>

</div>
