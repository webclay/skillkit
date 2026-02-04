# R2 Upload Patterns

## Pattern 1: Presigned URL Upload (Recommended)

Best for most use cases. Client uploads directly to R2.

### Server Action

**actions/upload.ts:**
```ts
'use server';

import { PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { r2, R2_BUCKET } from '@/lib/r2';

const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'];
const MAX_SIZE = 10 * 1024 * 1024; // 10MB

export async function createUploadUrl(
  filename: string,
  contentType: string,
  folder: string = 'uploads'
) {
  // Validate content type
  if (!ALLOWED_TYPES.includes(contentType)) {
    return { error: 'File type not allowed' };
  }

  // Generate unique key
  const ext = filename.split('.').pop();
  const key = `${folder}/${Date.now()}-${crypto.randomUUID()}.${ext}`;

  const command = new PutObjectCommand({
    Bucket: R2_BUCKET,
    Key: key,
    ContentType: contentType,
  });

  const uploadUrl = await getSignedUrl(r2, command, { expiresIn: 3600 });

  return { uploadUrl, key };
}

export async function getFileUrl(key: string) {
  const command = new GetObjectCommand({
    Bucket: R2_BUCKET,
    Key: key,
  });

  return getSignedUrl(r2, command, { expiresIn: 3600 });
}
```

### React Upload Hook

**hooks/use-upload.ts:**
```ts
'use client';

import { useState } from 'react';
import { createUploadUrl } from '@/actions/upload';

interface UploadState {
  isUploading: boolean;
  progress: number;
  error: string | null;
}

export function useUpload() {
  const [state, setState] = useState<UploadState>({
    isUploading: false,
    progress: 0,
    error: null,
  });

  const upload = async (file: File, folder?: string) => {
    setState({ isUploading: true, progress: 0, error: null });

    try {
      // Get presigned URL from server
      const result = await createUploadUrl(file.name, file.type, folder);

      if ('error' in result) {
        setState({ isUploading: false, progress: 0, error: result.error });
        return null;
      }

      // Upload directly to R2
      const response = await fetch(result.uploadUrl, {
        method: 'PUT',
        body: file,
        headers: {
          'Content-Type': file.type,
        },
      });

      if (!response.ok) {
        throw new Error('Upload failed');
      }

      setState({ isUploading: false, progress: 100, error: null });
      return result.key;
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Upload failed';
      setState({ isUploading: false, progress: 0, error: message });
      return null;
    }
  };

  return { ...state, upload };
}
```

### Upload Component

**components/file-upload.tsx:**
```tsx
'use client';

import { useUpload } from '@/hooks/use-upload';
import { Button } from '@/components/ui/button';

interface FileUploadProps {
  onUploadComplete: (key: string) => void;
  folder?: string;
  accept?: string;
}

export function FileUpload({ onUploadComplete, folder, accept }: FileUploadProps) {
  const { isUploading, error, upload } = useUpload();

  const handleChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    const key = await upload(file, folder);
    if (key) {
      onUploadComplete(key);
    }
  };

  return (
    <div>
      <input
        type="file"
        onChange={handleChange}
        disabled={isUploading}
        accept={accept}
        className="hidden"
        id="file-upload"
      />
      <label htmlFor="file-upload">
        <Button asChild disabled={isUploading}>
          <span>{isUploading ? 'Uploading...' : 'Choose File'}</span>
        </Button>
      </label>
      {error && <p className="text-sm text-destructive mt-2">{error}</p>}
    </div>
  );
}
```

---

## Pattern 2: Server-Side Upload

For small files or when you need to process files before storing.

### Server Action

**actions/upload-server.ts:**
```ts
'use server';

import { PutObjectCommand } from '@aws-sdk/client-s3';
import { r2, R2_BUCKET } from '@/lib/r2';

export async function uploadFile(formData: FormData) {
  const file = formData.get('file') as File;

  if (!file) {
    return { error: 'No file provided' };
  }

  // Convert to buffer
  const buffer = Buffer.from(await file.arrayBuffer());

  // Generate key
  const ext = file.name.split('.').pop();
  const key = `uploads/${Date.now()}-${crypto.randomUUID()}.${ext}`;

  await r2.send(new PutObjectCommand({
    Bucket: R2_BUCKET,
    Key: key,
    Body: buffer,
    ContentType: file.type,
  }));

  return { key };
}
```

