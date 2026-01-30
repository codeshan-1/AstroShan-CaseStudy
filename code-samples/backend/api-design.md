<div align="center">

# 🔌 API Design

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Serverless+API+Patterns;JWT+Authentication" alt="Typing"/>

> **Serverless API patterns used in AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🔐 1. Authentication API Routes

### Context

**Feature:** JWT-based authentication with OAuth  
**Complexity:** High  
**Technologies:** Next.js API Routes, Jose (JWT), Google OAuth 2.0

**The Challenge:**  
Implement secure authentication without a traditional backend server, using serverless functions with proper token management.

---

### The Problem

**Requirements:**
- Google OAuth integration
- Dual-token system (access + refresh)
- Stateless serverless functions
- Secure token storage and transmission

**Constraints:**
- No persistent server state
- Each function invocation is isolated
- Must prevent XSS and CSRF attacks

---

### API Structure

```
/api/auth/
├── google/
│   ├── signin/route.ts      → Initiate OAuth flow
│   └── callback/route.ts    → Handle OAuth callback
└── token/
    ├── refresh/route.ts     → Refresh access token
    └── revoke/route.ts      → Logout / invalidate
```

---

### Implementation (Simplified)

#### OAuth Signin Route

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - PKCE flow for security
 * - State parameter for CSRF protection
 * - Nonce for replay prevention
 */

// app/api/auth/google/signin/route.ts
import { NextResponse } from 'next/server';

const GOOGLE_AUTH_URL = 'https://accounts.google.com/o/oauth2/v2/auth';
const CLIENT_ID = process.env.GOOGLE_CLIENT_ID!;
const REDIRECT_URI = process.env.GOOGLE_REDIRECT_URI!;

export async function GET() {
  // Generate state for CSRF protection
  const state = crypto.randomUUID();
  
  const params = new URLSearchParams({
    client_id: CLIENT_ID,
    redirect_uri: REDIRECT_URI,
    response_type: 'code',
    scope: 'openid email profile',
    state,
    prompt: 'select_account',
  });

  const response = NextResponse.redirect(
    `${GOOGLE_AUTH_URL}?${params.toString()}`
  );
  
  // Store state in cookie for verification
  response.cookies.set('oauth_state', state, {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    maxAge: 600,  // 10 minutes
  });

  return response;
}
```

#### OAuth Callback Route

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes full error handling.
 */

// app/api/auth/google/callback/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { SignJWT } from 'jose';

const ACCESS_SECRET = new TextEncoder().encode(process.env.ACCESS_SECRET!);
const REFRESH_SECRET = new TextEncoder().encode(process.env.REFRESH_SECRET!);

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const code = searchParams.get('code');
  const state = searchParams.get('state');
  
  // Verify state matches cookie
  const storedState = request.cookies.get('oauth_state')?.value;
  if (state !== storedState) {
    return NextResponse.json(
      { error: 'Invalid state parameter' },
      { status: 400 }
    );
  }

  // Exchange code for tokens
  const googleTokens = await exchangeCodeForTokens(code!);
  
  // Get user info from Google
  const userInfo = await fetchGoogleUserInfo(googleTokens.access_token);
  
  // Create or fetch user in database
  const user = await upsertUser(userInfo);
  
  // Generate our own tokens
  const accessToken = await new SignJWT({ 
    userId: user.id,
    email: user.email,
  })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('15m')
    .sign(ACCESS_SECRET);

  const refreshToken = await new SignJWT({ 
    userId: user.id,
  })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('7d')
    .sign(REFRESH_SECRET);

  // Prepare response
  const response = NextResponse.redirect('/');
  
  // Set refresh token in HttpOnly cookie
  response.cookies.set('refresh_token', refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 60 * 60 * 24 * 7,  // 7 days
    path: '/',
  });
  
  // Clear OAuth state cookie
  response.cookies.delete('oauth_state');
  
  // Access token returned in response header
  // Frontend stores in memory (not localStorage)
  response.headers.set('X-Access-Token', accessToken);

  return response;
}

async function exchangeCodeForTokens(code: string) {
  const response = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      code,
      client_id: process.env.GOOGLE_CLIENT_ID!,
      client_secret: process.env.GOOGLE_CLIENT_SECRET!,
      redirect_uri: process.env.GOOGLE_REDIRECT_URI!,
      grant_type: 'authorization_code',
    }),
  });
  return response.json();
}
```

