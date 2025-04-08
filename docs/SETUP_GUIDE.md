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
npm install -D @nx/js @nx/workspace

# Install styling dependencies
npm install -D tailwindcss postcss autoprefixer # For web only
npm install react-native-paper react-native-safe-area-context react-native-vector-icons
npm install react-native-paper-dates # Optional, for date inputs

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

### 5. Configure Styling

#### For Web (Next.js with Tailwind)

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

#### For Mobile (React Native Paper)

```bash
# Navigate to mobile app directory
cd apps/mobile

# Update babel.config.js to include react-native-vector-icons
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
};

# For iOS, install pods
cd ios && pod install && cd ..
```

Create a theme configuration file:

```tsx
// apps/mobile/src/theme.ts
import { MD3LightTheme, configureFonts } from "react-native-paper";

export const theme = {
  ...MD3LightTheme,
  colors: {
    ...MD3LightTheme.colors,
    primary: "#2563eb", // blue-600
    secondary: "#4f46e5", // indigo-600
    // Add more custom colors as needed
  },
  fonts: configureFonts({
    config: {
      fontFamily: "System",
    },
  }),
};
```

Set up the Paper provider:

```tsx
// apps/mobile/src/App.tsx
import { PaperProvider } from "react-native-paper";
import { theme } from "./theme";

export default function App() {
  return <PaperProvider theme={theme}>{/* Your app content */}</PaperProvider>;
}
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

## Component Examples

### Mobile Components (React Native)

```tsx
// Example of a custom button component
import { StyleSheet } from "react-native";
import { Button } from "react-native-paper";

export function CustomButton({ onPress, children }) {
  return (
    <Button
      mode="contained"
      onPress={onPress}
      style={styles.button}
      labelStyle={styles.buttonText}
    >
      {children}
    </Button>
  );
}

const styles = StyleSheet.create({
  button: {
    borderRadius: 8,
    marginVertical: 8,
  },
  buttonText: {
    fontSize: 16,
    letterSpacing: 0.5,
  },
});

// Example of a card component
import { Card, Text } from "react-native-paper";

export function JobCard({ title, company, location }) {
  return (
    <Card style={styles.card}>
      <Card.Content>
        <Text variant="titleLarge" style={styles.title}>
          {title}
        </Text>
        <Text variant="bodyMedium">{company}</Text>
        <Text variant="bodySmall" style={styles.location}>
          {location}
        </Text>
      </Card.Content>
    </Card>
  );
}

const styles = StyleSheet.create({
  card: {
    marginVertical: 8,
    marginHorizontal: 16,
    elevation: 2,
  },
  title: {
    marginBottom: 8,
  },
  location: {
    marginTop: 4,
    color: "#666",
  },
});
```

### Web Components (Next.js)

```tsx
// Example of a button component
export function Button({ children, onClick }) {
  return (
    <button
      onClick={onClick}
      className="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
    >
      {children}
    </button>
  );
}

// Example of a card component
export function JobCard({ title, company, location }) {
  return (
    <div className="bg-white shadow-md rounded-lg p-6 mb-4">
      <h3 className="text-xl font-bold mb-2">{title}</h3>
      <p className="text-gray-700">{company}</p>
      <p className="text-gray-500 mt-1">{location}</p>
    </div>
  );
}
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

## Common Issues and Solutions

### React Native Paper Setup

1. If vector icons are not showing:

   - iOS: Make sure you've run `pod install`
   - Android: Add the following to `android/app/build.gradle`:
     ```gradle
     apply from: "../../node_modules/react-native-vector-icons/fonts.gradle"
     ```

2. If theme is not applying:
   - Check that PaperProvider is wrapping your app
   - Verify theme object structure

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
