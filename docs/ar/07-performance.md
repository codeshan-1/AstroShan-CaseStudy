<div align="center" dir="rtl">

# ⚡ 07 - تحسين الأداء

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=82%25+TBT+Reduction;Performance+Engineering;Optimization+Strategies" alt="Typing"/>

> **كيف حققنا انخفاضاً بنسبة 82% في زمن الحجب الكلي (TBT)**

<br/>

[![Prev](https://img.shields.io/badge/%E2%86%90_%D8%A7%D9%84%D8%AA%D8%AD%D8%AF%D9%8A%D8%A7%D8%AA-1eb8e4)](06-challenges-solutions.md)
[![Next: Testing & Quality](https://img.shields.io/badge/Next_→_Testing_%26_Quality-7565e3?style=for-the-badge)](08-testing-quality.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

<div dir="rtl">

---

## 📊 خط الأساس الأولي (Initial Baseline)

### قبل التحسين

| المقياس | القيمة | الهدف | الحالة |
|:------:|:-----:|:------:|:------:|
| TBT | 14,000ms | < 3,000ms | ❌ فشل |
| FCP | 1.8s | < 1.0s | ⚠️ تحذير |
| LCP | 2.4s | < 2.5s | ⚠️ حدي |
| TTI | 16s | < 5s | ❌ فشل |

### 🔍 تحديد عنق الزجاجة (Bottleneck Identification)

كشفت Chrome DevTools:
- **72% TBT**: دالة `generateParticles()` (المجرة)
- **15% TBT**: حزمة Framer Motion
- **8% TBT**: تهيئة محرر Monaco
- **5% TBT**: أخرى

---

## ⚡ التحسين 1: مجرة الـ Web Worker

### المشكلة
توليد 100k جسيم حجب الـ main thread لمدة 14 ثانية.

### الحل

```typescript
// useGalaxyWorker.ts
const worker = useMemo(() => 
  new Worker(new URL("../workers/galaxy.worker.ts", import.meta.url)),
[]);

useEffect(() => {
  worker.postMessage(params);
  worker.onmessage = (e) => {
    // نقل Zero-copy باستخدام Transferable
    updatePositions(e.data);
  };
}, []);
```

### النتائج

| قبل | بعد | التحسن |
|:------:|:-----:|:-----------:|
| 14,000ms | 2,500ms | **-82%** |

---

## 🎬 التحسين 2: LazyMotion

### المشكلة
تضيف Framer Motion أكثر من 30KB للحزمة.

### الحل

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
<m.div animate={{ opacity: 1 }} /> // ليس `motion.div`
```

### النتائج
- تم تفعيل Tree-shaking
- تحميل الميزات المستخدمة فقط
- الوضع الصارم (Strict mode) يلتقط الانتهاكات

---

## 📦 التحسين 3: الاستيراد الديناميكي (Dynamic Imports)

### المشكلة
يتم تحميل المكونات الثقيلة عند العرض الأولي.

### الحل

```typescript
// تحميل كسول للمكونات الثقيلة
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

## 🖥️ التحسين 4: SSR لـ LCP

### المشكلة
نص الـ Hero انتظر الـ JavaScript، مما أضر بـ LCP.

### الحل

```typescript
// نص Hero يعرض على السيرفر
export default function Hero() {
  return (
    <h1>Start Your Coding Journey</h1> // متاح فوراً
  );
}
```

```typescript
// تخصيص من جانب العميل بعد الـ hydration
useEffect(() => {
  const session = await getSession();
  if (session.name) setTitle(`Welcome, ${session.name}`);
}, []);
```

### النتائج
- LCP: ~0ms للنص
- تعزيز تدريجي (Progressive enhancement)

---

## 🎨 التحسين 5: CSS الحرج (Critical CSS)

### تكامل Critters

```javascript
// next.config.ts
const withCritters = require("critters-webpack-plugin");

module.exports = withCritters({
  preload: "swap",
  pruneSource: false
});
```

### الفوائد
- CSS الجزء العلوي (Above-fold) مضمن (Inlined)
- CSS المتبقي يتم تحميله بشكل غير متزامن
- FCP أسرع

---

## 📊 النتائج النهائية

| المقياس | قبل | بعد | التحسن |
|:------:|:------:|:-----:|:-----------:|
| **TBT** | 14,000ms | 2,500ms | **-82%** ✅ |
| **FCP** | 1.8s | 1.0s | **-44%** ✅ |
| **LCP** | 2.4s | ~0s (SSR) | **-100%** ✅ |
| **TTI** | 16s | 4s | **-75%** ✅ |

---

## 🎓 الدروس المستفادة

1. **قس قبل أن تحسن** - كشف التنميط (Profiling) عن عنق زجاجة مفاجئ
2. **Web Workers غير مستغلة كفاية** - أي عمل مرتبط بـ CPU يستفيد منها
3. **التعزيز التدريجي يعمل** - اعرض بالسيرفر أولاً، ثم عزز بالعميل
4. **تحليل الحزمة ضروري** - اعرف ما ترسله للمستخدم

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 وثائق ذات صلة

<div align="center">

| التنقل |
|:----------:|
| [💡 ← التحديات](06-challenges-solutions.md) |
| [🧪 الاختبار والجودة →](08-testing-quality.md) |

</div>

<div align="center">

[![Prev: Challenges & Solutions](https://img.shields.io/badge/←_Prev:_Challenges_%26_Solutions-1eb8e4?style=for-the-badge)](06-challenges-solutions.md) [![Next: Testing & Quality](https://img.shields.io/badge/Next:_Testing_%26_Quality_→-7565e3?style=for-the-badge)](08-testing-quality.md)

</div>
