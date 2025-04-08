# Authentication & Storage Guide

This guide covers authentication implementation and file storage solutions for TrackMyJob.

## Table of Contents

- [Authentication](#authentication)
- [File Storage](#file-storage)
- [Security Best Practices](#security-best-practices)

## Authentication

We use [Supabase Auth](https://supabase.com/docs/guides/auth) for authentication, which provides a secure, feature-rich authentication system with multiple providers.

### 1. Setup

```bash
# Install dependencies
npm install @supabase/supabase-js
```

### 2. Configuration

```typescript
// libs/shared/auth/src/lib/supabase.config.ts
import { createClient } from "@supabase/supabase-js";

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

### 3. Web Implementation

```typescript
// apps/web/app/providers.tsx
import { createBrowserSupabaseClient } from "@supabase/auth-helpers-nextjs";
import { SessionContextProvider } from "@supabase/auth-helpers-react";

export function Providers({ children }) {
  const [supabaseClient] = useState(() => createBrowserSupabaseClient());

  return (
    <SessionContextProvider supabaseClient={supabaseClient}>
      {children}
    </SessionContextProvider>
  );
}

// apps/web/app/auth/page.tsx
import { Auth } from "@supabase/auth-ui-react";
import { ThemeSupa } from "@supabase/auth-ui-shared";

export default function AuthPage() {
  return (
    <Auth
      supabaseClient={supabase}
      appearance={{ theme: ThemeSupa }}
      providers={["google", "github"]}
    />
  );
}
```

### 4. Mobile Implementation

```typescript
// apps/mobile/src/providers/AuthProvider.tsx
import { createClient } from "@supabase/supabase-js";
import { SessionContextProvider } from "@supabase/auth-helpers-react";

export function AuthProvider({ children }) {
  return (
    <SessionContextProvider supabaseClient={supabase}>
      {children}
    </SessionContextProvider>
  );
}

// apps/mobile/src/screens/AuthScreen.tsx
import { Auth } from "@supabase/auth-ui-react";
import { ThemeSupa } from "@supabase/auth-ui-shared";

export function AuthScreen() {
  return (
    <Auth
      supabaseClient={supabase}
      appearance={{ theme: ThemeSupa }}
      providers={["google", "github"]}
    />
  );
}
```

### 5. Protected Routes

```typescript
// libs/shared/auth/src/lib/protected-route.tsx
import { useSession, useSupabaseClient } from "@supabase/auth-helpers-react";
import { useRouter } from "next/navigation";

export function ProtectedRoute({ children }) {
  const session = useSession();
  const router = useRouter();

  if (!session) {
    router.push("/auth");
    return null;
  }

  return children;
}
```

### 6. API Authentication

```typescript
// apps/api/src/middleware/auth.middleware.ts
import { createClient } from "@supabase/supabase-js";
import { Request, Response, NextFunction } from "express";

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function authMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const token = req.headers.authorization?.split(" ")[1];
    if (!token) {
      return res.status(401).json({ message: "No token provided" });
    }

    const {
      data: { user },
      error,
    } = await supabase.auth.getUser(token);

    if (error || !user) {
      return res.status(401).json({ message: "Invalid token" });
    }

    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({ message: "Authentication failed" });
  }
}
```

## File Storage

We use [Supabase Storage](https://supabase.com/docs/guides/storage) for file storage, particularly for resumes and profile pictures.

### 1. Setup

Supabase Storage is already included with our Supabase setup.

### 2. Configuration

Create a bucket for file storage in your Supabase dashboard:

1. Go to Storage in Supabase Dashboard
2. Create a new bucket called "resumes"
3. Set the bucket's privacy settings
4. Configure CORS if needed

### 3. File Upload Service

```typescript
// libs/shared/storage/src/lib/storage.service.ts
import { supabase } from "./supabase.config";

export class StorageService {
  static async uploadFile(file: File, bucket: string, path: string) {
    try {
      const { data, error } = await supabase.storage
        .from(bucket)
        .upload(path, file);

      if (error) throw error;

      const {
        data: { publicUrl },
      } = supabase.storage.from(bucket).getPublicUrl(path);

      return {
        url: publicUrl,
        path: path,
      };
    } catch (error) {
      throw new Error("File upload failed");
    }
  }

  static async deleteFile(bucket: string, path: string) {
    try {
      const { error } = await supabase.storage.from(bucket).remove([path]);

      if (error) throw error;
    } catch (error) {
      throw new Error("File deletion failed");
    }
  }
}
```

### 4. File Upload Components

```typescript
// libs/shared/ui/src/lib/file-upload/FileUpload.tsx
import { useState } from "react";
import { StorageService } from "@trackmyjob/shared/storage";

interface FileUploadProps {
  onUpload: (url: string) => void;
  allowedTypes: string[];
  maxSize: number; // in bytes
  bucket: string;
}

export function FileUpload({
  onUpload,
  allowedTypes,
  maxSize,
  bucket,
}: FileUploadProps) {
  const [uploading, setUploading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleUpload = async (file: File) => {
    try {
      setUploading(true);
      setError(null);

      if (!allowedTypes.includes(file.type)) {
        throw new Error("File type not supported");
      }

      if (file.size > maxSize) {
        throw new Error("File size too large");
      }

      const path = `${Date.now()}-${file.name}`;
      const result = await StorageService.uploadFile(file, bucket, path);

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
        accept={allowedTypes.join(",")}
        onChange={(e) => e.target.files?.[0] && handleUpload(e.target.files[0])}
        disabled={uploading}
      />
      {error && <p className="text-red-500">{error}</p>}
      {uploading && <p>Uploading...</p>}
    </div>
  );
}
```

## Security Best Practices

1. **Authentication**

   - Use secure password policies
   - Implement MFA when available
   - Set appropriate session timeouts
   - Use secure token storage

2. **File Storage**

   - Validate file types and sizes
   - Scan for malware
   - Use unique file names
   - Set appropriate bucket policies
   - Implement file access controls

3. **General**
   - Keep dependencies updated
   - Use environment variables for sensitive data
   - Implement rate limiting
   - Monitor for suspicious activities
   - Regular security audits
