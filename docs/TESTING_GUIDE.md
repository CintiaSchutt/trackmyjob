# Testing Guide

This document outlines the testing strategy and practices for the TrackMyJob project.

## 📋 Testing Overview

Our testing strategy follows the Testing Trophy approach:

1. Static Testing (TypeScript, ESLint)
2. Unit Testing (Jest)
3. Integration Testing (React Testing Library)
4. End-to-End Testing (Cypress for web, Detox for mobile)

## 🔍 Static Testing

### TypeScript

- Strict mode enabled
- No implicit any
- Strict null checks
- Strict function types

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

### ESLint

Custom ESLint configuration extending recommended configs:

```javascript
module.exports = {
  extends: [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended",
    "plugin:testing-library/react",
    "plugin:jest/recommended",
  ],
};
```

## ⚡ Unit Testing

### Jest Configuration

```javascript
module.exports = {
  preset: "ts-jest",
  testEnvironment: "jsdom",
  setupFilesAfterEnv: ["<rootDir>/jest.setup.ts"],
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/src/$1",
  },
  collectCoverageFrom: ["src/**/*.{ts,tsx}", "!src/**/*.d.ts"],
};
```

### Example Unit Test

```typescript
// libs/shared/utils/date.test.ts
import { formatDate } from "./date";

describe("formatDate", () => {
  it("formats date correctly", () => {
    const date = new Date("2024-03-15");
    expect(formatDate(date)).toBe("March 15, 2024");
  });

  it("handles invalid dates", () => {
    expect(formatDate(null)).toBe("Invalid Date");
  });
});
```

## 🔄 Integration Testing

### React Testing Library Setup

```typescript
// jest.setup.ts
import "@testing-library/jest-dom";
import "jest-extended";
```

### Example Integration Test

```typescript
// apps/web/components/JobCard.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { JobCard } from "./JobCard";

describe("JobCard", () => {
  const mockJob = {
    id: "1",
    title: "Software Engineer",
    company: "Tech Corp",
    status: "applied",
  };

  it("renders job details correctly", () => {
    render(<JobCard job={mockJob} />);

    expect(screen.getByText("Software Engineer")).toBeInTheDocument();
    expect(screen.getByText("Tech Corp")).toBeInTheDocument();
    expect(screen.getByText("Applied")).toBeInTheDocument();
  });

  it("calls onStatusChange when status is updated", () => {
    const onStatusChange = jest.fn();
    render(<JobCard job={mockJob} onStatusChange={onStatusChange} />);

    fireEvent.click(screen.getByRole("button", { name: /change status/i }));
    fireEvent.click(screen.getByText("Interview"));

    expect(onStatusChange).toHaveBeenCalledWith("1", "interview");
  });
});
```

## 🌐 End-to-End Testing

### Cypress (Web)

Configuration:

```javascript
// cypress.config.ts
import { defineConfig } from "cypress";

export default defineConfig({
  e2e: {
    baseUrl: "http://localhost:3000",
    supportFile: "cypress/support/e2e.ts",
    specPattern: "cypress/e2e/**/*.cy.ts",
  },
});
```

Example test:

```typescript
// cypress/e2e/job-tracking.cy.ts
describe("Job Tracking", () => {
  beforeEach(() => {
    cy.login();
    cy.visit("/dashboard");
  });

  it("can add a new job application", () => {
    cy.get('[data-testid="add-job-button"]').click();
    cy.get('[data-testid="job-title"]').type("Frontend Developer");
    cy.get('[data-testid="company-name"]').type("Tech Inc");
    cy.get('[data-testid="submit-job"]').click();

    cy.contains("Frontend Developer").should("be.visible");
    cy.contains("Tech Inc").should("be.visible");
  });

  it("can update job status", () => {
    cy.get('[data-testid="job-card"]')
      .first()
      .find('[data-testid="status-button"]')
      .click();
    cy.get('[data-testid="status-interview"]').click();

    cy.get('[data-testid="job-card"]').first().should("contain", "Interview");
  });
});
```

### Detox (Mobile)

Configuration:

```javascript
// .detoxrc.js
module.exports = {
  testRunner: "jest",
  apps: {
    "ios.debug": {
      type: "ios.app",
      binaryPath:
        "ios/build/Build/Products/Debug-iphonesimulator/TrackMyJob.app",
      build:
        "xcodebuild -workspace ios/TrackMyJob.xcworkspace -scheme TrackMyJob -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build",
    },
  },
  devices: {
    simulator: {
      type: "ios.simulator",
      device: {
        type: "iPhone 14",
      },
    },
  },
};
```

Example test:

```typescript
// e2e/job-tracking.test.ts
describe("Job Tracking Flow", () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it("should add new job application", async () => {
    await element(by.id("add-job-button")).tap();
    await element(by.id("job-title")).typeText("Mobile Developer");
    await element(by.id("company-name")).typeText("App Co");
    await element(by.id("submit-job")).tap();

    await expect(element(by.text("Mobile Developer"))).toBeVisible();
    await expect(element(by.text("App Co"))).toBeVisible();
  });

  it("should update job status", async () => {
    await element(by.id("job-card")).atIndex(0).tap();
    await element(by.id("status-button")).tap();
    await element(by.text("Interview")).tap();

    await expect(element(by.text("Interview"))).toBeVisible();
  });
});
```

## 📊 Test Coverage

We aim for the following coverage targets:

- Unit Tests: 80%
- Integration Tests: 70%
- E2E Tests: Key user flows

Run coverage reports:

```bash
# Unit and Integration Tests
npm run test:coverage

# E2E Coverage (Cypress)
npm run cypress:coverage

# E2E Coverage (Detox)
npm run detox:coverage
```

## 🔄 CI/CD Integration

Tests are run in the following order in our CI pipeline:

1. Static Analysis:

   ```yaml
   static-analysis:
     runs-on: ubuntu-latest
     steps:
       - uses: actions/checkout@v3
       - run: npm ci
       - run: npm run type-check
       - run: npm run lint
   ```

2. Unit & Integration Tests:

   ```yaml
   tests:
     runs-on: ubuntu-latest
     steps:
       - uses: actions/checkout@v3
       - run: npm ci
       - run: npm test
   ```

3. E2E Tests:
   ```yaml
   e2e:
     runs-on: macos-latest
     steps:
       - uses: actions/checkout@v3
       - run: npm ci
       - run: npm run e2e:web
       - run: npm run e2e:mobile
   ```

## 🎯 Best Practices

1. **Test Organization**:

   - Keep tests close to implementation
   - Use descriptive test names
   - Follow AAA pattern (Arrange, Act, Assert)

2. **Test Data**:

   - Use factories for test data
   - Avoid sharing mutable state
   - Clean up after tests

3. **Mocking**:

   - Mock external dependencies
   - Use MSW for API mocking
   - Keep mocks simple and maintainable

4. **Continuous Improvement**:
   - Review test coverage regularly
   - Update tests with code changes
   - Refactor tests for maintainability
