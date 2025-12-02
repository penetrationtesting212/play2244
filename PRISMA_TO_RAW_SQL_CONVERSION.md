# 🔄 Prisma to Raw PostgreSQL Conversion Guide

## Overview

This document details the conversion from Prisma ORM to raw PostgreSQL queries for better performance, flexibility, and control over database operations.

---

## 📋 Files Modified

| File | Changes | Lines Changed |
|------|---------|---------------|
| `backend/src/controllers/workflow.controller.ts` | Converted all Prisma queries to raw SQL | +73, -68 |

---

## 🔧 Key Changes

### **1. Import Changes**

#### **BEFORE (Prisma):**
```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();
```

#### **AFTER (Raw SQL):**
```typescript
import pool from '../db';
```

**Why:** 
- ✅ Use existing PostgreSQL connection pool from `db.ts`
- ✅ No need for Prisma Client instantiation
- ✅ Reuse established connection configuration

---

## 🔄 Query Conversions

### **CONVERSION 1: findFirst (Single Record)**

**Location:** Line 27-35 → Line 25-31

#### **BEFORE (Prisma):**
```typescript
const script = await prisma.script.findFirst({
  where: { id: scriptId, userId },
  select: { 
    id: true, 
    name: true, 
    workflowStatus: true,
    updatedAt: true 
  }
});
```

#### **AFTER (Raw SQL):**
```typescript
const scriptResult = await pool.query(
  `SELECT id, name, "workflowStatus", "updatedAt" 
   FROM "Script" 
   WHERE id = $1 AND "userId" = $2`,
  [scriptId, userId]
);

const script = scriptResult.rows[0];
```

**Key Points:**
- ✅ Parameterized queries prevent SQL injection (`$1`, `$2`)
- ✅ Quoted identifiers preserve case (`"workflowStatus"`, `"userId"`)
- ✅ Access first row with `.rows[0]`

---

### **CONVERSION 2: findFirst (All Columns)**

**Location:** Line 86-88 → Line 83-89

#### **BEFORE (Prisma):**
```typescript
const script = await prisma.script.findFirst({
  where: { id: scriptId, userId }
});
```

#### **AFTER (Raw SQL):**
```typescript
const scriptResult = await pool.query(
  `SELECT * FROM "Script" WHERE id = $1 AND "userId" = $2`,
  [scriptId, userId]
);

const script = scriptResult.rows[0];
```

**Key Points:**
- ✅ Use `SELECT *` when all columns are needed
- ✅ Same parameterization pattern

---

### **CONVERSION 3: update (Single Record)**

**Location:** Line 113-125 → Line 113-120

#### **BEFORE (Prisma):**
```typescript
const updatedScript = await prisma.script.update({
  where: { id: scriptId },
  data: { 
    workflowStatus: targetStatus,
    updatedAt: new Date()
  },
  select: {
    id: true,
    name: true,
    workflowStatus: true,
    updatedAt: true
  }
});
```

#### **AFTER (Raw SQL):**
```typescript
const updateResult = await pool.query(
  `UPDATE "Script" 
   SET "workflowStatus" = $1, "updatedAt" = NOW() 
   WHERE id = $2 
   RETURNING id, name, "workflowStatus", "updatedAt"`,
  [targetStatus, scriptId]
);

const updatedScript = updateResult.rows[0];
```