#### Token Refresh Route

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 */

// app/api/auth/token/refresh/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { jwtVerify, SignJWT } from 'jose';

export async function POST(request: NextRequest) {
  const refreshToken = request.cookies.get('refresh_token')?.value;
  
  if (!refreshToken) {
    return NextResponse.json(
      { error: 'No refresh token' },
      { status: 401 }
    );
  }

  try {
    // Verify refresh token
    const { payload } = await jwtVerify(
      refreshToken,
      REFRESH_SECRET
    );
    
    // Generate new access token
    const accessToken = await new SignJWT({ 
      userId: payload.userId,
    })
      .setProtectedHeader({ alg: 'HS256' })
      .setExpirationTime('15m')
      .sign(ACCESS_SECRET);

    return NextResponse.json({ accessToken });
    
  } catch (error) {
    return NextResponse.json(
      { error: 'Invalid refresh token' },
      { status: 401 }
    );
  }
}
```

#### Token Revoke Route (Logout)

```typescript
// app/api/auth/token/revoke/route.ts
import { NextResponse } from 'next/server';

export async function POST() {
  const response = NextResponse.json({ success: true });
  
  // Clear refresh token cookie
  response.cookies.set('refresh_token', '', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 0,  // Expire immediately
    path: '/',
  });

  return response;
}
```

---

### Key Techniques Used

#### 1. HttpOnly Cookies for Refresh Tokens

```typescript
response.cookies.set('refresh_token', token, {
  httpOnly: true,  // JavaScript can't access
  secure: true,    // HTTPS only
  sameSite: 'strict',  // CSRF protection
});
```

*Why it matters:* XSS attacks can't steal refresh tokens.

#### 2. Short-Lived Access Tokens

```typescript
.setExpirationTime('15m')
```

*Why it matters:* Limits damage if access token is stolen.

#### 3. State Parameter for CSRF

```typescript
const state = crypto.randomUUID();
// ... store in cookie, verify on callback
```

*Why it matters:* Prevents OAuth CSRF attacks.

---

## 📊 2. Progress API Routes

### Context

**Feature:** User progress persistence  
**Complexity:** Medium  
**Technologies:** Next.js API Routes, MongoDB

**The Challenge:**  
Handle progress CRUD with atomic operations and proper authorization.

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes validation and error handling.
 */

// app/api/progress/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { jwtVerify } from 'jose';
import { getCollection } from '@/lib/db/mongodb';

// GET: Fetch user progress
export async function GET(request: NextRequest) {
  const userId = await validateAuth(request);
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const collection = await getCollection('user_progress');
  const progress = await collection.findOne({ userId });

  return NextResponse.json(progress ?? { html: {}, internet: {} });
}

// POST: Update progress
export async function POST(request: NextRequest) {
  const userId = await validateAuth(request);
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const body = await request.json();
  const { module, completedLessons, quizScores } = body;

  const collection = await getCollection('user_progress');
  
  // Atomic update - add to set, don't replace
  await collection.updateOne(
    { userId },
    {
      $addToSet: {
        [`${module}.completedLessons`]: { $each: completedLessons ?? [] },
      },
      $max: Object.entries(quizScores ?? {}).reduce((acc, [key, value]) => ({
        ...acc,
        [`${module}.quizScores.${key}`]: value,
      }), {}),
      $set: {
        updatedAt: new Date(),
      },
    },
    { upsert: true }
  );

  return NextResponse.json({ success: true });
}

async function validateAuth(request: NextRequest): Promise<string | null> {
  const authHeader = request.headers.get('authorization');
  if (!authHeader?.startsWith('Bearer ')) {
    return null;
  }

  const token = authHeader.slice(7);
  
  try {
    const { payload } = await jwtVerify(token, ACCESS_SECRET);
    return payload.userId as string;
  } catch {
    return null;
  }
}
```

---

### Key Techniques Used

#### 1. Atomic Updates with $addToSet

```typescript
await collection.updateOne(
  { userId },
  {
    $addToSet: {
      'html.completedLessons': { $each: newLessons }
    }
  }
);
```

