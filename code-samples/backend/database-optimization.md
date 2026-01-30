<div align="center">

# 🍃 Database Optimization

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=1EB8E4&center=true&vCenter=true&width=500&lines=MongoDB+Patterns;Connection+Pooling+%26+Atomic+Ops" alt="Typing"/>

> **MongoDB patterns used in AstroShan**

[![Back](https://img.shields.io/badge/⬅️_Back-1eb8e4?style=for-the-badge)](../README.md)

</div>

<div align="center">
<img width="600" src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png"/>
</div>

---

## 🔗 1. Connection Pooling (Serverless-Safe)

### Context

**Feature:** Efficient database connections in serverless environment  
**Complexity:** Medium  
**Technologies:** MongoDB, Next.js Serverless

**The Challenge:**  
Serverless functions are stateless and can run concurrently. Naive database connections would create connection exhaustion.

---

### The Problem

**Issues with naive approach:**
- Each function invocation creates new connection
- Connection pool exhaustion under load
- Cold starts become slower with repeated connections

**Requirements:**
- Reuse connections across function invocations
- Handle development mode HMR properly
- Minimal connection overhead

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 * Actual implementation includes:
 * - Connection monitoring
 * - Retry logic
 * - Error handling
 */

// lib/db/mongodb.ts
import { MongoClient, Db } from 'mongodb';

const uri = process.env.MONGODB_URI!;
const dbName = process.env.MONGODB_DB!;

const options = {
  maxPoolSize: 10,           // Limit connections
  minPoolSize: 2,            // Keep some warm
  maxIdleTimeMS: 60000,      // Close idle connections after 1 min
  serverSelectionTimeoutMS: 5000,
};

let clientPromise: Promise<MongoClient>;

// Serverless-safe singleton pattern
if (process.env.NODE_ENV === 'development') {
  // In development, use global variable to survive HMR
  const globalWithMongo = global as typeof globalThis & {
    _mongoClientPromise?: Promise<MongoClient>;
  };
  
  if (!globalWithMongo._mongoClientPromise) {
    const client = new MongoClient(uri, options);
    globalWithMongo._mongoClientPromise = client.connect();
    console.log('[MongoDB] Creating new development connection');
  }
  
  clientPromise = globalWithMongo._mongoClientPromise;
  
} else {
  // In production, use module-scoped variable
  const client = new MongoClient(uri, options);
  clientPromise = client.connect();
}

export async function getDatabase(): Promise<Db> {
  const client = await clientPromise;
  return client.db(dbName);
}

export async function getCollection<T extends Document>(
  collectionName: string
) {
  const db = await getDatabase();
  return db.collection<T>(collectionName);
}
```

### Usage

```typescript
// In any API route
import { getCollection } from '@/lib/db/mongodb';

export async function GET() {
  const collection = await getCollection('user_progress');
  const data = await collection.findOne({ userId: '...' });
  return NextResponse.json(data);
}
```

---

### Key Techniques Used

#### 1. Global Variable in Development

```typescript
const globalWithMongo = global as typeof globalThis & {
  _mongoClientPromise?: Promise<MongoClient>;
};
```

*Why it matters:* Survives Hot Module Replacement, prevents connection leaks.

#### 2. Promise Caching

```typescript
clientPromise = client.connect();
// Later...
const client = await clientPromise;
```

*Why it matters:* Multiple concurrent functions await the same connection.

#### 3. Connection Pool Limits

```typescript
{
  maxPoolSize: 10,
  minPoolSize: 2,
}
```

*Why it matters:* Prevents connection exhaustion while keeping some connections warm.

---

## ⚡ 2. Atomic Operations for Progress

### Context

**Feature:** Race-condition-free progress updates  
**Complexity:** Medium  
**Technologies:** MongoDB Update Operators

**The Challenge:**  
Multiple tabs or devices might update progress simultaneously. Read-modify-write cycles cause data loss.

---

### The Problem

**Scenario:**
1. Tab A reads: `completedLessons: [1, 2]`
2. Tab B reads: `completedLessons: [1, 2]`
3. Tab A completes lesson 3, writes: `[1, 2, 3]`
4. Tab B completes lesson 4, writes: `[1, 2, 4]`
5. **Result:** Lesson 3 is lost!

**Requirements:**
- No data loss from concurrent updates
- Arrays should union, not replace
- Scores should use maximum
- Timestamps should use latest

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified for demonstration.
 */

async function updateProgress(
  userId: string,
  module: string,
  newLessons: number[],
  newQuizScores: Record<string, number>
): Promise<void> {
  const collection = await getCollection('user_progress');
  
  // Build atomic update
  const updateDoc: Record<string, unknown> = {
    // $addToSet adds to array only if not present
    $addToSet: {
      [`${module}.completedLessons`]: { $each: newLessons },
    },
    // Set last updated
    $set: {
      updatedAt: new Date(),
    },
  };
  
  // $max keeps the highest value for each quiz
  if (Object.keys(newQuizScores).length > 0) {
    const maxUpdates: Record<string, number> = {};
    for (const [quizId, score] of Object.entries(newQuizScores)) {
      maxUpdates[`${module}.quizScores.${quizId}`] = score;
    }
    updateDoc.$max = maxUpdates;
  }
  
  await collection.updateOne(
    { userId },
    updateDoc,
    { upsert: true }  // Create if doesn't exist
  );
}
```

### Atomic Operators Explained

```typescript
// $addToSet: Add to array only if not present
{ $addToSet: { lessons: 3 } }
// [1, 2] → [1, 2, 3]
// [1, 2, 3] → [1, 2, 3] (no duplicate)

// $each: Use with $addToSet for multiple items
{ $addToSet: { lessons: { $each: [3, 4, 5] } } }

// $max: Only update if new value is greater
{ $max: { "quizScores.1": 85 } }
// 80 → 85 ✓
// 90 → 90 (unchanged)

// $min: Only update if new value is smaller
{ $min: { "bestTime": 120 } }

// $inc: Increment by value
{ $inc: { "attempts": 1 } }
```

---

### Key Techniques Used

#### 1. $addToSet for Array Union

```typescript
$addToSet: {
  'html.completedLessons': { $each: [3, 4] }
}
```

*Why it matters:* Concurrent updates both add their items without overwriting.

#### 2. $max for Score Preservation

```typescript
$max: {
  'html.quizScores.5': 92
}
```

*Why it matters:* Student never loses their best score.

#### 3. Upsert Pattern

```typescript
{ upsert: true }
```

*Why it matters:* Creates document if missing, updates if exists.

---

## 📊 3. Indexing Strategy

### Context

**Feature:** Fast queries on large collections  
**Complexity:** Low  
**Technologies:** MongoDB Indexes

---

### Implementation

```typescript
/**
 * Index creation for user_progress collection
 */

async function createIndexes() {
  const collection = await getCollection('user_progress');
  
  // Primary lookup index
  await collection.createIndex(
    { userId: 1 },
    { unique: true }  // One document per user
  );
  
  // Time-based queries for analytics
  await collection.createIndex(
    { updatedAt: -1 }  // -1 for descending
  );
  
  // Compound index for module-specific queries
  await collection.createIndex({
    userId: 1,
    'html.completedLessons': 1,
  });
}
```

### Index Considerations

| Query Pattern | Index Needed |
|--------------|--------------|
| Find by userId | `{ userId: 1 }` ✅ |
| Recent updates | `{ updatedAt: -1 }` |
| Module progress | Compound index |
| Full collection scan | ❌ Avoid |

---

## 🔍 4. Projection for Minimal Data Transfer

### Context

**Feature:** Reduce network bandwidth and memory usage  
**Complexity:** Low  
**Technologies:** MongoDB Projection

---

### Implementation

```typescript
/**
 * Only fetch needed fields
 */

// ❌ Bad: Fetches entire document
const user = await collection.findOne({ userId });

// ✅ Good: Fetches only needed fields
const user = await collection.findOne(
  { userId },
  {
    projection: {
      displayName: 1,
      email: 1,
      avatarUrl: 1,
      _id: 0,  // Exclude _id if not needed
    },
  }
);

// Exclude large fields
const progressSummary = await collection.findOne(
  { userId },
  {
    projection: {
      'html.completedLessons': 1,
      'html.quizScores': 0,  // Exclude detailed scores
      'internet.completedLessons': 1,
    },
  }
);
```

---

## 📈 5. Aggregation for Analytics

### Context

**Feature:** Compute statistics across users  
**Complexity:** Medium  
**Technologies:** MongoDB Aggregation Pipeline

---

### Implementation (Simplified)

```typescript
/**
 * DISCLAIMER: Simplified analytics queries.
 */

// Get completion statistics by module
async function getModuleStats(): Promise<{
  module: string;
  totalStudents: number;
  avgCompletion: number;
}[]> {
  const collection = await getCollection('user_progress');
  
  const stats = await collection.aggregate([
    // Only users with progress
    {
      $match: {
        'html.completedLessons': { $exists: true, $ne: [] },
      },
    },
    // Calculate completion percentage
    {
      $project: {
        htmlCompletion: {
          $multiply: [
            { $divide: [{ $size: '$html.completedLessons' }, 57] },
            100,
          ],
        },
      },
    },
    // Group for statistics
    {
      $group: {
        _id: null,
        totalStudents: { $sum: 1 },
        avgCompletion: { $avg: '$htmlCompletion' },
      },
    },
  ]).toArray();

  return stats;
}

// Get leaderboard
async function getLeaderboard(limit = 10) {
  const collection = await getCollection('user_progress');
  
  return collection.aggregate([
    {
      $project: {
        userId: 1,
        displayName: 1,
        completedCount: { $size: { $ifNull: ['$html.completedLessons', []] } },
      },
    },
    { $sort: { completedCount: -1 } },
    { $limit: limit },
  ]).toArray();
}
```

---

## 🛡️ 6. Error Handling & Retry Logic

### Context

**Feature:** Resilient database operations  
**Complexity:** Medium  
**Technologies:** TypeScript, MongoDB

---

### Implementation

```typescript
/**
 * Database operation with retry
 */

interface RetryOptions {
  maxAttempts?: number;
  delayMs?: number;
  backoff?: boolean;
}

async function withRetry<T>(
  operation: () => Promise<T>,
  options: RetryOptions = {}
): Promise<T> {
  const { 
    maxAttempts = 3, 
    delayMs = 100, 
    backoff = true 
  } = options;
  
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error as Error;
      
      // Don't retry on validation errors
      if (isValidationError(error)) {
        throw error;
      }
      
      // Log retry attempt
      console.warn(
        `[MongoDB] Attempt ${attempt}/${maxAttempts} failed:`,
        error instanceof Error ? error.message : error
      );
      
      // Wait before retry (with exponential backoff if enabled)
      if (attempt < maxAttempts) {
        const delay = backoff 
          ? delayMs * Math.pow(2, attempt - 1) 
          : delayMs;
        await sleep(delay);
      }
    }
  }
  
  throw lastError!;
}

// Usage
const progress = await withRetry(
  () => collection.findOne({ userId }),
  { maxAttempts: 3, backoff: true }
);

function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}

function isValidationError(error: unknown): boolean {
  return error instanceof Error && 
    error.message.includes('validation');
}
```

---

## 💡 Lessons Learned

### What Worked Well

- **Connection pooling** eliminates cold start overhead
- **Atomic operators** prevent race conditions completely
- **Projections** dramatically reduce response times

### What I'd Do Differently

- Add query performance monitoring from the start
- Implement connection health checks
- Add circuit breaker for database failures

### Key Takeaways

1. Serverless needs special connection handling
2. Atomic operations > read-modify-write
3. Indexes are critical for performance at scale

---

## 🔗 Related Patterns

- [API Design](./api-design.md) - Routes that use these patterns
- [State Management](../frontend/state-management.md) - How frontend syncs with DB
- [Design Patterns](../patterns/design-patterns.md) - Singleton for connections

---

## 📚 Further Reading

- [MongoDB Connection Pooling](https://www.mongodb.com/docs/drivers/node/current/fundamentals/connection/connection-options/)
- [MongoDB Update Operators](https://www.mongodb.com/docs/manual/reference/operator/update/)
- [MongoDB Indexes](https://www.mongodb.com/docs/manual/indexes/)
- [Serverless Database Connections](https://www.mongodb.com/docs/atlas/manage-connections-aws-lambda/)
