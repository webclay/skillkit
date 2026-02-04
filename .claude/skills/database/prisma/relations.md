# Prisma Relations Reference

## One-to-One

```prisma
model User {
  id      Int      @id @default(autoincrement())
  profile Profile?
}

model Profile {
  id     Int  @id @default(autoincrement())
  bio    String?
  userId Int  @unique  // Foreign key, unique for 1:1
  user   User @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

**Query:**
```ts
const userWithProfile = await db.user.findUnique({
  where: { id: 1 },
  include: { profile: true },
});
```

**Create with relation:**
```ts
const user = await db.user.create({
  data: {
    email: 'alice@example.com',
    profile: {
      create: { bio: 'Software developer' },
    },
  },
  include: { profile: true },
});
```

## One-to-Many

```prisma
model User {
  id    Int    @id @default(autoincrement())
  posts Post[]
}

model Post {
  id       Int  @id @default(autoincrement())
  title    String
  authorId Int
  author   User @relation(fields: [authorId], references: [id], onDelete: Cascade)

  @@index([authorId])
}
```

**Query:**
```ts
const userWithPosts = await db.user.findUnique({
  where: { id: 1 },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
    },
  },
});
```

**Create with nested records:**
```ts
const user = await db.user.create({
  data: {
    email: 'bob@example.com',
    posts: {
      create: [
        { title: 'First Post' },
        { title: 'Second Post' },
      ],
    },
  },
});
```

**Add to existing:**
```ts
const post = await db.post.create({
  data: {
    title: 'New Post',
    author: { connect: { id: 1 } },
  },
});
```

## Many-to-Many (Implicit)

Prisma auto-creates join table:

```prisma
model Post {
  id   Int   @id @default(autoincrement())
  tags Tag[]
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}
```

**Query:**
```ts
const postWithTags = await db.post.findUnique({
  where: { id: 1 },
  include: { tags: true },
});
```

**Connect existing:**
```ts
const post = await db.post.update({
  where: { id: 1 },
  data: {
    tags: {
      connect: [{ id: 1 }, { id: 2 }],
    },
  },
});
```

**Create or connect:**
```ts
const post = await db.post.create({
  data: {
    title: 'Tagged Post',
    tags: {
      connectOrCreate: [
        {
          where: { name: 'typescript' },
          create: { name: 'typescript' },
        },
      ],
    },
  },
});
```

**Disconnect:**
```ts
await db.post.update({
  where: { id: 1 },
  data: {
    tags: { disconnect: [{ id: 1 }] },
  },
});
```

**Set (replace all):**
```ts
await db.post.update({
  where: { id: 1 },
  data: {
    tags: { set: [{ id: 2 }, { id: 3 }] },
  },
});
```

## Many-to-Many (Explicit)

For extra fields on the join:

```prisma
model Post {
  id       Int        @id @default(autoincrement())
  postTags PostTag[]
}

model Tag {
  id       Int        @id @default(autoincrement())
  name     String     @unique
  postTags PostTag[]
}

model PostTag {
  postId    Int
  tagId     Int
  assignedAt DateTime @default(now())
  assignedBy String?

  post Post @relation(fields: [postId], references: [id], onDelete: Cascade)
  tag  Tag  @relation(fields: [tagId], references: [id], onDelete: Cascade)

  @@id([postId, tagId])
}
```

**Query:**
```ts
const postWithTags = await db.post.findUnique({
  where: { id: 1 },
  include: {
    postTags: {
      include: { tag: true },
    },
  },
});
```

**Create:**
```ts
await db.postTag.create({
  data: {
    postId: 1,
    tagId: 2,
    assignedBy: 'admin',
  },
});
```

## Self-Relations

### One-to-Many (Parent-Child)

```prisma
model Category {
  id       Int        @id @default(autoincrement())
  name     String
  parentId Int?
  parent   Category?  @relation("CategoryTree", fields: [parentId], references: [id])
  children Category[] @relation("CategoryTree")
}
```

### Many-to-Many (Followers)

```prisma
model User {
  id        Int    @id @default(autoincrement())
  followers User[] @relation("UserFollows")
  following User[] @relation("UserFollows")
}
```

## Referential Actions

```prisma
model Post {
  authorId Int
  author   User @relation(fields: [authorId], references: [id], onDelete: Cascade, onUpdate: Cascade)
}
```

| Action | On Delete | On Update |
|--------|-----------|-----------|
| `Cascade` | Delete related | Update related |
| `Restrict` | Prevent delete | Prevent update |
| `SetNull` | Set to null | Set to null |
| `SetDefault` | Set to default | Set to default |
| `NoAction` | No action | No action |

## Multiple Relations Between Same Models

```prisma
model User {
  id            Int     @id @default(autoincrement())
  writtenPosts  Post[]  @relation("PostAuthor")
  editedPosts   Post[]  @relation("PostEditor")
}

model Post {
  id       Int   @id @default(autoincrement())
  authorId Int
  editorId Int?
  author   User  @relation("PostAuthor", fields: [authorId], references: [id])
  editor   User? @relation("PostEditor", fields: [editorId], references: [id])
}
```

## Relation Queries

### Filter by relation

```ts
// Users who have posts
const usersWithPosts = await db.user.findMany({
  where: { posts: { some: {} } },
});

// Users with published posts
const usersWithPublished = await db.user.findMany({
  where: { posts: { some: { published: true } } },
});

// Users where ALL posts are published
const usersAllPublished = await db.user.findMany({
  where: { posts: { every: { published: true } } },
});

// Users with no posts
const usersWithoutPosts = await db.user.findMany({
  where: { posts: { none: {} } },
});
```

### Count relations

```ts
const users = await db.user.findMany({
  include: {
    _count: { select: { posts: true } },
  },
});
// Access: user._count.posts
```

### Order by relation count

```ts
const users = await db.user.findMany({
  orderBy: { posts: { _count: 'desc' } },
});
```

## Best Practices

1. **Always add `@@index`** on foreign keys
2. **Use `Cascade`** for owned relations (user → posts)
3. **Use `Restrict`** for shared resources (post → category)
4. **Use explicit join tables** when you need extra fields
5. **Name relations** when multiple exist between same models
