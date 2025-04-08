# Deployment & CI/CD Guide

This guide covers deployment configurations and CI/CD setup for TrackMyJob.

## Table of Contents

- [Deployment Environments](#deployment-environments)
- [CI/CD Pipeline](#cicd-pipeline)
- [Environment Variables](#environment-variables)
- [Monitoring & Logging](#monitoring--logging)

## Deployment Environments

### Web (Next.js)

We use Vercel for web deployment:

1. **Setup**

```bash
# Install Vercel CLI
npm i -g vercel

# Login to Vercel
vercel login

# Deploy
vercel
```

2. **Configuration** (`vercel.json`)

```json
{
  "version": 2,
  "buildCommand": "nx build web",
  "outputDirectory": "dist/apps/web/.next",
  "framework": "nextjs",
  "env": {
    "NEXT_PUBLIC_API_URL": "@next_public_api_url",
    "NEXT_PUBLIC_RAPIDAPI_KEY": "@next_public_rapidapi_key"
  }
}
```

### Mobile (React Native)

We use EAS (Expo Application Services) for mobile deployment:

1. **Setup**

```bash
# Install EAS CLI
npm install -g eas-cli

# Login to EAS
eas login

# Configure EAS
eas build:configure
```

2. **Configuration** (`eas.json`)

```json
{
  "cli": {
    "version": ">= 3.13.3"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890",
        "appleTeamId": "ABCDEF1234"
      },
      "android": {
        "serviceAccountKeyPath": "./path-to-key.json",
        "track": "production"
      }
    }
  }
}
```

### Backend (Express)

We use Railway for backend deployment:

1. **Setup**

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login to Railway
railway login

# Initialize project
railway init

# Deploy
railway up
```

2. **Configuration** (`railway.toml`)

```toml
[build]
builder = "nixpacks"
buildCommand = "nx build api"

[deploy]
startCommand = "node dist/apps/api/main.js"
healthcheckPath = "/api/health"
healthcheckTimeout = 100
restartPolicyType = "on_failure"

[service]
name = "trackmyjob-api"
```

## CI/CD Pipeline

We use GitHub Actions for CI/CD:

```yaml
# .github/workflows/ci.yml
name: CI

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

  deploy-web:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: "--prod"

  deploy-api:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Railway CLI
        run: npm i -g @railway/cli

      - name: Deploy to Railway
        run: railway up
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}

  build-mobile:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup EAS
        uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - name: Build Android
        run: eas build --platform android --non-interactive

      - name: Build iOS
        run: eas build --platform ios --non-interactive
```

## Environment Variables

### Required Variables

1. **Web**

```env
NEXT_PUBLIC_API_URL=https://api.trackmyjob.com
NEXT_PUBLIC_RAPIDAPI_KEY=your_key
```

2. **Mobile**

```env
API_URL=https://api.trackmyjob.com
RAPIDAPI_KEY=your_key
```

3. **Backend**

```env
DATABASE_URL=your_neon_db_url
JWT_SECRET=your_secret
RAPIDAPI_KEY=your_key
```

### CI/CD Secrets

Required GitHub repository secrets:

- `VERCEL_TOKEN`
- `VERCEL_ORG_ID`
- `VERCEL_PROJECT_ID`
- `RAILWAY_TOKEN`
- `EXPO_TOKEN`
- `APPLE_APP_SPECIFIC_PASSWORD`
- `GOOGLE_PLAY_API_KEY`

## Monitoring & Logging

### Application Monitoring

We use [Sentry](https://sentry.io) for error tracking and monitoring:

1. **Web Setup**

```typescript
// apps/web/app/sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0,
});
```

2. **Mobile Setup**

```typescript
// apps/mobile/src/utils/sentry.ts
import * as Sentry from "@sentry/react-native";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 1.0,
});
```

3. **Backend Setup**

```typescript
// apps/api/src/main.ts
import * as Sentry from "@sentry/node";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 1.0,
});
```

### Performance Monitoring

We use [New Relic](https://newrelic.com) for performance monitoring:

```typescript
// apps/api/src/utils/newrelic.ts
require("newrelic");

export const newrelicConfig = {
  appName: ["TrackMyJob API"],
  licenseKey: process.env.NEW_RELIC_LICENSE_KEY,
  logging: {
    level: "info",
  },
};
```

### Health Checks

```typescript
// apps/api/src/health/health.controller.ts
import { Controller, Get } from "@nestjs/common";
import { HealthCheck, HealthCheckService } from "@nestjs/terminus";

@Controller("health")
export class HealthController {
  constructor(private health: HealthCheckService) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([]);
  }
}
```
