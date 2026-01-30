<div align="center">

# 🛡️ Middleware Pattern

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=Request+Response+Handling;Auth+Guards+%26+Validation" alt="Typing"/>

> **Request/response handling patterns used in AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🔒 1. Authentication Middleware

### Context

**Feature:** Protect API routes requiring authentication  
**Complexity:** Medium  
**Technologies:** Next.js Middleware, JWT (Jose)

**The Challenge:**  
Create reusable authentication logic that can protect routes while supporting token refresh.

---

### The Problem

**Requirements:**
- Verify JWT on protected routes
- Support automatic token refresh on expiry
- Handle anonymous vs authenticated access
- Don't block public routes

**Constraints:**
- Next.js Middleware runs at the edge
- Limited runtime (no Node.js APIs)
- Must be fast (runs on every request)

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - More nuanced route matching
 * - Token refresh logic
 * - Error logging
 */

// middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { jwtVerify } from 'jose';

// Routes that require authentication
const PROTECTED_ROUTES = [
  '/api/progress',
  '/api/profile',
  '/profile',
];

// Routes that should bypass middleware
const PUBLIC_ROUTES = [
  '/api/auth',
  '/_next',
  '/favicon.ico',
];

const ACCESS_SECRET = new TextEncoder().encode(process.env.ACCESS_SECRET!);
const REFRESH_SECRET = new TextEncoder().encode(process.env.REFRESH_SECRET!);

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  // Skip public routes
  if (PUBLIC_ROUTES.some(route => pathname.startsWith(route))) {
    return NextResponse.next();
  }
  
  // Check if route is protected
  const isProtected = PROTECTED_ROUTES.some(route => 
    pathname.startsWith(route)
  );
  
  if (!isProtected) {
    return NextResponse.next();
  }

  // API routes use Bearer token
  if (pathname.startsWith('/api/')) {
    return handleAPIAuth(request);
  }
  
  // Page routes check for valid session
  return handlePageAuth(request);
}

async function handleAPIAuth(request: NextRequest): Promise<NextResponse> {
  const authHeader = request.headers.get('authorization');
  
  if (!authHeader?.startsWith('Bearer ')) {
    return NextResponse.json(
      { error: 'No authorization header' },
      { status: 401 }
    );
  }

  const token = authHeader.slice(7);
  
  try {
    await jwtVerify(token, ACCESS_SECRET);
    return NextResponse.next();
  } catch (error) {
    // Token expired or invalid
    return NextResponse.json(
      { error: 'Invalid or expired token' },
      { status: 401 }
    );
  }
}

async function handlePageAuth(request: NextRequest): Promise<NextResponse> {
  const refreshToken = request.cookies.get('refresh_token')?.value;
  
  if (!refreshToken) {
    // Redirect to login
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('redirect', request.nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }

  try {
    // Verify refresh token (indicates valid session)
    await jwtVerify(refreshToken, REFRESH_SECRET);
    return NextResponse.next();
  } catch (error) {
    // Invalid session, redirect to login
    const response = NextResponse.redirect(new URL('/login', request.url));
    response.cookies.delete('refresh_token');
    return response;
  }
}

export const config = {
  matcher: [
    /*
     * Match all paths except:
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */
    '/((?!_next/static|_next/image|favicon.ico).*)',
  ],
};
```

---

### Key Techniques Used

#### 1. Route Pattern Matching

```typescript
const isProtected = PROTECTED_ROUTES.some(route => 
  pathname.startsWith(route)
);
```

*Why it matters:* Flexible protection without hardcoding every route.

#### 2. Different Auth Strategies by Route Type

```typescript
if (pathname.startsWith('/api/')) {
  return handleAPIAuth(request);  // Bearer token
}
return handlePageAuth(request);   // Cookie-based
```

*Why it matters:* APIs use Bearer tokens, pages use cookie sessions.

#### 3. Matcher Configuration

```typescript
export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

*Why it matters:* Skips static assets, reducing middleware overhead.

---

## 🔄 2. Token Refresh Middleware

### Context

**Feature:** Automatic access token refresh  
**Complexity:** High  
**Technologies:** Next.js Middleware, JWT

**The Challenge:**  
Refresh expired access tokens transparently without breaking the request flow.

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * This approach extends the basic auth middleware.
 */

async function handleAPIAuthWithRefresh(
  request: NextRequest
): Promise<NextResponse> {
  const authHeader = request.headers.get('authorization');
  
  if (!authHeader?.startsWith('Bearer ')) {
    // No auth header - try refresh token
    return attemptTokenRefresh(request);
  }

  const token = authHeader.slice(7);
  
  try {
    await jwtVerify(token, ACCESS_SECRET);
    return NextResponse.next();
  } catch (error) {
    // Access token expired - attempt refresh
    return attemptTokenRefresh(request);
  }
}

async function attemptTokenRefresh(
  request: NextRequest
): Promise<NextResponse> {
  const refreshToken = request.cookies.get('refresh_token')?.value;
  
  if (!refreshToken) {
    return NextResponse.json(
      { error: 'No refresh token' },
      { status: 401 }
    );
  }

  try {
    // Verify refresh token
    const { payload } = await jwtVerify(refreshToken, REFRESH_SECRET);
    
    // Generate new access token
    const newAccessToken = await new SignJWT({ 
      userId: payload.userId 
    })
      .setProtectedHeader({ alg: 'HS256' })
      .setExpirationTime('15m')
      .sign(ACCESS_SECRET);

    // Clone the request with new token
    const newHeaders = new Headers(request.headers);
    newHeaders.set('Authorization', `Bearer ${newAccessToken}`);
    
    const response = NextResponse.next({
      request: {
        headers: newHeaders,
      },
    });
    
    // Include new token in response for client to store
    response.headers.set('X-New-Access-Token', newAccessToken);
    
    return response;
    
  } catch (error) {
    return NextResponse.json(
      { error: 'Refresh token expired' },
      { status: 401 }
    );
  }
}
```

---

## 🛡️ 3. Security Headers Middleware

### Context

**Feature:** Add security headers to all responses  
**Complexity:** Low  
**Technologies:** Next.js Middleware

---

### Implementation

```typescript
/**
 * Security headers applied via next.config.ts
 * but can also be done in middleware for more control.
 */

// In middleware or next.config.ts
const securityHeaders = [
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload',
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff',
  },
  {
    key: 'X-Frame-Options',
    value: 'DENY',
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin',
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()',
  },
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on',
  },
];

