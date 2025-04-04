# Technical Decisions Documentation

This document outlines the technical decisions, architecture, and setup instructions for the TrackMyJob project.

## 🏗️ Architecture Decisions

### Monorepo Structure (Nx)

We chose Nx monorepo for the following reasons:
1. **Code Sharing**: Efficient sharing of code between web and mobile applications
   - Types/interfaces for job applications
   - API calls and data fetching logic
   - Business logic
   - Utility functions
   - State management

2. **Development Benefits**:
   - Single command to run multiple apps
   - Automatic dependency graph management
   - Smart rebuilds (only what changed)
   - Consistent code style and linting
   - Shared testing utilities

### Project Structure
```
trackmyjob/
├── apps/
│   ├── web/                    # Next.js web app
│   ├── mobile/                 # React Native app
│   └── api/                    # Express backend
├── libs/
│   ├── shared/
│   │   ├── types/             # Shared TypeScript interfaces
│   │   ├── ui/                # Shared UI components
│   │   ├── utils/             # Shared utilities
│   │   ├── api/               # Shared API calls
│   │   └── state/             # Shared state management
│   ├── web/
│   │   ├── components/        # Web-specific components
│   │   └── features/          # Web-specific features
│   └── mobile/
│       ├── components/        # Mobile-specific components
│       └── features/          # Mobile-specific features
```

## 🛠️ Technical Stack Decisions

### Frontend (Web)
- **Next.js**: Chosen for:
  - Server-side rendering capabilities
  - Built-in API routes
  - Excellent TypeScript support
  - Strong SEO capabilities
  - Fast development and production performance

- **Tailwind CSS**: Selected for:
  - Utility-first approach
  - Easy customization
  - Good performance
  - Extensive documentation
  - Large community

### Mobile
- **React Native**: Chosen for:
  - Code sharing with web platform
  - Native performance
  - Large ecosystem
  - Strong community support

### Backend
- **Express.js**: Selected for:
  - Lightweight and flexible
  - Easy integration with TypeScript
  - Large ecosystem
  - Simple to extend

### Database
- **Neon PostgreSQL**: Chosen for:
  - Serverless PostgreSQL
  - Built-in scalability
  - Strong data consistency
  - Advanced querying capabilities

### State Management
- **Zustand**: Selected over Redux for:
  - Simpler learning curve
  - Less boilerplate
  - Easy TypeScript integration
  - Small bundle size
  - Works well in both web and mobile

### Data Fetching
- **React Query**: Chosen for:
  - Powerful caching
  - Server state management
  - Real-time updates
  - Works in both web and mobile
  - TypeScript support

## 📝 Setup Instructions

### 1. Initial Nx Workspace Setup

```bash
# Create a new Nx workspace
npx create-nx-workspace@latest trackmyjob \
  --preset=npm \
  --nx-cloud=false \
  --packageManager=npm
```

[View complete setup instructions](./SETUP_GUIDE.md)

### 2. Development Environment Setup

Required tools and versions:
- Node.js >= 16.x
- npm >= 8.x
- Git >= 2.x
- Xcode (for iOS development)
- Android Studio (for Android development)

### 3. Environment Variables

```env
# Web
NEXT_PUBLIC_API_URL=
NEXT_PUBLIC_RAPIDAPI_KEY=

# Mobile
API_URL=
RAPIDAPI_KEY=

# Backend
DATABASE_URL=
JWT_SECRET=
```

## 🔄 Development Workflow

### Code Sharing Strategy

1. **Shared Code Principles**:
   - Place shared business logic in shared libraries
   - Use TypeScript for type safety across platforms
   - Keep platform-specific code in respective apps

2. **Example of Shared Feature**:
   ```typescript
   // libs/shared/features/job-tracking/
   export const useJobApplication = (jobId: string) => {
     // Shared tracking logic that works in both platforms
   };
   ```

### Testing Strategy

1. **Unit Tests**:
   - Jest for testing logic
   - React Testing Library for components
   - Shared test utilities

2. **E2E Tests**:
   - Cypress for web
   - Detox for mobile

### CI/CD Pipeline

1. **GitHub Actions Workflow**:
   - Lint and test on PR
   - Build check on PR
   - Automatic deployment on merge to main

2. **Deployment Strategy**:
   - Web: Vercel
   - Mobile: App stores
   - API: Cloud hosting (e.g., Railway)

## 📚 Additional Resources

- [Nx Documentation](https://nx.dev/getting-started/intro)
- [Next.js Documentation](https://nextjs.org/docs)
- [React Native Documentation](https://reactnative.dev/docs/getting-started)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
- [React Query Documentation](https://tanstack.com/query/latest)