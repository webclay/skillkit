# Prisma Schema Reference

## Basic Schema Structure

```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DATABASE_URL_UNPOOLED") // For migrations
}

generator client {
  provider = "prisma-client"  // v7 syntax
}
```

## Field Types

| Type | Description |
|------|-------------|
| `String` | Text |
| `Int` | Integer |
| `Float` | Decimal number |
| `Boolean` | True/false |
| `DateTime` | Date and time |
| `Json` | JSON data |
| `Bytes` | Binary data |
| `Decimal` | Precise decimals |
| `BigInt` | Large integers |

## Modifiers

| Modifier | Description |
|----------|-------------|
| `?` | Optional field |
| `[]` | Array |
| `@id` | Primary key |
| `@unique` | Unique constraint |
| `@default()` | Default value |
| `@relation` | Relation definition |
| `@map()` | Custom column name |
| `@@map()` | Custom table name |
| `@@index()` | Database index |
| `@@unique()` | Composite unique |

## Complete Example

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  role      Role     @default(USER)
  profile   Profile?
  posts     Post[]
  comments  Comment[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
  @@map("users")
}

model Profile {
  id     Int     @id @default(autoincrement())
  bio    String?
  avatar String?
  userId String  @unique
  user   User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("profiles")
}

model Post {
  id        Int       @id @default(autoincrement())
  title     String
  content   String?   @db.Text
  published Boolean   @default(false)
  authorId  String
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments  Comment[]
  tags      Tag[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([authorId])
  @@index([published])
  @@map("posts")
}

model Comment {
  id        Int      @id @default(autoincrement())
  content   String
  postId    Int
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())

  @@index([postId])
  @@index([authorId])
  @@map("comments")
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]

  @@map("tags")
}

enum Role {
  USER
  ADMIN
  MODERATOR
}
```

## Relations

### One-to-One

```prisma
model User {
  id      Int      @id
  profile Profile?
}

model Profile {
  id     Int  @id
  userId Int  @unique
  user   User @relation(fields: [userId], references: [id])
}
```

### One-to-Many

```prisma
model User {
  id    Int    @id
  posts Post[]
}

model Post {
  id       Int  @id
  authorId Int
  author   User @relation(fields: [authorId], references: [id])
}
```

### Many-to-Many (Implicit)

```prisma
model Post {
  id   Int   @id
  tags Tag[]
}

model Tag {
  id    Int    @id
  posts Post[]
}
```

### Many-to-Many (Explicit)

```prisma
model Post {
  id       Int        @id
  postTags PostTag[]
}

model Tag {
  id       Int        @id
  postTags PostTag[]
}

model PostTag {
  postId Int
  tagId  Int
  post   Post @relation(fields: [postId], references: [id])
  tag    Tag  @relation(fields: [tagId], references: [id])

  @@id([postId, tagId])
}
```

## Enums (v7 with mapping)

```prisma
enum Status {
  ACTIVE   @map("active")
  INACTIVE @map("inactive")
  PENDING  @map("pending")
}
```

## Default Values

```prisma
model Example {
  id        Int      @id @default(autoincrement())
  uuid      String   @default(uuid())
  cuid      String   @default(cuid())
  now       DateTime @default(now())
  bool      Boolean  @default(false)
  number    Int      @default(0)
  text      String   @default("default")
}
```
