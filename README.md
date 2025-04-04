# TrackMyJob

A comprehensive job search and application tracking platform built with React (web) and React Native (mobile). This project serves as a learning journey from Angular to React/React Native with TypeScript.

## 🎯 Vision

TrackMyJob aims to be your all-in-one job search companion, helping you:
- Search and discover job opportunities through RapidAPI integration
- Tailor your applications with AI-powered resume and cover letter customization
- Track your job applications systematically
- Manage your job search process efficiently

## 📱 Key Features

### Job Search
- Centralized job search interface integrated with RapidAPI
- Save jobs to favorites
- Detailed job viewing and analysis

### AI-Powered Application Tools
- Smart resume tailoring based on job descriptions
- Cover letter customization assistance
- Job description analysis and keyword extraction

### Application Tracking
- Comprehensive application status management:
  - Company and position details
  - Work type (Remote/On-site/Hybrid)
  - Application stages (Interested/Applied/Interviewed/Rejected)
  - Referral tracking
  - Follow-up message management
  - Important dates
  - Salary information
  - Document management (Resumes/Cover Letters)
  - Company information and website

## 🛠️ Tech Stack

### Frontend
- React (Web Application)
- React Native (Mobile Application)
- TypeScript
- Modern UI/UX libraries (to be determined)

### Backend
- Node.js
- Neon PostgreSQL
- RESTful API

### Integration & Services
- RapidAPI for job listings
- AI services for document customization
- GitHub Actions for CI/CD

### Development Tools
- GitHub for:
  - Version Control
  - Project Management (Issues & Projects)
  - CI/CD (GitHub Actions)
  - Deployment

## 🚀 Getting Started

### Prerequisites

- Node.js (version 16 or higher)
- npm or yarn
- Git
- PostgreSQL (for local development)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/cintiaschutt/trackmyjob.git
```

2. Navigate to the project directory:
```bash
cd trackmyjob
```

3. Install dependencies:
```bash
npm install
# or
yarn install
```

4. Set up environment variables:
```bash
cp .env.example .env
```
Then edit `.env` with your configuration

5. Start the development server:
```bash
# For web
npm run dev
# or
yarn dev

# For mobile
npm run start
# or
yarn start
```

## 📱 Mobile Development

For React Native development:

```bash
# Run on iOS
npm run ios
# or
yarn ios

# Run on Android
npm run android
# or
yarn android
```

## 🗃️ Project Structure

```
trackmyjob/
├── web/                   # React web application
├── mobile/                # React Native mobile application
├── shared/                # Shared components and utilities
├── server/                # Backend API
└── docs/                  # Documentation
```

## 🤝 Contributing

This project is currently in development. Contributions, ideas, and feedback are welcome! Feel free to check the [issues page](https://github.com/cintiaschutt/trackmyjob/issues).

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👩‍💻 Author

**Cintia Schutt**

* Github: [@cintiaschutt](https://github.com/cintiaschutt)

## ⭐️ Show your support

Give a ⭐️ if this project interests you!