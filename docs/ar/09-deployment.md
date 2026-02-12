<div align="center" dir="rtl">

# 🚀 09 - النشر و DevOps

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Infrastructure+Architecture;CI/CD+Pipeline;Serverless+Deployment" alt="Typing"/>

> **معمارية البنية التحتية، خط أنابيب النشر، والممارسات التشغيلية**

<br/>

[![Prev](https://img.shields.io/badge/%E2%86%90_%D8%A7%D9%84%D8%A7%D8%AE%D8%AA%D8%A8%D8%A7%D8%B1-1eb8e4)](08-testing-quality.md)
[![Next](https://img.shields.io/badge/%D8%A7%D9%84%D9%86%D8%AA%D8%A7%D8%A6%D8%AC_%E2%86%92-7565e3)](10-results-impact.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

<div dir="rtl">

---

## 🏗️ نظرة عامة على البنية التحتية

يستخدم AstroShan **معمارية Serverless** على Vercel:

```
┌──────────────────────────────────────────────────────────────┐
│   GitHub Repository                                          │
│        │ push/merge                                          │
│        ▼                                                     │
│   ┌─────────────┐                    ┌─────────────────┐    │
│   │   Vercel    │──── deploy ────────│  Vercel Edge    │    │
│   │   Build     │                    │  Functions      │    │
│   └─────────────┘                    └────────┬────────┘    │
│        │                                       │             │
│        ▼                                       ▼             │
│   ┌─────────────┐                    ┌─────────────────┐    │
│   │   Vercel    │                    │  MongoDB Atlas  │    │
│   │   Edge CDN  │                    │  Cloudinary CDN │    │
│   └─────────────┘                    └─────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

### الفوائد

| الميزة | الوصف |
|:-------:|:------------|
| 🔧 إدارة خادم صفرية | لا بنية تحتية للصيانة |
| 🌍 نشر Edge عالمي | المحتوى من أقرب منطقة |
| 📈 توسع تلقائي | التعامل مع طفرات الزيارات |
| 🔄 CI/CD مدمج | نشر تلقائي عند الـ git push |

---

## 🔧 خط أنابيب النشر (Deployment Pipeline)

### سير عمل Git

```
main branch ─────► Production (astroshan.vercel.app)
develop branch ─► Preview (develop-astroshan.vercel.app)
feature/* ──────► Preview ([branch]-astroshan.vercel.app)
```

### استراتيجية البيئات

| البيئة | الفرع | العنوان (URL) | الغرض |
|:-----------:|:------:|:---:|:--------|
| الإنتاج | `main` | astroshan.vercel.app | المستخدمون المباشرون |
| المعاينة | `develop` | develop-*.vercel.app | Staging |
| الميزة | `feature/*` | [branch]-*.vercel.app | اختبار الـ PR |

---

## 🔐 متغيرات البيئة (Environment Variables)

### الفئات

**عامة (NEXT_PUBLIC_*):**
```bash
NEXT_PUBLIC_APP_URL=https://astroshan.vercel.app
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your-cloud
```

**للخادم فقط (Server-only):**
```bash
MONGODB_URI=mongodb+srv://...
JWT_SECRET=...
GOOGLE_CLIENT_ID=...
```

### ممارسات الأمان

- ✅ الأسرار في Vercel Environment Variables
- ✅ أسرار مختلفة لكل بيئة
- ✅ لا أسرار في المستودع (Repo)
- ✅ تنقيح تلقائي في السجلات (Logs)

---

## 🛡️ ترويسات الأمان (Security Headers)

```typescript
const securityHeaders = [
  { key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains" },
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  { key: "Permissions-Policy", value: "camera=(), microphone=()" }
];
```

---

## 🗄️ إدارة قاعدة البيانات

### MongoDB Atlas

| الإعداد | القيمة |
|:-------:|:------|
| المستوى (Tier) | M0 (مجاني) |
| الإصدار | MongoDB 7.0 |
| النسخ الاحتياطي | يومي تلقائي |

### تجميع الاتصالات (Connection Pooling)

```typescript
// Serverless-safe singleton
if (process.env.NODE_ENV === 'development') {
  if (!global._mongoClientPromise) {
    global._mongoClientPromise = new MongoClient(uri).connect();
  }
  clientPromise = global._mongoClientPromise;
} else {
  clientPromise = new MongoClient(uri).connect();
}
```

---

## 🔄 إجراءات التراجع (Rollback Procedures)

### تراجع فوري (Instant Rollback)

1. اذهب لـ Vercel Dashboard → Deployments
2. جد آخر نشر يعمل
3. اضغط "..." → "Promote to Production"

**الفوائد:**
- ⚡ يستغرق ثواني (بدون إعادة بناء)
- 📦 الإصدار السابق لا يزال موجوداً
- 🔍 يمكن مقارنة الإصدارات جنباً إلى جنب

### الطوارئ

```bash
git revert HEAD
git push origin main
# نشر جديد يبدأ تلقائياً
```

---

## 📈 اعتبارات التوسع (Scaling Considerations)

| الموارد | الحد الأقصى | الحالي |
|:---------|:-----:|:-------:|
| دوال Serverless | 100 متزامن | منخفض |
| عرض النطاق (Bandwidth) | 100GB/شهر | منخفض |
| زمن البناء | 45 دقيقة | ~2 دقيقة |
| اتصالات MongoDB | 500 | منخفض |

---

## 🎓 دروس DevOps المستفادة

1. **Serverless يبسط العمليات** - لا إدارة خوادم
2. **نشرات المعاينة لا تقدر بثمن** - كل PR يحصل على رابط
3. **تكافؤ البيئات مهم** - نفس التكوين في كل مكان
4. **النشرات غير القابلة للتغيير تعطي الثقة** - تراجع فوري

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 وثائق ذات صلة

<div align="center">

| التنقل |
|:----------:|
| [🧪 ← الاختبار والجودة](08-testing-quality.md) |
| [📊 النتائج والأثر →](10-results-impact.md) |

</div>

<div align="center">

---

*التالي: [10 - النتائج والأثر](10-results-impact.md)*

</div>

</div>
