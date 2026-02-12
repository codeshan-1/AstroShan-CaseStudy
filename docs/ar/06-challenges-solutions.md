<div align="center" dir="rtl">

# 💡 06 - التحديات والحلول

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Engineering+Problem+Solving;Real-World+Challenges;Innovative+Solutions" alt="Typing"/>

> **قصص هندسية عن مشاكل تمت مواجهتها وحلول إبداعية تم تنفيذها**

<br/>

[![Prev](https://img.shields.io/badge/%E2%86%90_%D8%A7%D9%84%D9%82%D8%B1%D8%A7%D8%B1%D8%A7%D8%AA-1eb8e4)](05-technical-decisions.md)
[![Next: Performance](https://img.shields.io/badge/Next_→_Performance-7565e3?style=for-the-badge)](07-performance.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

<div dir="rtl">

---

## 🔥 التحدي 1: تجميد الـ 14 ثانية

### 📋 المشكلة
تسبب أنيميشن المجرة في زمن حجب كلي (TBT) قدره 14 ثانية على الموبايل.

### 🔍 التحقيق
كشفت أداة Chrome DevTools Performance أن:
- 72% من TBT سببه دالة واحدة: `generateParticles()`
- 100k تكرار × حسابات الموقع + اللون
- كل ذلك على الـ Main Thread، مما يحجب تفاعل المستخدم

### ✅ الحل
**معمارية Web Worker**

```
Main Thread              Web Worker
     │                       │
     │  postMessage(params)  │
     ├──────────────────────►│
     │                       │ [حسابات ثقيلة]
     │  onmessage(data)      │
     │◄──────────────────────┤
     ▼                       ▼
```

### 📊 النتائج

| المقياس | قبل | بعد | التحسن |
|:------:|:------:|:-----:|:-----------:|
| TBT | 14,000ms | 2,500ms | **-82%** |
| إدراك المستخدم | متجمد | مستجيب | ✅ |

---

## 💻 التحدي 2: Monaco على الأجهزة الضعيفة

### 📋 المشكلة
استهلك محرر Monaco ذاكرة كبيرة جداً على الهواتف الاقتصادية (2GB RAM).

### 🔍 التحقيق
- يقوم Monaco بتحميل محرك المحرر بالكامل عند أول عرض
- تكامل TypeScript يضيف حملاً كبيراً
- لا توجد آلية تحميل كسول (Lazy loading) مدمجة

### ✅ الحل

1. **تحميل Monaco بشكل كسول فقط في صفحات الدروس**
```typescript
const Monaco = dynamic(() => import("@monaco-editor/react"), {
  ssr: false,
  loading: () => <EditorSkeleton />
});
```

2. **تعطيل الميزات غير الضرورية**
```typescript
options={{
  minimap: { enabled: false },
  lineNumbers: screenWidth > 768 ? "on" : "off",
  suggest: { showIcons: false }
}}
```

3. **تنظيف الذاكرة عند إلغاء التحميل (Unmount)**

### 📊 النتائج
- ✅ تقليل استهلاك الذاكرة بنسبة 40%
- ✅ تحميل أولي أسرع للصفحة
- ✅ يعمل على الأجهزة الاقتصادية

---

## 🔄 التحدي 3: محتوى ثنائي الاتجاه RTL/LTR

### 📋 المشكلة
الواجهة العربية (RTL) مع أمثلة كود إنجليزية (LTR) سببت مشاكل في العرض.

### 🔍 الأعراض
- مقتطفات الكود تظهر معكوسة
- المؤشر يقفز في المحرر
- أسماء المتغيرات تظهر فاسدة

### ✅ الحل

**نهج متعدد الطبقات:**

1. **عزل CSS**
```css
.code-block {
  direction: ltr !important;
  unicode-bidi: embed;
}
```

2. **إعداد المحرر**
```typescript
<Editor dir="ltr" options={{ rtlAlign: false }} />
```

3. **حدود المكونات**
```typescript
<div dir="rtl"> {/* محتوى عربي */}
  <div dir="ltr"> {/* قسم الكود */}
    <MonacoEditor />
  </div>
</div>
```

### 🎯 الدرس المستفاد
> المحتوى ثنائي الاتجاه يتطلب تحديداً صريحاً للاتجاه عند كل حد.

---

## 🔐 التحدي 4: تضارب مزامنة التقدم

### 📋 المشكلة
تحديثات localStorage و MongoDB المتزامنة تسببت في فقدان البيانات.

### 🔍 السيناريو
```
Tab 1: حفظ درس 3 ← localStorage
Tab 2: حفظ درس 5 ← localStorage (يكتب فوقه)
Cloud: يستقبل درس 5 فقط
```

### ✅ الحل

**مزامنة قائمة على الأحداث:**

```typescript
// On any save, dispatch event
localStorage.setItem(key, data);
window.dispatchEvent(new Event("local-storage-update"));

// Other tabs listen
window.addEventListener("storage", handleCrossTabSync);
```

**عمليات MongoDB الذرية:**
```typescript
await collection.updateOne(
  { userId },
  { $addToSet: { completedLessons: lessonId } } // لا يفقد البيانات أبداً
);
```

### 📊 النتائج
- ✅ صفر فقدان بيانات عبر التبويبات
- ✅ اتساق نهائي مع السحابة
- ✅ عمليات MongoDB ذرية (Atomic)

---

## 🎭 التحدي 5: عدم تطابق الـ Hydration

### 📋 المشكلة
المتوى المعروض من السيرفر لم يطابق حالة العميل، مما سبب أخطاء React.

### 🔍 السبب
حالة المصادقة متاحة فقط على العميل:
```
Server: لا جلسة ← يعرض hero "زائر"
Client: الجلسة موجودة ← يتوقع "أهلاً، [الاسم]"
Mismatch!
```

### ✅ الحل

**عرض ثنائي المرحلة:**
```typescript
function HeroSection() {
  const [variant, setVariant] = useState<HeroVariant>("guest");
  const [hydrated, setHydrated] = useState(false);

  useEffect(() => {
    setHydrated(true);
    // تحديد المتغير الفعلي بناءً على المصادقة
    determineVariant().then(setVariant);
  }, []);

  // Server and initial client render: consistent "guest"
  // After hydration: update to actual variant with transition
}
```

### 🎯 الدرس المستفاد
> اعرض دائماً افتراضياً آمناً للسيرفر، ثم حدث على العميل.

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 وثائق ذات صلة

<div align="center">

| التنقل |
|:----------:|
| [🔧 ← القرارات التقنية](05-technical-decisions.md) |
| [⚡ الأداء →](07-performance.md) |

</div>

<div align="center">

[![Prev: Technical Decisions](https://img.shields.io/badge/←_Prev:_Technical_Decisions-1eb8e4?style=for-the-badge)](05-technical-decisions.md) [![Next: Performance](https://img.shields.io/badge/Next:_Performance_→-7565e3?style=for-the-badge)](07-performance.md)

</div>