*Why it matters:* No race conditions, lessons never accidentally removed.

#### 2. Bearer Token Authentication

```typescript
const authHeader = request.headers.get('authorization');
const token = authHeader.slice(7);  // Remove "Bearer "
```

*Why it matters:* Standard pattern for API authentication.

---

## 🖼️ 3. Profile API Routes

### Context

**Feature:** User profile management with avatar upload  
**Complexity:** Medium  
**Technologies:** Next.js, Cloudinary

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 */

// app/api/profile/route.ts
export async function GET(request: NextRequest) {
  const userId = await validateAuth(request);
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const collection = await getCollection('users');
  const user = await collection.findOne(
    { _id: userId },
    { projection: { displayName: 1, email: 1, avatarUrl: 1 } }
  );

  return NextResponse.json(user);
}

// app/api/profile/avatar/route.ts
export async function POST(request: NextRequest) {
  const userId = await validateAuth(request);
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const formData = await request.formData();
  const file = formData.get('avatar') as File;
  
  if (!file) {
    return NextResponse.json({ error: 'No file provided' }, { status: 400 });
  }

  // Upload to Cloudinary
  const arrayBuffer = await file.arrayBuffer();
  const base64 = Buffer.from(arrayBuffer).toString('base64');
  const dataUri = `data:${file.type};base64,${base64}`;

  const uploadResult = await cloudinary.uploader.upload(dataUri, {
    folder: 'avatars',
    public_id: userId,
    overwrite: true,
    transformation: [
      { width: 200, height: 200, crop: 'fill' },
      { quality: 'auto', fetch_format: 'auto' },
    ],
  });

  // Update user record
  const collection = await getCollection('users');
  await collection.updateOne(
    { _id: userId },
    { $set: { avatarUrl: uploadResult.secure_url } }
  );

  return NextResponse.json({ avatarUrl: uploadResult.secure_url });
}
```

---

## 🛡️ Error Handling Pattern

### Consistent Error Responses

```typescript
/**
 * Standard error response format
 */

interface APIError {
  error: string;
  code?: string;
  details?: Record<string, unknown>;
}

function createErrorResponse(
  message: string,
  status: number,
  code?: string
): NextResponse<APIError> {
  return NextResponse.json(
    { error: message, code },
    { status }
  );
}

// Usage
if (!userId) {
  return createErrorResponse('Authentication required', 401, 'AUTH_REQUIRED');
}

if (!isValidInput(body)) {
  return createErrorResponse('Invalid input', 400, 'VALIDATION_ERROR');
}
```

---

## 📋 API Summary

| Endpoint | Method | Purpose | Auth Required |
|----------|--------|---------|---------------|
| `/api/auth/google/signin` | GET | Start OAuth | No |
| `/api/auth/google/callback` | GET | OAuth callback | No |
| `/api/auth/token/refresh` | POST | Refresh access token | Cookie |
| `/api/auth/token/revoke` | POST | Logout | Cookie |
| `/api/progress` | GET | Fetch progress | Bearer |
| `/api/progress` | POST | Update progress | Bearer |
| `/api/profile` | GET | Fetch profile | Bearer |
| `/api/profile/avatar` | POST | Upload avatar | Bearer |

---

## 💡 Lessons Learned

### What Worked Well

- **Dual-token system** provides security without UX friction
- **Atomic MongoDB operations** prevent race conditions
- **Serverless functions** scale automatically

### What I'd Do Differently

- Add rate limiting from the start
- Implement request logging for debugging
- Add API versioning

### Key Takeaways

1. HttpOnly cookies are essential for refresh tokens
2. Short access token lifetimes limit blast radius
3. Atomic database operations prevent data corruption

---

## 🔗 Related Patterns

- [State Management](../frontend/state-management.md) - How frontend consumes APIs
- [Middleware Pattern](./middleware-pattern.md) - Request/response transformation
- [Database Optimization](./database-optimization.md) - MongoDB patterns

---

## 📚 Further Reading

- [Next.js Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)
- [Jose JWT Library](https://github.com/panva/jose)
- [OAuth 2.0 Best Practices](https://oauth.net/2/)
- [Cloudinary Upload API](https://cloudinary.com/documentation/image_upload_api_reference)
