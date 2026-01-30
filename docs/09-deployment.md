<div align="center">

# 🚀 09 - Deployment & DevOps

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Infrastructure+Architecture;CI/CD+Pipeline;Serverless+Deployment" alt="Typing"/>

> **Infrastructure architecture, deployment pipeline, and operational practices**

<br/>

[![Prev](https://img.shields.io/badge/←_Testing-1eb8e4?style=for-the-badge)](08-testing-quality.md)
[![Next](https://img.shields.io/badge/Results_→-7565e3?style=for-the-badge)](10-results-impact.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🏗️ Infrastructure Overview

AstroShan uses **serverless architecture** on Vercel:

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

### Benefits

| Feature | Description |
|:-------:|:------------|
| 🔧 Zero server management | No infrastructure to maintain |
| 🌍 Global edge deployment | Content from closest region |
| 📈 Automatic scaling | Handle traffic spikes |
| 🔄 Built-in CI/CD | Auto deploy on git push |

---

## 🔧 Deployment Pipeline

### Git Workflow

```
main branch ─────► Production (astroshan.vercel.app)
develop branch ─► Preview (develop-astroshan.vercel.app)
feature/* ──────► Preview ([branch]-astroshan.vercel.app)
```

### Environment Strategy

| Environment | Branch | URL | Purpose |
|:-----------:|:------:|:---:|:--------|
| Production | `main` | astroshan.vercel.app | Live users |
| Preview | `develop` | develop-*.vercel.app | Staging |
| Feature | `feature/*` | [branch]-*.vercel.app | PR testing |

---

## 🔐 Environment Variables

### Categories

**Public (NEXT_PUBLIC_*):**
```bash
NEXT_PUBLIC_APP_URL=https://astroshan.vercel.app
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your-cloud
```

**Server-only:**
```bash
MONGODB_URI=mongodb+srv://...
JWT_SECRET=...
GOOGLE_CLIENT_ID=...
```

### Security Practices

- ✅ Secrets in Vercel Environment Variables
- ✅ Different secrets per environment
- ✅ No secrets in repository
- ✅ Automatic redaction in logs

---

## 🛡️ Security Headers

```typescript
const securityHeaders = [
  { key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains" },
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  { key: "Permissions-Policy", value: "camera=(), microphone=()" }
];
```

---

## 🗄️ Database Management

### MongoDB Atlas

| Setting | Value |
|:-------:|:------|
| Tier | M0 (Free) |
| Version | MongoDB 7.0 |
| Backups | Daily automatic |

### Connection Pooling

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

## 🔄 Rollback Procedures

### Instant Rollback

1. Go to Vercel Dashboard → Deployments
2. Find last working deployment
3. Click "..." → "Promote to Production"

**Benefits:**
- ⚡ Takes seconds (no rebuild)
- 📦 Previous version still exists
- 🔍 Can compare versions side-by-side

### Emergency

```bash
git revert HEAD
git push origin main
# New deployment triggered automatically
```

---

## 📈 Scaling Considerations

| Resource | Limit | Current |
|:---------|:-----:|:-------:|
| Serverless Functions | 100 concurrent | Low |
| Bandwidth | 100GB/month | Low |
| Build time | 45 min | ~2 min |
| MongoDB connections | 500 | Low |

---

## 🎓 DevOps Learnings

1. **Serverless simplifies operations** - No server management
2. **Preview deployments are invaluable** - Every PR gets a URL
3. **Environment parity matters** - Same config everywhere
4. **Immutable deployments enable confidence** - Instant rollback

---

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

## 🔗 Related Documents

| Navigation |
|:----------:|
| [🧪 ← Testing & Quality](08-testing-quality.md) |
| [📊 Results & Impact →](10-results-impact.md) |

---

*Next: [10 - Results & Impact](10-results-impact.md)*