### Form Component

```tsx
'use client';

import { uploadFile } from '@/actions/upload-server';

export function ServerUploadForm() {
  const handleSubmit = async (formData: FormData) => {
    const result = await uploadFile(formData);
    if (result.key) {
      console.log('Uploaded:', result.key);
    }
  };

  return (
    <form action={handleSubmit}>
      <input type="file" name="file" required />
      <button type="submit">Upload</button>
    </form>
  );
}
```

---

## Pattern 3: Image Upload with Preview

For avatar or image uploads with immediate preview.

**components/image-upload.tsx:**
```tsx
'use client';

import { useState } from 'react';
import { useUpload } from '@/hooks/use-upload';
import { getFileUrl } from '@/actions/upload';
import Image from 'next/image';

interface ImageUploadProps {
  currentImage?: string;
  onUploadComplete: (key: string) => void;
}

export function ImageUpload({ currentImage, onUploadComplete }: ImageUploadProps) {
  const [preview, setPreview] = useState<string | null>(null);
  const [imageUrl, setImageUrl] = useState<string | null>(null);
  const { isUploading, upload } = useUpload();

  // Load current image URL
  useEffect(() => {
    if (currentImage) {
      getFileUrl(currentImage).then(setImageUrl);
    }
  }, [currentImage]);

  const handleChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    // Show local preview immediately
    const localUrl = URL.createObjectURL(file);
    setPreview(localUrl);

    const key = await upload(file, 'avatars');
    if (key) {
      onUploadComplete(key);
      URL.revokeObjectURL(localUrl);
    }
  };

  const displayUrl = preview || imageUrl;

  return (
    <div className="flex items-center gap-4">
      {displayUrl && (
        <Image
          src={displayUrl}
          alt="Preview"
          width={80}
          height={80}
          className="rounded-full object-cover"
        />
      )}
      <input
        type="file"
        onChange={handleChange}
        disabled={isUploading}
        accept="image/*"
      />
    </div>
  );
}
```

---

## Pattern 4: Multiple File Upload

For uploading multiple files at once.

**hooks/use-multi-upload.ts:**
```ts
'use client';

import { useState } from 'react';
import { createUploadUrl } from '@/actions/upload';

interface FileProgress {
  file: File;
  key?: string;
  status: 'pending' | 'uploading' | 'complete' | 'error';
  error?: string;
}

export function useMultiUpload() {
  const [files, setFiles] = useState<FileProgress[]>([]);

  const uploadAll = async (fileList: FileList, folder?: string) => {
    const fileArray = Array.from(fileList);

    // Initialize all files as pending
    setFiles(fileArray.map(file => ({ file, status: 'pending' })));

    const results: string[] = [];

    for (let i = 0; i < fileArray.length; i++) {
      const file = fileArray[i];

      setFiles(prev => prev.map((f, idx) =>
        idx === i ? { ...f, status: 'uploading' } : f
      ));

      try {
        const result = await createUploadUrl(file.name, file.type, folder);

        if ('error' in result) {
          throw new Error(result.error);
        }

        await fetch(result.uploadUrl, {
          method: 'PUT',
          body: file,
          headers: { 'Content-Type': file.type },
        });

        results.push(result.key);

        setFiles(prev => prev.map((f, idx) =>
          idx === i ? { ...f, status: 'complete', key: result.key } : f
        ));
      } catch (err) {
        setFiles(prev => prev.map((f, idx) =>
          idx === i ? { ...f, status: 'error', error: 'Upload failed' } : f
        ));
      }
    }

    return results;
  };

  const reset = () => setFiles([]);

  return { files, uploadAll, reset };
}
```
