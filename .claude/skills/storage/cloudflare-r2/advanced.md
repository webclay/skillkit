# Advanced R2 Patterns

## Error Handling

Parse specific R2 error types for helpful user feedback:

```ts
catch (error) {
  let errorMessage = 'Failed to process upload';
  if (error instanceof Error) {
    if (error.message.includes('EPROTO') || error.message.includes('SSL')) {
      errorMessage = 'R2 connection failed - check API credentials in .env';
    } else if (error.message.includes('AccessDenied')) {
      errorMessage = 'R2 access denied - check API token permissions';
    } else if (error.message.includes('NoSuchBucket')) {
      errorMessage = 'R2 bucket not found - check R2_BUCKET_NAME in .env';
    }
  }
}
```

---

## File Organization Pattern

User-centric folder structure for multi-tenant apps:

```
{userId}/images/{timestamp}-{sanitized-filename}
{userId}/videos/{timestamp}-{sanitized-filename}
```

Benefits:
- Easy to query/delete all files for a user
- Natural isolation between users
- Timestamp prefix prevents filename collisions

---

## Helper Functions

### Filename Sanitization

```ts
function sanitizeFileName(fileName: string): string {
  return fileName
    .toLowerCase()
    .replace(/[^a-z0-9.-]/g, '-')
    .replace(/-+/g, '-')
    .replace(/^-|-$/g, '');
}
```

### Content Type Detection

```ts
function getContentTypeFromFileName(fileName: string): string {
  const ext = fileName.split('.').pop()?.toLowerCase();
  const types: Record<string, string> = {
    jpg: 'image/jpeg',
    jpeg: 'image/jpeg',
    png: 'image/png',
    webp: 'image/webp',
    gif: 'image/gif',
    mp4: 'video/mp4',
    webm: 'video/webm',
    mov: 'video/quicktime',
  };
  return types[ext || ''] || 'application/octet-stream';
}
```

### Extract Filename from URL

```ts
function getFileNameFromUrl(url: string): string {
  const urlPath = new URL(url).pathname;
  const fileName = urlPath.split('/').pop() || 'file';
  return sanitizeFileName(fileName);
}
```

---

## Core Upload Functions

### Direct Upload (User-uploaded files)

```ts
import { PutObjectCommand } from '@aws-sdk/client-s3';
import { r2, R2_BUCKET, R2_PUBLIC_URL } from '@/lib/r2';

export async function uploadToR2(
  userId: string,
  fileName: string,
  body: Buffer | Uint8Array,
  contentType: string,
  folder: 'images' | 'videos' = 'images'
): Promise<string> {
  const timestamp = Date.now();
  const sanitizedName = sanitizeFileName(fileName);
  const key = `${userId}/${folder}/${timestamp}-${sanitizedName}`;

  await r2.send(
    new PutObjectCommand({
      Bucket: R2_BUCKET,
      Key: key,
      Body: body,
      ContentType: contentType,
    })
  );

  return `${R2_PUBLIC_URL}/${key}`;
}
```

### Copy from External URL (AI service to R2)

For save-to-library pattern - copies temporary AI-generated content to permanent R2 storage:

```ts
export async function copyToR2(
  userId: string,
  sourceUrl: string,
  folder: 'images' | 'videos' = 'images'
): Promise<string> {
  const response = await fetch(sourceUrl);
  if (!response.ok) {
    throw new Error(`Failed to fetch from ${sourceUrl}`);
  }

  const arrayBuffer = await response.arrayBuffer();
  const buffer = Buffer.from(arrayBuffer);

  const fileName = getFileNameFromUrl(sourceUrl);
  const contentType =
    response.headers.get('content-type') ||
    getContentTypeFromFileName(fileName);

  const timestamp = Date.now();
  const key = `${userId}/${folder}/${timestamp}-${fileName}`;

  await r2.send(
    new PutObjectCommand({
      Bucket: R2_BUCKET,
      Key: key,
      Body: buffer,
      ContentType: contentType,
    })
  );

  return `${R2_PUBLIC_URL}/${key}`;
}
```

---

## API Endpoint Pattern

### File Upload Validation

```ts
const MAX_FILE_SIZE = 30 * 1024 * 1024; // 30MB
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];

// Validate type
if (!ALLOWED_TYPES.includes(file.type)) {
  return Response.json(
    { error: 'Invalid file type. Only JPEG, PNG, and WebP are allowed.' },
    { status: 400 }
  );
}

// Validate size
if (file.size > MAX_FILE_SIZE) {
  return Response.json(
    { error: 'File too large. Maximum size is 30MB.' },
    { status: 400 }
  );
}
```

### FormData to Buffer

```ts
const formData = await request.formData();
const file = formData.get('file') as File | null;

if (!file) {
  return Response.json({ error: 'No file provided' }, { status: 400 });
}

const arrayBuffer = await file.arrayBuffer();
const buffer = Buffer.from(arrayBuffer);
```

---

## Save-to-Library Pattern

For AI-generated content, don't save everything immediately. Use a two-phase approach:

### Phase 1: Store Temporary URL

When AI generates content, store the temporary URL from the AI service:

```ts
// During generation - just store the temp URL
const aiResult = await generateImage(prompt);
await db.image.create({
  data: {
    userId,
    imageUrl: aiResult.tempUrl, // Temporary URL from AI service
    isSaved: false,
  },
});
```

### Phase 2: Copy to R2 on Explicit Save

Only when user clicks "Save to Library":

```ts
// When user saves
const image = await db.image.findUnique({ where: { id: imageId } });

if (!image.isSaved && image.imageUrl) {
  const r2Url = await copyToR2(userId, image.imageUrl, 'images');
  await db.image.update({
    where: { id: imageId },
    data: {
      imageUrl: r2Url,
      isSaved: true,
    },
  });
}
```

### Benefits
- No wasted storage for discarded content
- Faster generation (no upload blocking)
- Lower costs
- Better UX (user controls what's saved)

---

## Database Schema Pattern

Add `isSaved` field for save-to-library functionality:

```prisma
model Image {
  id        String   @id @default(cuid())
  userId    String
  imageUrl  String?  // Initially temp URL, then R2 URL after save
  isSaved   Boolean  @default(false)
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
}

model Video {
  id           String   @id @default(cuid())
  userId       String
  videoUrl     String?
  thumbnailUrl String?
  isSaved      Boolean  @default(false)
  createdAt    DateTime @default(now())
  user         User     @relation(fields: [userId], references: [id])
}
```

---

## Environment Variables

```env
CLOUDFLARE_ACCOUNT_ID=your-account-id
R2_ACCESS_KEY_ID=your-access-key
R2_SECRET_ACCESS_KEY=your-secret-key
R2_BUCKET_NAME=your-bucket
R2_PUBLIC_URL=https://media.yourdomain.com
```

---

## Quick Reference

| Issue | Solution |
|-------|----------|
| SSL handshake error | Add `forcePathStyle: true` to S3Client |
| Access denied | Check R2 API token permissions in Cloudflare dashboard |
| Bucket not found | Verify `R2_BUCKET_NAME` matches exactly |
| CORS on download | Use direct link, not fetch-based download |
| High storage costs | Use save-to-library pattern (don't save everything) |
| Filename collisions | Use timestamp prefix: `{timestamp}-{filename}` |
