# Prisma Query Reference

## Filtering

### Basic Filters

```ts
// Equals
const user = await db.user.findFirst({
  where: { email: 'alice@example.com' },
});

// Not equals
const users = await db.user.findMany({
  where: { NOT: { role: 'ADMIN' } },
});

// Contains (case-insensitive with mode)
const users = await db.user.findMany({
  where: {
    name: { contains: 'alice', mode: 'insensitive' },
  },
});

// Starts with / Ends with
const users = await db.user.findMany({
  where: {
    email: { startsWith: 'user', endsWith: '@gmail.com' },
  },
});
```

### Comparison Operators

```ts
const posts = await db.post.findMany({
  where: {
    viewCount: {
      gt: 100,    // Greater than
      gte: 100,   // Greater than or equal
      lt: 1000,   // Less than
      lte: 1000,  // Less than or equal
    },
  },
});

// Date comparisons
const recentPosts = await db.post.findMany({
  where: {
    createdAt: { gte: new Date('2024-01-01') },
  },
});
```

### Logical Operators

```ts
// AND (implicit)
const users = await db.user.findMany({
  where: {
    role: 'USER',
    createdAt: { gte: new Date('2024-01-01') },
  },
});

// OR
const users = await db.user.findMany({
  where: {
    OR: [
      { email: { contains: '@gmail.com' } },
      { email: { contains: '@yahoo.com' } },
    ],
  },
});

// Complex nested
const posts = await db.post.findMany({
  where: {
    OR: [
      { AND: [{ published: true }, { viewCount: { gte: 100 } }] },
      { author: { role: 'ADMIN' } },
    ],
  },
});
```

### Array/List Filters

```ts
// In list
const users = await db.user.findMany({
  where: { id: { in: [1, 2, 3] } },
});

// Not in list
const users = await db.user.findMany({
  where: { id: { notIn: [1, 2, 3] } },
});
```

### Relation Filters

```ts
// Has related records
const usersWithPosts = await db.user.findMany({
  where: { posts: { some: {} } },
});

// Has specific related records
const usersWithPublishedPosts = await db.user.findMany({
  where: { posts: { some: { published: true } } },
});

// All related records match
const usersAllPublished = await db.user.findMany({
  where: { posts: { every: { published: true } } },
});

// No related records
const usersWithoutPosts = await db.user.findMany({
  where: { posts: { none: {} } },
});
```

## Selecting & Including

### Select Specific Fields

```ts
const users = await db.user.findMany({
  select: {
    id: true,
    email: true,
    name: true,
  },
});
// Returns: { id, email, name } only
```

### Include Relations

```ts
const users = await db.user.findMany({
  include: {
    posts: true,
    profile: true,
  },
});
```

### Nested Select in Relations

```ts
const users = await db.user.findMany({
  select: {
    id: true,
    name: true,
    posts: {
      select: { title: true, published: true },
      where: { published: true },
      orderBy: { createdAt: 'desc' },
      take: 5,
    },
  },
});
```

### Relation Count

```ts
const users = await db.user.findMany({
  include: {
    _count: {
      select: { posts: true, comments: true },
    },
  },
});
// Access: user._count.posts
```

## Ordering & Pagination

### Order By

```ts
const posts = await db.post.findMany({
  orderBy: { createdAt: 'desc' },
});

// Multiple fields
const posts = await db.post.findMany({
  orderBy: [
    { published: 'desc' },
    { createdAt: 'desc' },
  ],
});

// Order by relation
const users = await db.user.findMany({
  orderBy: { posts: { _count: 'desc' } },
});
```

### Offset Pagination

```ts
const page = 2;
const pageSize = 10;

const users = await db.user.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: { createdAt: 'desc' },
});
```

### Cursor Pagination (Better Performance)

```ts
const users = await db.user.findMany({
  take: 10,
  skip: 1, // Skip the cursor itself
  cursor: { id: lastUserId },
  orderBy: { id: 'asc' },
});
```

## Aggregations

### Count

```ts
const count = await db.user.count();

const publishedCount = await db.post.count({
  where: { published: true },
});
```

### Aggregate Functions

```ts
const stats = await db.post.aggregate({
  _count: true,
  _sum: { viewCount: true },
  _avg: { viewCount: true },
  _min: { createdAt: true },
  _max: { viewCount: true },
  where: { published: true },
});
```

### Group By

```ts
const postsByAuthor = await db.post.groupBy({
  by: ['authorId'],
  _count: { id: true },
  _sum: { viewCount: true },
  orderBy: { _count: { id: 'desc' } },
});
```

## Transactions

### Batch Operations

```ts
const result = await db.$transaction([
  db.user.create({ data: { email: 'user@example.com' } }),
  db.post.create({ data: { title: 'Post', authorId: 1 } }),
]);
```

### Interactive Transactions

```ts
const transfer = await db.$transaction(async (tx) => {
  const from = await tx.account.update({
    where: { id: 1 },
    data: { balance: { decrement: 100 } },
  });

  if (from.balance < 0) throw new Error('Insufficient funds');

  await tx.account.update({
    where: { id: 2 },
    data: { balance: { increment: 100 } },
  });

  return from;
});
```

## Upsert

```ts
const user = await db.user.upsert({
  where: { email: 'alice@example.com' },
  update: { name: 'Alice Updated' },
  create: { email: 'alice@example.com', name: 'Alice' },
});
```

## Atomic Operations

```ts
// Increment/Decrement
const post = await db.post.update({
  where: { id: 1 },
  data: { viewCount: { increment: 1 } },
});

// Multiply/Divide
const product = await db.product.update({
  where: { id: 1 },
  data: { price: { multiply: 1.1 } }, // 10% increase
});
```

## Raw SQL

```ts
// Query (read)
const users = await db.$queryRaw<User[]>`
  SELECT * FROM users WHERE email LIKE ${'%@example.com'}
`;

// Execute (write)
await db.$executeRaw`
  UPDATE users SET name = ${'Updated'} WHERE id = ${1}
`;
```

## Full-Text Search

**Enable in schema:**
```prisma
model Post {
  // ...fields
  @@fulltext([title, content])
}
```

**Search:**
```ts
const posts = await db.post.findMany({
  where: {
    title: { search: 'typescript react' },
  },
});
```
