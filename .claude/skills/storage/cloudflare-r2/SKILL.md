---
name: cloudflare-r2
description: Cloudflare R2 object storage for file uploads. Trigger words - r2, cloudflare r2, file upload, image upload, object storage, s3, presigned url, upload file, store files, bucket
---

# Cloudflare R2

S3-compatible object storage with zero egress fees. Ideal for file uploads, image storage, and serving static assets.

## When to Use This Skill

- User asks to add file uploads or image uploads
- User mentions Cloudflare R2 or S3-compatible storage
- User wants to store user-uploaded files (avatars, documents, media)
- package.json contains "@aws-sdk/client-s3"
- User asks about presigned URLs or direct uploads

## Instructions

### Step 1: Install AWS SDK

R2 uses the S3-compatible API, so we use AWS SDK v3:

```bash
pnpm add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

### Step 2: Set Environment Variables

Get these from Cloudflare Dashboard > R2 > Manage R2 API Tokens:

```env
CLOUDFLARE_ACCOUNT_ID=your-account-id
R2_ACCESS_KEY_ID=your-access-key-id
R2_SECRET_ACCESS_KEY=your-secret-access-key
R2_BUCKET_NAME=your-bucket-name
```

### Step 3: Create R2 Client

**lib/r2.ts:**
```ts
import { S3Client } from '@aws-sdk/client-s3';

export const r2 = new S3Client({
  region: 'auto',
  endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
  forcePathStyle: true, // REQUIRED for R2
});

export const R2_BUCKET = process.env.R2_BUCKET_NAME!;
export const R2_PUBLIC_URL = process.env.R2_PUBLIC_URL!; // For public buckets
```

### Step 4: Implement Upload Pattern

Choose the right pattern based on your needs:

**Option A: Server-side upload** (small files, simple)
- File goes to your server first, then to R2
- Good for files under 10MB
- See [patterns.md](patterns.md) for implementation

**Option B: Presigned URL upload** (recommended for most cases)
- Client uploads directly to R2
- Your server only generates the upload URL
- Better performance, no server bandwidth cost
- See [patterns.md](patterns.md) for implementation

## Examples

**Generate presigned upload URL:**
```ts
import { PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { r2, R2_BUCKET } from '@/lib/r2';

export async function getUploadUrl(filename: string, contentType: string) {
  const key = `uploads/${Date.now()}-${filename}`;

  const command = new PutObjectCommand({
    Bucket: R2_BUCKET,
    Key: key,
    ContentType: contentType,
  });

  const uploadUrl = await getSignedUrl(r2, command, { expiresIn: 3600 });

  return { uploadUrl, key };
}
```

**Generate presigned download URL:**
```ts
import { GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { r2, R2_BUCKET } from '@/lib/r2';

export async function getDownloadUrl(key: string) {
  const command = new GetObjectCommand({
    Bucket: R2_BUCKET,
    Key: key,
  });

  return getSignedUrl(r2, command, { expiresIn: 3600 });
}
```

**Delete file:**
```ts
import { DeleteObjectCommand } from '@aws-sdk/client-s3';
import { r2, R2_BUCKET } from '@/lib/r2';

export async function deleteFile(key: string) {
  await r2.send(new DeleteObjectCommand({
    Bucket: R2_BUCKET,
    Key: key,
  }));
}
```

## Reference Files

- [patterns.md](patterns.md) - Full upload patterns with React components
- [public-bucket.md](public-bucket.md) - Setting up public access
- [advanced.md](advanced.md) - Helper functions, error handling, save-to-library pattern

## Tips

- Use presigned URLs for files over 5MB
- Set short expiration (1 hour) for upload URLs
- Include content-type in presigned URL to prevent wrong file types
- Store the object key in your database, not the full URL
- Use folders in keys: `avatars/`, `documents/`, `uploads/`

## How to Verify

### Quick Checks
- Presigned URL generation returns a valid URL
- Upload to presigned URL returns 200 status
- File appears in Cloudflare Dashboard > R2 > [Bucket]

### Common Issues

| Issue | Solution |
|-------|----------|
| SSL handshake error | Add `forcePathStyle: true` to S3Client |
| SignatureDoesNotMatch | Check R2_SECRET_ACCESS_KEY is correct |
| AccessDenied | Verify API token has Object Read & Write permission |
| NoSuchBucket | Check R2_BUCKET_NAME matches exactly |
| CORS on download | Use direct link, not fetch-based download |
| CORS on upload | Configure CORS in bucket settings (see [public-bucket.md](public-bucket.md))

## Critical Rules

**ALWAYS include `forcePathStyle: true`** - R2 requires path-style URLs. Without it, you'll get SSL handshake errors.

```ts
// WRONG - will cause SSL errors
const r2 = new S3Client({
  region: 'auto',
  endpoint: `https://${ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: { ... },
});

// CORRECT
const r2 = new S3Client({
  region: 'auto',
  endpoint: `https://${ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: { ... },
  forcePathStyle: true, // REQUIRED
});
```

**Account ID is just the ID, not a URL:**
```env
# WRONG
CLOUDFLARE_ACCOUNT_ID=https://abc123.r2.cloudflarestorage.com

# CORRECT
CLOUDFLARE_ACCOUNT_ID=abc123
```

**Don't use fetch() for downloads** - causes CORS errors:
```ts
// WRONG - CORS error
const response = await fetch(imageUrl);
const blob = await response.blob();

// CORRECT - direct link
const link = document.createElement('a');
link.href = imageUrl;
link.download = 'filename.png';
link.click();
```

**Don't save everything immediately** - use save-to-library pattern for AI-generated content:
```ts
// WRONG - wastes storage if user discards
const image = await generateImage();
await uploadToR2(image);

// CORRECT - only save when user explicitly saves
await db.image.create({ imageUrl: tempUrl, isSaved: false });
// Later, when user clicks save:
const r2Url = await copyToR2(userId, tempUrl);
await db.image.update({ imageUrl: r2Url, isSaved: true });
```

See [advanced.md](advanced.md) for full patterns and helper functions.