// In middleware
function addSecurityHeaders(response: NextResponse): NextResponse {
  securityHeaders.forEach(({ key, value }) => {
    response.headers.set(key, value);
  });
  return response;
}
```

---

## 📝 4. Request Validation Pattern

### Context

**Feature:** Validate API request bodies  
**Complexity:** Medium  
**Technologies:** TypeScript, Zod (optional)

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified validation approach.
 * Production would use Zod or similar.
 */

// Validation utility
type ValidationResult<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

function validateProgressUpdate(body: unknown): ValidationResult<{
  module: string;
  completedLessons: number[];
}> {
  if (!body || typeof body !== 'object') {
    return { success: false, error: 'Request body must be an object' };
  }

  const { module, completedLessons } = body as Record<string, unknown>;

  if (typeof module !== 'string' || !['html', 'internet'].includes(module)) {
    return { success: false, error: 'Invalid module' };
  }

  if (!Array.isArray(completedLessons) || 
      !completedLessons.every(n => typeof n === 'number')) {
    return { success: false, error: 'completedLessons must be an array of numbers' };
  }

  return { 
    success: true, 
    data: { module, completedLessons: completedLessons as number[] } 
  };
}

// Usage in route handler
export async function POST(request: NextRequest) {
  const body = await request.json();
  
  const validation = validateProgressUpdate(body);
  if (!validation.success) {
    return NextResponse.json(
      { error: validation.error },
      { status: 400 }
    );
  }
  
  const { module, completedLessons } = validation.data;
  // ... proceed with validated data
}
```

### With Zod (Recommended Pattern)

```typescript
import { z } from 'zod';

const ProgressUpdateSchema = z.object({
  module: z.enum(['html', 'internet']),
  completedLessons: z.array(z.number().int().positive()),
  quizScores: z.record(z.number().int().min(0).max(100)).optional(),
});

type ProgressUpdate = z.infer<typeof ProgressUpdateSchema>;

export async function POST(request: NextRequest) {
  const body = await request.json();
  
  const result = ProgressUpdateSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json(
      { error: 'Validation failed', details: result.error.format() },
      { status: 400 }
    );
  }
  
  const { module, completedLessons, quizScores } = result.data;
  // ... proceed with type-safe data
}
```

---

## 🔀 5. Request/Response Transformation

### Context

**Feature:** Normalize API responses  
**Complexity:** Low  
**Technologies:** TypeScript, next.js

---

### Implementation

```typescript
/**
 * Standard API response wrapper
 */

interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    message: string;
    code?: string;
  };
  meta?: {
    timestamp: string;
    requestId: string;
  };
}

function createSuccessResponse<T>(
  data: T,
  status = 200
): NextResponse<APIResponse<T>> {
  return NextResponse.json({
    success: true,
    data,
    meta: {
      timestamp: new Date().toISOString(),
      requestId: crypto.randomUUID(),
    },
  }, { status });
}

function createErrorResponse(
  message: string,
  status: number,
  code?: string
): NextResponse<APIResponse<never>> {
  return NextResponse.json({
    success: false,
    error: { message, code },
    meta: {
      timestamp: new Date().toISOString(),
      requestId: crypto.randomUUID(),
    },
  }, { status });
}

// Usage
export async function GET(request: NextRequest) {
  try {
    const data = await fetchData();
    return createSuccessResponse(data);
  } catch (error) {
    return createErrorResponse(
      'Failed to fetch data',
      500,
      'FETCH_ERROR'
    );
  }
}
```

---

## 💡 Lessons Learned

### What Worked Well

- **Centralized middleware** reduces code duplication
- **Pattern matching** provides flexible route protection
- **Consistent response format** simplifies client handling

### What I'd Do Differently

- Add request/response logging from the start
- Implement rate limiting in middleware
- Add correlation IDs for request tracing

### Key Takeaways

1. Middleware is powerful but should be lightweight
2. Consistent error responses improve developer experience
3. Input validation prevents entire categories of bugs

---

## 🔗 Related Patterns

- [API Design](./api-design.md) - Route handlers that use middleware
- [State Management](../frontend/state-management.md) - Client-side token handling
- [Design Patterns](../patterns/design-patterns.md) - Decorator pattern in middleware

---

## 📚 Further Reading

- [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)
- [Zod Validation Library](https://zod.dev)
- [OWASP Security Headers](https://owasp.org/www-project-secure-headers/)
- [JWT Best Practices](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims)
