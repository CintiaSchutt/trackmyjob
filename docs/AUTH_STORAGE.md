# Authentication & Storage Guide

This guide covers authentication implementation and file storage solutions for TrackMyJob.

## Table of Contents
- [Authentication](#authentication)
- [File Storage](#file-storage)
- [Security Best Practices](#security-best-practices)

## Authentication

We use [Stack Auth](https://stackframe.co) for authentication, which provides a secure, feature-rich authentication system.

### 1. Setup

```bash
# Install dependencies
npm install @stackframe/stack
npx @stackframe/init-stack@2.7.25 . --no-browser
```

### 2. Configuration

```typescript
// libs/shared/auth/src/lib/auth.config.ts
export const authConfig = {
  projectId: process.env.NEXT_PUBLIC_STACK_PROJECT_ID,
  publishableKey: process.env.NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY,
  secretKey: process.env.STACK_SECRET_SERVER_KEY,
};
```

### 3. Web Implementation

```typescript
// apps/web/app/providers.tsx
import { StackProvider, StackTheme } from '@stackframe/stack';
import { authConfig } from '@trackmyjob/shared/auth';

export function Providers({ children }) {
  return (
    <StackProvider config={authConfig}>
      <StackTheme>{children}</StackTheme>
    </StackProvider>
  );
}

// apps/web/app/auth/page.tsx
import { SignIn } from '@stackframe/stack';

export default function AuthPage() {
  return <SignIn />;
}
```

### 4. Mobile Implementation

```typescript
// apps/mobile/src/providers/AuthProvider.tsx
import { StackProvider, StackTheme } from '@stackframe/stack/native';
import { authConfig } from '@trackmyjob/shared/auth';

export function AuthProvider({ children }) {
  return (
    <StackProvider config={authConfig}>
      <StackTheme>{children}</StackTheme>
    </StackProvider>
  );
}

// apps/mobile/src/screens/AuthScreen.tsx
import { SignIn } from '@stackframe/stack/native';

export function AuthScreen() {
  return <SignIn />;
}
```

### 5. Protected Routes

```typescript
// libs/shared/auth/src/lib/protected-route.tsx
import { useUser } from '@stackframe/stack';
import { useRouter } from 'next/navigation';

export function ProtectedRoute({ children }) {
  const user = useUser({ or: 'redirect' });
  const router = useRouter();

  if (!user) {
    router.push('/auth');
    return null;
  }

  return children;
}
```

### 6. API Authentication

```typescript
// apps/api/src/middleware/auth.middleware.ts
import { stackServerApp } from '@trackmyjob/shared/auth';
import { Request, Response, NextFunction } from 'express';

export async function authMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) {
      return res.status(401).json({ message: 'No token provided' });
    }

    const user = await stackServerApp.getUser(token);
    if (!user) {
      return res.status(401).json({ message: 'Invalid token' });
    }

    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({ message: 'Authentication failed' });
  }
}
```

## File Storage

We use [Cloudinary](https://cloudinary.com) for file storage, particularly for resumes and profile pictures.

### 1. Setup

```bash
# Install dependencies
npm install cloudinary @cloudinary/url-gen
```

### 2. Configuration

```typescript
// libs/shared/storage/src/lib/storage.config.ts
import { v2 as cloudinary } from 'cloudinary';

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

export { cloudinary };
```

### 3. File Upload Service

```typescript
// libs/shared/storage/src/lib/storage.service.ts
import { cloudinary } from './storage.config';

export class StorageService {
  static async uploadFile(
    file: Buffer,
    folder: string,
    allowedTypes: string[]
  ) {
    try {
      const result = await cloudinary.uploader.upload(file, {
        folder,
        resource_type: 'auto',
        allowed_formats: allowedTypes,
      });

      return {
        url: result.secure_url,
        publicId: result.public_id,
      };
    } catch (error) {
      throw new Error('File upload failed');
    }
  }

  static async deleteFile(publicId: string) {
    try {
      await cloudinary.uploader.destroy(publicId);
    } catch (error) {
      throw new Error('File deletion failed');
    }
  }
}
```

### 4. File Upload Components

```typescript
// libs/shared/ui/src/lib/file-upload/FileUpload.tsx
import { useState } from 'react';
import { StorageService } from '@trackmyjob/shared/storage';

interface FileUploadProps {
  onUpload: (url: string) => void;
  allowedTypes: string[];
  maxSize: number; // in bytes
}

export function FileUpload({ onUpload, allowedTypes, maxSize }: FileUploadProps) {
  const [uploading, setUploading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleUpload = async (file: File) => {
    try {
      setUploading(true);
      setError(null);

      if (!allowedTypes.includes(file.type)) {
        throw new Error('File type not supported');
      }

      if (file.size > maxSize) {
        throw new Error('File size too large');
      }

      const buffer = await file.arrayBuffer();
      const result = await StorageService.uploadFile(
        Buffer.from(buffer),
        'resumes',
        allowedTypes
      );

      onUpload(result.url);
    } catch (error) {
      setError(error.message);
    } finally {
      setUploading(false);
    }
  };

  return (
    <div>
      <input
        type="file"
        accept={allowedTypes.join(',')}
        onChange={(e) => e.target.files?.[0] && handleUpload(e.target.files[0])}
        disabled={uploading}
      />
      {uploading && <p>Uploading...</p>}
      {error && <p className="text-red-500">{error}</p>}
    </div>
  );
}
```

## Security Best Practices

### 1. File Upload Security

- Validate file types on both client and server
- Set maximum file size limits
- Scan files for malware
- Use signed upload URLs
- Store file metadata in database

### 2. Authentication Security

- Use secure session management
- Implement rate limiting
- Enable two-factor authentication
- Monitor suspicious activities
- Regular security audits

### 3. Environment Variables

Required environment variables for auth and storage:

```env
# Auth
NEXT_PUBLIC_STACK_PROJECT_ID=your_project_id
NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY=your_publishable_key
STACK_SECRET_SERVER_KEY=your_secret_key

# Storage
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

### 4. Database Schema

```sql
-- User Files Table
CREATE TABLE user_files (
  id SERIAL PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id),
  file_url TEXT NOT NULL,
  public_id TEXT NOT NULL,
  file_type TEXT NOT NULL,
  file_size INTEGER NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Create index for faster queries
CREATE INDEX idx_user_files_user_id ON user_files(user_id);
```