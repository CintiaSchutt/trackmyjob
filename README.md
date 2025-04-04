# TrackMyJob

A comprehensive job application tracking platform built with Next.js (web) and React Native (mobile).

## 📱 Features

- Track job applications status
- Manage application deadlines
- Store important job details
- Monitor interview processes
- Keep notes for each application
- AI-powered resume and cover letter customization
- Job search integration via RapidAPI

## 🚀 Getting Started

### Prerequisites

- Node.js (version 16 or higher)
- npm or yarn
- React Native development environment setup
- iOS Simulator (for Mac users) or Android Emulator
- Git

### Nx Monorepo Setup Guide

#### 1. Create Nx Workspace

```bash
# Create a new Nx workspace
npx create-nx-workspace@latest trackmyjob \
  --preset=npm \
  --nx-cloud=false \
  --packageManager=npm

# Navigate to project directory
cd trackmyjob
```

#### 2. Install Required Dependencies

```bash
# Install Nx plugins and core dependencies
npm install -D @nx/react @nx/next @nx/react-native @nx/node @nx/express
npm install -D tailwindcss postcss autoprefixer
npm install -D @nx/js @nx/workspace
```

#### 3. Create Applications

```bash
# Create Next.js web application
nx g @nx/next:app web \
  --directory=apps/web \
  --style=css \
  --routing=true

# Create React Native application
nx g @nx/react-native:app mobile \
  --directory=apps/mobile \
  --routing=true

# Create Express backend
nx g @nx/express:app api \
  --directory=apps/api \
  --frontendProject=web
```

#### 4. Create Shared Libraries

```bash
# Create shared types library
nx g @nx/js:lib types \
  --directory=libs/shared/types \
  --bundler=none

# Create shared UI components library
nx g @nx/react:lib ui \
  --directory=libs/shared/ui \
  --bundler=vite

# Create shared utilities library
nx g @nx/js:lib utils \
  --directory=libs/shared/utils \
  --bundler=none

# Create shared API interfaces library
nx g @nx/js:lib api \
  --directory=libs/shared/api \
  --bundler=none

# Create shared state management library
nx g @nx/js:lib state \
  --directory=libs/shared/state \
  --bundler=none
```

#### 5. Configure Tailwind CSS (Web)

```bash
# Initialize Tailwind CSS
cd apps/web
npx tailwindcss init -p

# Update tailwind.config.js
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
    '../../libs/shared/ui/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

#### 6. Set Up Development Environment

```bash
# Install additional dependencies
npm install @tanstack/react-query zustand axios

# Install development dependencies
npm install -D @testing-library/react @testing-library/react-native jest
```

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

### Running the Applications

```bash
# Start web application
nx serve web

# Start mobile application
nx serve mobile

# Start backend
nx serve api

# Run multiple applications
nx run-many --target=serve --projects=web,api
```

### Development Workflow

1. Create new features in shared libraries when they can be used by both platforms
2. Use platform-specific libraries for features that are unique to web or mobile
3. Use the `@trackmyjob` namespace to import shared code:

```typescript
import { Job } from '@trackmyjob/shared/types';
import { useJobs } from '@trackmyjob/shared/state';
import { formatDate } from '@trackmyjob/shared/utils';
```

## 🛠️ Built With

- Next.js & React Native
- TypeScript
- Tailwind CSS
- Zustand for state management
- React Query for data fetching
- Express.js backend
- Neon PostgreSQL
- GitHub Actions for CI/CD

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👩‍💻 Author

**Cintia Schutt**

* Github: [@cintiaschutt](https://github.com/cintiaschutt)

## ⭐️ Show your support

Give a ⭐️ if this project interests you!