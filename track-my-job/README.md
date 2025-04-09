# TrackMyJob

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-18.0-blue.svg)](https://reactjs.org/)
[![React Native](https://img.shields.io/badge/React_Native-0.72-blue.svg)](https://reactnative.dev/)
[![Next.js](https://img.shields.io/badge/Next.js-14.0-black.svg)](https://nextjs.org/)
[![Nx](https://img.shields.io/badge/Nx-17.0-blue.svg)](https://nx.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.0-38B2AC.svg)](https://tailwindcss.com/)
[![React Native Paper](https://img.shields.io/badge/React_Native_Paper-5.0-purple.svg)](https://callstack.github.io/react-native-paper/)

</div>

## 📱 About

TrackMyJob is a cross-platform application that helps you manage your job search process efficiently. Built with React Native and Next.js in a monorepo structure, it provides a seamless experience across web and mobile platforms.

### Features

- 📝 Track job applications
- 📅 Manage interviews
- 📎 Store resumes and cover letters
- 📊 View analytics and insights
- 🔔 Get reminders and notifications
- 🌐 Cross-platform synchronization
- 🔒 Secure data storage
- 📱 Offline support

## 🚀 Quick Start

### Prerequisites

- Node.js >= 16.x
- npm >= 8.x
- Git >= 2.x
- Xcode (for iOS development)
- Android Studio (for Android development)

### Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/cintiaschutt/trackmyjob.git
   cd trackmyjob
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Set up environment variables:

   ```bash
   cp .env.example .env.local
   ```

4. Start development servers:

   ```bash
   # Web application
   npm run dev:web

   # Mobile application
   npm run dev:mobile

   # API server
   npm run dev:api
   ```

For detailed setup instructions, see [SETUP_GUIDE.md](docs/SETUP_GUIDE.md).

## 📚 Documentation

- [Setup Guide](docs/SETUP_GUIDE.md)
- [Technical Decisions](docs/TECHNICAL_DECISIONS.md)
- [Testing Guide](docs/TESTING_GUIDE.md)
- [Infrastructure Guide](docs/INFRASTRUCTURE.md)
- [Deployment Guide](docs/DEPLOYMENT.md)
- [Database Guide](docs/DATABASE_GUIDE.md)
- [Authentication Guide](docs/AUTH_STORAGE.md)
- [FAQ](docs/FAQ.md)

## 🏗️ Architecture

TrackMyJob uses an Nx monorepo structure:

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

## 🛠️ Tech Stack

- **Frontend (Web)**

  - Next.js
  - Tailwind CSS
  - React Query
  - Zustand

- **Mobile**

  - React Native
  - React Native Paper
  - React Navigation
  - React Query

- **Backend**

  - Express.js
  - Neon PostgreSQL
  - Supabase Auth
  - Supabase Storage

- **DevOps**
  - GitHub Actions
  - Vercel
  - EAS (Expo Application Services)
  - Railway

## 🧪 Testing

```bash
# Run all tests
npm test

# Run web tests
npm run test:web

# Run mobile tests
npm run test:mobile

# Run API tests
npm run test:api

# Run E2E tests
npm run test:e2e
```

For detailed testing information, see [TESTING_GUIDE.md](docs/TESTING_GUIDE.md).

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Team

- **Cintia Schutt** - _Initial work_ - [GitHub](https://github.com/cintiaschutt)

## 🙏 Acknowledgments

- [Nx](https://nx.dev/) for the excellent monorepo tooling
- [React Native Paper](https://callstack.github.io/react-native-paper/) for the beautiful mobile components
- [Tailwind CSS](https://tailwindcss.com/) for the utility-first CSS framework
- [Supabase](https://supabase.com/) for authentication and storage
- [Neon](https://neon.tech/) for serverless PostgreSQL

## 📞 Support

- Check our [FAQ](docs/FAQ.md)
- Create an issue on GitHub
