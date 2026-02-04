---
name: appwrite
description: Appwrite Backend-as-a-Service platform. Trigger words - appwrite, baas, backend as a service, appwrite auth, appwrite database, appwrite storage, appwrite functions
---

# Appwrite

Complete BaaS platform with auth, databases, storage, functions, and realtime.

## When to Use This Skill

- Need complete backend without building from scratch
- Want self-hosting option for data sovereignty
- Building cross-platform apps (web, mobile, desktop)
- Need realtime updates across clients
- Want multiple auth methods out of the box

## When NOT to Use

- Need fine-grained SQL control (use Postgres)
- Require specific cloud provider integrations
- Building simple static sites

## Setup

```bash
npm install appwrite
```

```typescript
// lib/appwrite.ts
import { Client, Account, Databases, Storage, ID } from 'appwrite'

export const client = new Client()
  .setEndpoint('https://cloud.appwrite.io/v1')
  .setProject('<PROJECT_ID>')

export const account = new Account(client)
export const databases = new Databases(client)
export const storage = new Storage(client)
export { ID }
```

## Authentication

```typescript
// Register
async function register(email: string, password: string, name: string) {
  await account.create(ID.unique(), email, password, name)
  await login(email, password)
}

// Login
async function login(email: string, password: string) {
  await account.createEmailPasswordSession(email, password)
}

// Logout
async function logout() {
  await account.deleteSession('current')
}

// Get current user
async function getCurrentUser() {
  try {
    return await account.get()
  } catch {
    return null
  }
}
```

### OAuth

```typescript
import { OAuthProvider } from 'appwrite'

function loginWithGoogle() {
  account.createOAuth2Session(
    OAuthProvider.Google,
    'http://localhost:3000/dashboard',
    'http://localhost:3000/login'
  )
}
```

## Database Operations

```typescript
import { Query } from 'appwrite'

const DATABASE_ID = 'main'
const COLLECTION_ID = 'posts'

// Create
async function createPost(title: string, content: string) {
  return await databases.createDocument(
    DATABASE_ID,
    COLLECTION_ID,
    ID.unique(),
    { title, content, createdAt: new Date().toISOString() }
  )
}

// List with queries
async function getPosts() {
  return await databases.listDocuments(DATABASE_ID, COLLECTION_ID, [
    Query.orderDesc('createdAt'),
    Query.limit(10)
  ])
}

// Update
async function updatePost(id: string, data: Partial<Post>) {
  return await databases.updateDocument(DATABASE_ID, COLLECTION_ID, id, data)
}

// Delete
async function deletePost(id: string) {
  await databases.deleteDocument(DATABASE_ID, COLLECTION_ID, id)
}
```

### Query Examples

```typescript
Query.equal('status', 'published')
Query.notEqual('status', 'draft')
Query.lessThan('price', 100)
Query.search('title', 'keyword')
Query.contains('tags', ['react', 'typescript'])
Query.orderDesc('createdAt')
Query.limit(25)
Query.offset(50)
```

## Storage

```typescript
const BUCKET_ID = 'uploads'

// Upload
async function uploadFile(file: File) {
  return await storage.createFile(BUCKET_ID, ID.unique(), file)
}

// Get URL
function getFileUrl(fileId: string) {
  return storage.getFileView(BUCKET_ID, fileId)
}

// Get preview with transformations
function getFilePreview(fileId: string, width?: number, height?: number) {
  return storage.getFilePreview(BUCKET_ID, fileId, width, height)
}

// Delete
async function deleteFile(fileId: string) {
  await storage.deleteFile(BUCKET_ID, fileId)
}
```

## Realtime

```typescript
// Subscribe to collection changes
const unsubscribe = client.subscribe(
  `databases.main.collections.posts.documents`,
  (response) => {
    if (response.events.includes('databases.*.collections.*.documents.*.create')) {
      console.log('New document:', response.payload)
    }
  }
)

// Cleanup
unsubscribe()
```

## Permissions

```typescript
import { Permission, Role } from 'appwrite'

// Create with permissions
await databases.createDocument(
  DATABASE_ID,
  COLLECTION_ID,
  ID.unique(),
  { title: 'My Post' },
  [
    Permission.read(Role.any()),           // Public read
    Permission.update(Role.user(userId)),  // Owner can update
    Permission.delete(Role.user(userId))   // Owner can delete
  ]
)
```

## Server SDK (Node.js)

```typescript
// lib/appwrite-server.ts
import { Client, Databases, Users } from 'node-appwrite'

export function createAdminClient() {
  const client = new Client()
    .setEndpoint(process.env.APPWRITE_ENDPOINT!)
    .setProject(process.env.APPWRITE_PROJECT_ID!)
    .setKey(process.env.APPWRITE_API_KEY!)

  return {
    databases: new Databases(client),
    users: new Users(client)
  }
}
```

## Environment Variables

```env
NEXT_PUBLIC_APPWRITE_ENDPOINT=https://cloud.appwrite.io/v1
NEXT_PUBLIC_APPWRITE_PROJECT_ID=your_project_id
APPWRITE_API_KEY=your_api_key  # Server-only
```

## Tips

- Use server SDK for sensitive operations
- Set proper permissions on collections
- Use indexes for frequently queried attributes
- Never expose API keys in client code
- Use `Role.any()` only for read, not write

## How to Verify

### Quick Checks
- Auth flow works (register, login, logout)
- Documents created in Appwrite Console
- Files uploaded to storage bucket
- Realtime events fire on changes

### Common Issues
- "Unauthorized": Check project ID and permissions
- "Document not found": Verify collection permissions
- OAuth redirect fails: Check callback URLs in console
