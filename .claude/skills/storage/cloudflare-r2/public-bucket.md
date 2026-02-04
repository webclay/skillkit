# Public Bucket Setup

For serving files publicly (profile images, public documents, etc.), you can enable public access on your R2 bucket.

## Enable Public Access

1. Go to Cloudflare Dashboard > R2
2. Select your bucket
3. Go to Settings > Public Access
4. Enable "Allow Access"

Your files will be available at:
```
https://pub-{hash}.r2.dev/{key}
```

Or connect a custom domain for cleaner URLs.

## Custom Domain Setup

1. In bucket Settings > Custom Domains
2. Add your subdomain (e.g., `files.yourdomain.com`)
3. Cloudflare will automatically configure DNS

Files accessible at:
```
https://files.yourdomain.com/{key}
```

## CORS Configuration

For browser-based uploads, configure CORS in bucket Settings > CORS:

```json
[
  {
    "AllowedOrigins": ["https://yourdomain.com", "http://localhost:3000"],
    "AllowedMethods": ["GET", "PUT", "HEAD"],
    "AllowedHeaders": ["Content-Type"],
    "MaxAgeSeconds": 3600
  }
]
```

**Note:** Replace `yourdomain.com` with your actual domain.

## Public vs Private Files

### Public Bucket Use Cases
- Profile avatars
- Public images/media
- Static assets
- Files that don't require authentication

### Private Bucket Use Cases (use presigned URLs)
- User documents
- Sensitive uploads
- Files requiring access control
- Temporary download links

## Code Pattern for Public Files

When using a public bucket, you don't need presigned URLs for reading:

**lib/r2.ts (extended):**
```ts
// Add to existing r2.ts

export const R2_PUBLIC_URL = process.env.R2_PUBLIC_URL!;
// Set this to: https://pub-{hash}.r2.dev or https://files.yourdomain.com

export function getPublicUrl(key: string) {
  return `${R2_PUBLIC_URL}/${key}`;
}
```

**Environment variable:**
```env
R2_PUBLIC_URL=https://files.yourdomain.com
```

**Usage:**
```ts
import { getPublicUrl } from '@/lib/r2';

// No async needed for public files
const imageUrl = getPublicUrl('avatars/user-123.jpg');
```

## Cache Control

Set cache headers for better performance:

```ts
const command = new PutObjectCommand({
  Bucket: R2_BUCKET,
  Key: key,
  Body: buffer,
  ContentType: file.type,
  CacheControl: 'public, max-age=31536000', // 1 year for immutable content
});
```

For mutable content (like avatars that can change):
```ts
CacheControl: 'public, max-age=3600', // 1 hour
```

## Security Considerations

- Public buckets expose ALL files - don't mix private and public content
- Use separate buckets for public vs private files
- Consider using unique/random file names to prevent enumeration
- Never store sensitive data in public buckets
