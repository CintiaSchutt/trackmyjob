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

## 🚀 Tech Stack

### Frontend (Web)
- Next.js with TypeScript
- Tailwind CSS for styling
- React Query for data fetching
- Zustand for state management

### Mobile
- React Native with TypeScript
- React Native Paper
- Same state management and data fetching as web

### Backend
- Node.js with Express
- TypeScript
- Neon PostgreSQL

### Development
- Nx Monorepo
- ESLint + Prettier
- Jest for testing
- GitHub Actions for CI/CD

## 📂 Project Structure

```
trackmyjob/
├── apps/
│   ├── web/                    # Next.js web app
│   ├── mobile/                 # React Native app
│   └── api/                    # Express backend
├── libs/
│   ├── shared/                 # Shared libraries
│   ├── web/                    # Web-specific code
│   └── mobile/                 # Mobile-specific code
└── docs/                       # Documentation
```

## 📖 Documentation

- [Technical Decisions](docs/TECHNICAL_DECISIONS.md)
- [Setup Guide](docs/SETUP_GUIDE.md)

## 🏁 Getting Started

1. Clone the repository
```bash
git clone https://github.com/cintiaschutt/trackmyjob.git
cd trackmyjob
```

2. Install dependencies
```bash
npm install
```

3. Set up environment variables
```bash
# Copy example env files
cp .env.example .env
```

4. Start development servers
```bash
# Web + API
npm run dev

# Mobile
npm run mobile
```

For detailed setup instructions, see our [Setup Guide](docs/SETUP_GUIDE.md).

## 🧪 Testing

```bash
# Run all tests
npm test

# Run specific tests
npm test web
npm test mobile
npm test api
```

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👩‍💻 Author

**Cintia Schutt**

* Github: [@cintiaschutt](https://github.com/cintiaschutt)

## ⭐️ Show your support

Give a ⭐️ if this project interests you!