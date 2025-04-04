# Setup Guide

This document provides detailed instructions for setting up the TrackMyJob development environment.

## Initial Setup

### 1. Create Nx Workspace

```bash
# Create a new Nx workspace
npx create-nx-workspace@latest trackmyjob \
  --preset=npm \
  --nx-cloud=false \
  --packageManager=npm

# Navigate to project directory
cd trackmyjob
```

### 2. Install Dependencies

```bash
# Install Nx plugins and core dependencies
npm install -D @nx/react @nx/next @nx/react-native @nx/node @nx/express
npm install -D tailwindcss postcss autoprefixer
npm install -D @nx/js @nx/workspace

# Install project dependencies
npm install @tanstack/react-query zustand axios
npm install @testing-library/react @testing-library/react-native jest --save-dev
```

### 3. Create Applications

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

### 4. Create Shared Libraries

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

### 5. Configure Tailwind CSS

```bash
# Navigate to web app directory
cd apps/web

# Initialize Tailwind CSS
npx tailwindcss init -p

# Update tailwind.config.js content
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

# Create global CSS file
echo "@tailwind base;
@tailwind components;
@tailwind utilities;" > styles/globals.css
```

### 6. Configure Environment Variables

Create `.env` files for each application:

```bash
# Web (.env.local in apps/web)
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_RAPIDAPI_KEY=your_key_here

# Mobile (.env in apps/mobile)
API_URL=http://localhost:3001
RAPIDAPI_KEY=your_key_here

# API (.env in apps/api)
DATABASE_URL=your_neon_db_url
JWT_SECRET=your_secret_here
```

## Running the Applications

### Development Mode

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

### Building for Production

```bash
# Build web application
nx build web

# Build mobile application
nx build mobile

# Build backend
nx build api

# Build all applications
nx run-many --target=build --projects=web,mobile,api
```

## Testing

```bash
# Run tests for web
nx test web

# Run tests for mobile
nx test mobile

# Run tests for api
nx test api

# Run all tests
nx run-many --target=test --projects=web,mobile,api
```

## Linting

```bash
# Lint web application
nx lint web

# Lint mobile application
nx lint mobile

# Lint api
nx lint api

# Lint all applications
nx run-many --target=lint --projects=web,mobile,api
```

## Common Issues and Solutions

### React Native Setup

If you encounter issues with React Native setup:

1. Make sure you have Xcode installed (Mac only)
```bash
xcode-select --install
```

2. Install CocoaPods (Mac only)
```bash
sudo gem install cocoapods
```

3. Install Android Studio and configure environment variables (All platforms)

### Nx Cache Issues

If you encounter Nx cache issues:

```bash
# Clear Nx cache
nx reset

# Rebuild all applications
nx run-many --target=build --projects=web,mobile,api --skip-nx-cache
```

### Dependency Issues

If you encounter peer dependency issues:

```bash
# Clean install with legacy peer deps
rm -rf node_modules package-lock.json
npm install --legacy-peer-deps
```

## Additional Setup

### Git Hooks (Optional)

1. Install husky
```bash
npm install husky --save-dev
npx husky install
```

2. Add pre-commit hook
```bash
npx husky add .husky/pre-commit "nx affected:lint && nx affected:test"
```

### VS Code Extensions (Recommended)

- Nx Console
- ESLint
- Prettier
- TypeScript and JavaScript Language Features
- Tailwind CSS IntelliSense

## Next Steps

After completing the setup:

1. Configure authentication
2. Set up database schema
3. Create initial components
4. Set up CI/CD pipeline

For more details on these steps, refer to their respective documentation in the `docs` folder.