# Infrastructure Guide

A simplified guide for TrackMyJob's infrastructure using industry-standard tools.

## Tech Stack

- **Authentication & Database**: Supabase
- **File Storage**: Supabase Storage
- **CI/CD**: GitHub Actions
- **Web Hosting**: Vercel
- **API Hosting**: Vercel (Serverless Functions)
- **Mobile**: Expo + EAS (Expo Application Services)

## Setup Guide

### 1. Supabase Setup

1. Create a new Supabase project at [supabase.com](https://supabase.com)

2. Install dependencies:

```bash
npm install @supabase/supabase-js
npm install @supabase/auth-helpers-nextjs # for web
npm install @supabase/auth-helpers-react-native # for mobile
```

3. Initialize Supabase client:

```typescript
// libs/shared/supabase/src/lib/supabase.ts
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### 2. Authentication

```typescript
// libs/shared/auth/src/lib/auth.ts
import { supabase } from "@trackmyjob/shared/supabase";

export const auth = {
  signUp: async (email: string, password: string) => {
    const { data, error } = await supabase.auth.signUp({
      email,
      password,
    });
    if (error) throw error;
    return data;
  },

  signIn: async (email: string, password: string) => {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });
    if (error) throw error;
    return data;
  },

  signOut: async () => {
    const { error } = await supabase.auth.signOut();
    if (error) throw error;
  },

  getUser: async () => {
    const {
      data: { user },
    } = await supabase.auth.getUser();
    return user;
  },
};
```

### 3. File Storage

```typescript
// libs/shared/storage/src/lib/storage.ts
import { supabase } from "@trackmyjob/shared/supabase";

export const storage = {
  uploadFile: async (bucket: string, path: string, file: File) => {
    const { data, error } = await supabase.storage
      .from(bucket)
      .upload(path, file);
    if (error) throw error;
    return data;
  },

  getFileUrl: (bucket: string, path: string) => {
    const { data } = supabase.storage.from(bucket).getPublicUrl(path);
    return data.publicUrl;
  },

  deleteFile: async (bucket: string, path: string) => {
    const { error } = await supabase.storage.from(bucket).remove([path]);
    if (error) throw error;
  },
};
```

### 4. Database Schema

```sql
-- Create tables in Supabase SQL editor

-- Users Profile (extends Supabase auth.users)
create table profiles (
  id uuid references auth.users on delete cascade,
  full_name text,
  avatar_url text,
  created_at timestamptz default now(),
  primary key (id)
);

-- Job Applications
create table job_applications (
  id uuid default uuid_generate_v4() primary key,
  user_id uuid references auth.users on delete cascade,
  company text not null,
  position text not null,
  status text not null,
  applied_date date not null,
  resume_url text,
  notes text,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Enable RLS (Row Level Security)
alter table profiles enable row level security;
alter table job_applications enable row level security;

-- Create policies
create policy "Users can view own profile"
  on profiles for select
  using (auth.uid() = id);

create policy "Users can update own profile"
  on profiles for update
  using (auth.uid() = id);

create policy "Users can view own applications"
  on job_applications for select
  using (auth.uid() = user_id);

create policy "Users can insert own applications"
  on job_applications for insert
  with check (auth.uid() = user_id);

create policy "Users can update own applications"
  on job_applications for update
  using (auth.uid() = user_id);

create policy "Users can delete own applications"
  on job_applications for delete
  using (auth.uid() = user_id);
```

### 5. CI/CD with GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Run lint
        run: npm run lint

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: "18"

      # Deploy to Vercel
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: "--prod"

      # Build mobile app with EAS
      - name: Setup EAS
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Build preview
        run: eas build --platform all --non-interactive --profile preview
```

### 6. Environment Variables

```env
# .env.example
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# Vercel
VERCEL_TOKEN=your_vercel_token
VERCEL_ORG_ID=your_org_id
VERCEL_PROJECT_ID=your_project_id

# Expo
EXPO_TOKEN=your_expo_token
```

## Development Workflow

1. **Local Development**

```bash
# Start web app
nx serve web

# Start mobile app
nx serve mobile
```

2. **Testing**

```bash
# Run all tests
nx run-many --target=test --all

# Run specific project tests
nx test web
nx test mobile
```

3. **Deployment**

- Push to `main` branch triggers automatic deployment
- Web app deploys to Vercel
- Mobile app builds on EAS
- Database migrations run through Supabase dashboard

## Best Practices

1. **Security**

   - Always use RLS policies in Supabase
   - Keep environment variables secure
   - Regular security audits through Supabase dashboard

2. **Performance**

   - Use Supabase edge functions for better performance
   - Enable CDN caching where appropriate
   - Monitor performance through Vercel analytics

3. **Monitoring**
   - Use Supabase dashboard for database monitoring
   - Vercel dashboard for web performance
   - Expo dashboard for mobile analytics

This setup provides:

- Simple, yet powerful authentication
- Scalable file storage
- Real-time database capabilities
- Automated deployments
- Easy monitoring and maintenance
