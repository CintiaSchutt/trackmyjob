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

## Railway Deployment Guide

Railway is a modern deployment platform that simplifies the deployment and management of full-stack applications. This section provides a comprehensive guide for deploying and managing the TrackMyJob API on Railway.

### Why Railway?

Railway offers several advantages for our backend deployment:

1. **Zero Configuration**

   - Automatic framework detection
   - Built-in support for Node.js and Express
   - Automatic HTTPS and SSL certificate management

2. **Database Integration**

   - Native PostgreSQL support
   - Seamless integration with Neon
   - Automatic backups and monitoring

3. **Developer Experience**

   - Real-time logs and metrics
   - Easy environment variable management
   - Instant rollbacks
   - Automatic branch deployments

4. **Scaling & Performance**
   - Automatic horizontal scaling
   - Global CDN
   - DDoS protection
   - Zero-downtime deployments

### Setup Process

1. **Initial Setup**

```bash
# Install Railway CLI
npm i -g @railway/cli

# Login to Railway
railway login

# Initialize in your project
railway init

# Link to existing project
railway link
```

2. **Project Configuration**

Create a `railway.toml` file in your project root:

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
healthcheck = "/health"
port = "3000"

[database]
name = "neondb"
```

3. **Environment Variables**

Configure these in Railway Dashboard:

```env
NODE_ENV=production
DATABASE_URL=your_neon_db_url
JWT_SECRET=your_secret
API_KEY=your_api_key
PORT=3000
```

### Deployment Workflow

1. **Manual Deployment**

```bash
# Deploy current branch
railway up

# Deploy specific service
railway up --service api

# Deploy with environment variables
railway up --env production
```

2. **Automatic Deployments**

Configure automatic deployments in Railway Dashboard:

- Connect your GitHub repository
- Enable automatic deployments
- Configure branch deployments

### CI/CD Integration

1. **GitHub Actions Integration**

Add this to your GitHub Actions workflow:

```yaml
name: Railway Deployment
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"

      - name: Install Dependencies
        run: npm ci

      - name: Build
        run: npm run build:api

      - name: Run Tests
        run: npm test:api

      - name: Install Railway CLI
        run: npm i -g @railway/cli

      - name: Deploy to Railway
        run: railway up
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

2. **Required Secrets**

Add these secrets to your GitHub repository:

- `RAILWAY_TOKEN`: Your Railway API token
- `RAILWAY_PROJECT_ID`: Your Railway project ID

### Monitoring & Logging

1. **Application Logs**

Access logs through:

- Railway Dashboard
- CLI command: `railway logs`
- Integrated log forwarding to external services

2. **Metrics & Analytics**

Monitor:

- CPU usage
- Memory consumption
- Network traffic
- Response times
- Error rates

3. **Alerts & Notifications**

Configure alerts for:

- Service downtime
- High error rates
- Performance degradation
- Resource usage thresholds

### Best Practices

1. **Development Workflow**

   - Use feature branches for development
   - Set up preview environments for PRs
   - Implement proper testing before deployment

2. **Security**

   - Store sensitive data in environment variables
   - Use Railway's secret management
   - Implement proper access controls
   - Regular security audits

3. **Performance**

   - Optimize build process
   - Implement caching strategies
   - Monitor resource usage
   - Regular performance testing

4. **Maintenance**
   - Regular dependency updates
   - Scheduled maintenance windows
   - Backup strategy
   - Disaster recovery plan

### Troubleshooting

Common issues and solutions:

1. **Deployment Failures**

   - Check build logs
   - Verify environment variables
   - Validate dependencies
   - Check resource limits

2. **Performance Issues**

   - Monitor resource usage
   - Check database connections
   - Analyze request patterns
   - Review caching strategy

3. **Connection Issues**
   - Verify network settings
   - Check DNS configuration
   - Validate SSL certificates
   - Review firewall rules

### Scaling Guidelines

1. **Vertical Scaling**

   - Increase compute resources
   - Optimize memory usage
   - Upgrade database tier

2. **Horizontal Scaling**
   - Enable auto-scaling
   - Configure load balancing
   - Implement caching
   - Optimize database queries

### Cost Management

1. **Resource Optimization**

   - Monitor usage patterns
   - Implement auto-scaling
   - Use appropriate instance sizes
   - Regular cost analysis

2. **Development vs Production**
   - Use development environments efficiently
   - Implement proper staging environments
   - Control preview environment usage

### Migration Guide

If migrating from another platform:

1. **Preparation**

   - Backup existing data
   - Document current configuration
   - Plan maintenance window
   - Prepare rollback strategy

2. **Migration Steps**

   - Set up Railway project
   - Configure environment
   - Migrate database
   - Update DNS records
   - Verify functionality

3. **Post-Migration**
   - Monitor performance
   - Verify all features
   - Update documentation
   - Train team members

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
