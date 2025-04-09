# Database Guide

## Why Supabase Instead of Prisma?

### What is Prisma?

Prisma is an ORM (Object-Relational Mapping) that:

- Requires schema definition in a separate schema file
- Needs migrations to be generated and applied
- Requires a separate database connection
- Adds an extra layer between your app and database

### What is Supabase?

Supabase is a full backend solution that includes:

- PostgreSQL database
- Auto-generated TypeScript types
- Built-in authentication
- Real-time subscriptions
- Row Level Security
- File storage
- Database backups

## Using Supabase

### 1. Setup

```typescript
// libs/shared/database/src/lib/supabase.ts
import { createClient } from "@supabase/supabase-js";
import { Database } from "./types"; // Auto-generated types

export const supabase = createClient<Database>(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

### 2. Type Generation

1. Install Supabase CLI:

```bash
npm install supabase --save-dev
```

2. Generate types:

```bash
supabase gen types typescript --project-id your-project-id > libs/shared/database/src/lib/types.ts
```

### 3. Basic CRUD Operations

```typescript
// libs/shared/database/src/lib/job-applications.ts
import { supabase } from "./supabase";

export const jobApplications = {
  // Create
  create: async (data: JobApplication) => {
    const { data: newJob, error } = await supabase
      .from("job_applications")
      .insert(data)
      .select()
      .single();

    if (error) throw error;
    return newJob;
  },

  // Read
  getAll: async (userId: string) => {
    const { data, error } = await supabase
      .from("job_applications")
      .select("*")
      .eq("user_id", userId)
      .order("applied_date", { ascending: false });

    if (error) throw error;
    return data;
  },

  // Update
  update: async (id: string, data: Partial<JobApplication>) => {
    const { data: updatedJob, error } = await supabase
      .from("job_applications")
      .update(data)
      .eq("id", id)
      .select()
      .single();

    if (error) throw error;
    return updatedJob;
  },

  // Delete
  delete: async (id: string) => {
    const { error } = await supabase
      .from("job_applications")
      .delete()
      .eq("id", id);

    if (error) throw error;
  },

  // Real-time subscription
  subscribe: (userId: string, callback: (job: JobApplication) => void) => {
    const subscription = supabase
      .from("job_applications")
      .select("*")
      .eq("user_id", userId)
      .on("*", (payload) => {
        callback(payload.new);
      })
      .subscribe();

    return () => subscription.unsubscribe();
  },
};
```

### 4. Using in Components

```typescript
// apps/web/src/app/jobs/page.tsx
"use client";

import { useEffect, useState } from "react";
import { jobApplications } from "@trackmyjob/shared/database";
import { useUser } from "@supabase/auth-helpers-react";

export default function JobsPage() {
  const [jobs, setJobs] = useState<JobApplication[]>([]);
  const user = useUser();

  useEffect(() => {
    if (!user) return;

    // Load initial data
    jobApplications.getAll(user.id).then(setJobs);

    // Subscribe to real-time updates
    const unsubscribe = jobApplications.subscribe(user.id, (newJob) => {
      setJobs((prev) => [...prev, newJob]);
    });

    return unsubscribe;
  }, [user]);

  return (
    <div>
      {jobs.map((job) => (
        <JobCard key={job.id} job={job} />
      ))}
    </div>
  );
}
```

### 5. Complex Queries

```typescript
// Example of a more complex query
const getJobsWithStats = async (userId: string) => {
  const { data, error } = await supabase
    .from("job_applications")
    .select(
      `
      *,
      interviews (
        id,
        date,
        type,
        notes
      )
    `
    )
    .eq("user_id", userId)
    .order("applied_date", { ascending: false });

  if (error) throw error;
  return data;
};
```

## Benefits of Using Supabase Directly

1. **Simpler Setup**

   - No need for separate database connection
   - No migration files to manage
   - No extra ORM layer

2. **Real-time Built-in**

   - Subscribe to database changes
   - Perfect for collaborative features
   - No additional websocket setup needed

3. **Type Safety**

   - Auto-generated types from your database schema
   - Full TypeScript support
   - IDE autocompletion

4. **Security**

   - Row Level Security built-in
   - Policies defined in database
   - No need for extra authorization layer

5. **Performance**
   - Direct PostgreSQL access
   - No ORM overhead
   - Edge Functions for better latency

## Common Database Operations

### Filtering

```typescript
// Get jobs by status
const getJobsByStatus = async (userId: string, status: string) => {
  const { data, error } = await supabase
    .from("job_applications")
    .select("*")
    .eq("user_id", userId)
    .eq("status", status);

  if (error) throw error;
  return data;
};
```

### Full Text Search

```typescript
// Search jobs by company or position
const searchJobs = async (userId: string, query: string) => {
  const { data, error } = await supabase
    .from("job_applications")
    .select("*")
    .eq("user_id", userId)
    .or(`company.ilike.%${query}%,position.ilike.%${query}%`);

  if (error) throw error;
  return data;
};
```

### Aggregations

```typescript
// Get application statistics
const getStats = async (userId: string) => {
  const { data, error } = await supabase
    .from("job_applications")
    .select("status")
    .eq("user_id", userId);

  if (error) throw error;

  return data.reduce((acc, curr) => {
    acc[curr.status] = (acc[curr.status] || 0) + 1;
    return acc;
  }, {} as Record<string, number>);
};
```

This approach gives you:

- Simpler codebase
- Better performance
- Real-time capabilities
- Built-in security
- Type safety
- Less configuration