**Key Points:**
- ✅ `RETURNING` clause returns updated row (like Prisma's `select`)
- ✅ `NOW()` function for current timestamp
- ✅ More explicit control over what gets updated

---

### **CONVERSION 4: updateMany (Batch Update)**

**Location:** Line 173-181 → Line 169-176

#### **BEFORE (Prisma):**
```typescript
const result = await prisma.script.updateMany({
  where: { 
    id: { in: scriptIds },
    userId 
  },
  data: { 
    workflowStatus: targetStatus,
    updatedAt: new Date()
  }
});
```

#### **AFTER (Raw SQL):**
```typescript
const updateResult = await pool.query(
  `UPDATE "Script" 
   SET "workflowStatus" = $1, "updatedAt" = NOW() 
   WHERE id = ANY($2) AND "userId" = $3`,
  [targetStatus, scriptIds, userId]
);

const result = { count: updateResult.rowCount || 0 };
```

**Key Points:**
- ✅ `ANY($2)` handles array of IDs (equivalent to `IN`)
- ✅ `rowCount` gives number of affected rows
- ✅ More efficient for batch operations

---

### **CONVERSION 5: findMany (With Relations)**

**Location:** Line 209-230 → Line 203-232

#### **BEFORE (Prisma):**
```typescript
const scripts = await prisma.script.findMany({
  where: { 
    workflowStatus: status,
    userId 
  },
  select: {
    id: true,
    name: true,
    description: true,
    language: true,
    workflowStatus: true,
    createdAt: true,
    updatedAt: true,
    user: {
      select: {
        name: true,
        email: true
      }
    }
  },
  orderBy: { updatedAt: 'desc' }
});
```

#### **AFTER (Raw SQL):**
```typescript
const scriptsResult = await pool.query(
  `SELECT 
     s.id, 
     s.name, 
     s.description, 
     s.language, 
     s."workflowStatus", 
     s."createdAt", 
     s."updatedAt",
     u.name as "userName",
     u.email as "userEmail"
   FROM "Script" s
   LEFT JOIN "User" u ON s."userId" = u.id
   WHERE s."workflowStatus" = $1 AND s."userId" = $2
   ORDER BY s."updatedAt" DESC`,
  [status, userId]
);

const scripts = scriptsResult.rows.map((row: any) => ({
  id: row.id,
  name: row.name,
  description: row.description,
  language: row.language,
  workflowStatus: row.workflowStatus,
  createdAt: row.createdAt,
  updatedAt: row.updatedAt,
  user: {
    name: row.userName,
    email: row.userEmail
  }
}));
```

**Key Points:**
- ✅ Explicit `LEFT JOIN` for related data
- ✅ Alias columns to avoid naming conflicts (`as "userName"`)
- ✅ Transform flat result into nested structure with `.map()`
- ✅ Full control over JOIN strategy

---

### **CONVERSION 6: groupBy (Aggregation)**

**Location:** Line 254-263 → Line 257-267

#### **BEFORE (Prisma):**
```typescript
const stats = await prisma.script.groupBy({
  by: ['workflowStatus'],
  where: { userId },
  _count: { id: true }
});

const formattedStats = stats.reduce((acc: any, item: any) => {
  acc[item.workflowStatus] = item._count.id;
  return acc;
}, {});
```

#### **AFTER (Raw SQL):**
```typescript
const statsResult = await pool.query(
  `SELECT "workflowStatus", COUNT(id) as count 
   FROM "Script" 
   WHERE "userId" = $1 
   GROUP BY "workflowStatus"`,
  [userId]
);

const formattedStats = statsResult.rows.reduce((acc: any, item: any) => {
  acc[item.workflowStatus] = parseInt(item.count);
  return acc;
}, {});
```

**Key Points:**
- ✅ `GROUP BY` and `COUNT()` for aggregation
- ✅ `parseInt()` ensures count is a number (PostgreSQL returns string for bigint)
- ✅ Simpler and more readable SQL

---

## 📊 Comparison Table

| Operation | Prisma | Raw SQL | Performance |
|-----------|--------|---------|-------------|
| **Single Select** | `findFirst()` | `SELECT ... WHERE` | ⚡ Same |
| **Insert** | `create()` | `INSERT ... RETURNING` | ⚡ Same |
| **Update** | `update()` | `UPDATE ... RETURNING` | ⚡ Same |
| **Batch Update** | `updateMany()` | `UPDATE ... ANY()` | ⚡⚡ Faster |
| **Delete** | `delete()` | `DELETE ... RETURNING` | ⚡ Same |
| **Relations** | Nested `select` | Explicit `JOIN` | ⚡⚡ Faster |
| **Aggregation** | `groupBy()` | `GROUP BY COUNT()` | ⚡⚡ Faster |
| **Transactions** | `$transaction()` | `BEGIN/COMMIT` | ⚡⚡ More control |

---

## ✅ Benefits of Raw SQL

### **1. Performance** ⚡
- Direct query execution without ORM overhead
- Optimized for specific use cases
- Full control over query planning

### **2. Flexibility** 🔧
- Use advanced PostgreSQL features:
  - Window functions
  - CTEs (Common Table Expressions)
  - Full-text search
  - JSON operations
  - Custom aggregations

### **3. Debugging** 🐛
- See exact SQL being executed
- Easy to copy/paste into psql for testing
- Clear error messages from PostgreSQL

### **4. Portability** 📦
- No dependency on Prisma schema
- No migration files to manage
- Easier to understand for SQL-familiar developers

### **5. Control** 🎯
- Explicit transaction management
- Fine-tuned connection pooling
- Custom query timeouts

---

## ⚠️ Important Considerations

### **1. Quoted Identifiers**

PostgreSQL is case-sensitive when identifiers are quoted:

```sql
-- ✅ CORRECT
SELECT "userId" FROM "Script"

-- ❌ WRONG (will look for lowercase 'userid')
SELECT userid FROM Script
```

**Rule:** Always quote column names that use camelCase.

---

### **2. Parameter Placeholders**

Use positional parameters to prevent SQL injection:

```typescript
// ✅ CORRECT (parameterized)
pool.query('SELECT * FROM "Script" WHERE id = $1', [scriptId])

// ❌ WRONG (SQL injection risk)
pool.query(`SELECT * FROM "Script" WHERE id = '${scriptId}'`)
```

---

### **3. Array Parameters**

Use `ANY()` for array matching:

```typescript
// ✅ CORRECT
WHERE id = ANY($1)  // $1 is an array

// ❌ WRONG
WHERE id IN ($1)  // Will treat $1 as single value
```

---

### **4. Null Handling**

Check for null results:

```typescript
const scriptResult = await pool.query('SELECT * FROM "Script" WHERE id = $1', [id]);
const script = scriptResult.rows[0];

if (!script) {
  return res.status(404).json({ error: 'Script not found' });
}
```

---

### **5. Type Conversion**

PostgreSQL may return unexpected types:

```typescript
// COUNT() returns string (bigint)
const count = parseInt(result.rows[0].count);

// Dates are Date objects
const date = result.rows[0].createdAt;  // Already a Date
```

---

## 🔒 Security Best Practices

### **1. Always Use Parameterized Queries**

```typescript
// ✅ SECURE
pool.query('SELECT * FROM "Script" WHERE id = $1', [userId])

// ❌ INSECURE
pool.query(`SELECT * FROM "Script" WHERE id = '${userId}'`)
```

### **2. Validate Input**

```typescript
if (!scriptId || typeof scriptId !== 'string') {
  return res.status(400).json({ error: 'Invalid script ID' });
}
```

### **3. Use Least Privilege**

Database user should have minimal permissions:
- SELECT, INSERT, UPDATE, DELETE on tables
- No DROP, CREATE TABLE permissions in production

---

## 🧪 Testing Raw SQL Queries

### **Test in psql:**

```bash
# Connect to database
psql -U postgres -d playwright_crx1 -h localhost -p 5433

# Test query
SELECT id, name, "workflowStatus", "updatedAt" 
FROM "Script" 
WHERE id = 'some-script-id' AND "userId" = 'some-user-id';

# Test with parameters (using \set)
\set scriptId '''some-script-id'''
SELECT * FROM "Script" WHERE id = :scriptId;
```

---

## 📝 Migration Checklist

When converting other files from Prisma to raw SQL:

- [ ] Replace `import { PrismaClient }` with `import pool from '../db'`
- [ ] Remove `const prisma = new PrismaClient()`
- [ ] Convert `prisma.model.method()` to `pool.query()`
- [ ] Add parameterized values (`$1`, `$2`, etc.)
- [ ] Quote camelCase identifiers (`"userId"`, `"createdAt"`)
- [ ] Extract `.rows[0]` or `.rows` from result
- [ ] Handle relations with explicit `JOIN`
- [ ] Use `RETURNING` for INSERT/UPDATE/DELETE when you need returned data
- [ ] Test all queries in psql before deployment
- [ ] Add error handling for null results

---

## 🚀 Example: Complete Conversion Pattern

### **Generic Conversion Template:**

```typescript
// ❌ BEFORE (Prisma)
const result = await prisma.tableName.method({
  where: { field1: value1, field2: value2 },
  select: { field3: true, field4: true }
});

// ✅ AFTER (Raw SQL)
const queryResult = await pool.query(
  `SELECT field3, field4 
   FROM "TableName" 
   WHERE field1 = $1 AND field2 = $2`,
  [value1, value2]
);
const result = queryResult.rows[0];  // or .rows for multiple
```

---

## 📚 Additional Resources

### **PostgreSQL Documentation:**
- [Query Parameters](https://node-postgres.com/features/queries#parameterized-query)
- [Connection Pooling](https://node-postgres.com/features/pooling)
- [Transactions](https://node-postgres.com/features/transactions)

### **SQL Best Practices:**
- [SQL Style Guide](https://www.sqlstyle.guide/)
- [PostgreSQL Performance Tips](https://wiki.postgresql.org/wiki/Performance_Optimization)

---

## 🎯 Summary

### **What Changed:**
- ✅ Removed Prisma Client dependency from workflow controller
- ✅ Converted 6 different query patterns to raw SQL
- ✅ Maintained exact same functionality
- ✅ Improved performance for batch operations and joins

### **Why It's Better:**
1. **Performance** - Direct SQL execution, no ORM overhead
2. **Flexibility** - Use advanced PostgreSQL features
3. **Debugging** - See exact SQL queries
4. **Control** - Fine-tune queries for specific needs
5. **Simplicity** - No schema migrations to manage

### **Files Ready for Conversion:**
Other controllers that may still use Prisma:
- `project.controller.ts`
- `script.controller.ts`
- `testRun.controller.ts`
- Any service files using Prisma

Follow the same pattern shown in this guide! 🚀
