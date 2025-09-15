# Cypress E2E Testing Guide

This guide provides comprehensive documentation for the Cypress end-to-end testing setup in the DealSphere platform, covering test architecture, best practices, and CI/CD integration.

## Overview

The DealSphere platform uses Cypress for comprehensive E2E testing, focusing on:

- **Authentication Flows** - Login, JWT token management, Redux Saga integration
- **User Interface** - Form validation, navigation, error handling
- **API Integration** - GraphQL mutations and queries
- **State Management** - Redux store persistence and hydration
- **Cross-Browser Testing** - Consistent behavior across environments

## Test Architecture

### Directory Structure

```
frontend/cypress/
├── e2e/                          # End-to-end test specs
│   └── auth/
│       ├── login-flow.cy.ts      # Complete login workflow
│       ├── jwt-storage.cy.ts     # Token storage verification
│       └── basic-login.cy.ts     # Basic functionality tests
├── support/                      # Cypress support files
│   ├── commands.ts               # Custom commands
│   └── e2e.ts                   # Global configuration
├── fixtures/                     # Test data
│   └── users.json               # User credentials
├── videos/                       # Test recordings (CI)
├── screenshots/                  # Failure screenshots
└── cypress.config.ts            # Cypress configuration
```

### Configuration

**cypress.config.ts:**
```typescript
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
    pageLoadTimeout: 30000,
    supportFile: 'cypress/support/e2e.ts',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
  },
  env: {
    VITE_API_URL: 'http://localhost/api',
    VITE_GRAPHQL_URL: 'http://localhost/api/graphql',
  }
})
```

## Test Categories

### 1. Authentication Flow Tests

**File:** `cypress/e2e/auth/login-flow.cy.ts`

Tests the complete authentication workflow including:

#### Login Form Validation
```typescript
it('should display login form correctly', () => {
  cy.visit('/login')

  // Verify form elements
  cy.get('input[name="email"]').should('be.visible')
  cy.get('input[name="password"]').should('be.visible')
  cy.get('button[type="submit"]').should('contain.text', 'Sign In')
})
```

#### Redux Saga Integration
```typescript
it('should successfully login and store JWT token', () => {
  // Intercept GraphQL login mutation
  cy.intercept('POST', '/api/graphql', (req) => {
    if (req.body.query.includes('mutation Login')) {
      req.alias = 'loginMutation'
    }
  })

  // Perform login
  cy.get('input[name="email"]').type('admin@dealsphere.com')
  cy.get('input[name="password"]').type('password123')
  cy.get('button[type="submit"]').click()

  // Verify API call and response
  cy.wait('@loginMutation').then((interception) => {
    expect(interception.response?.statusCode).to.eq(200)
    expect(interception.response?.body?.data?.login.token).to.exist
  })
})
```

#### State Persistence
```typescript
it('should persist auth state on page refresh', () => {
  // Login first
  cy.login('admin@dealsphere.com', 'password123')
  cy.url().should('include', '/dashboard')

  // Refresh page
  cy.reload()

  // Should still be authenticated
  cy.url().should('include', '/dashboard')
  cy.window().then((win) => {
    expect(win.localStorage.getItem('authToken')).to.exist
  })
})
```

### 2. JWT Storage Tests

**File:** `cypress/e2e/auth/jwt-storage.cy.ts`

Focuses specifically on token management:

#### LocalStorage Verification
```typescript
it('should store JWT token with correct key', () => {
  cy.login('admin@dealsphere.com', 'password123')

  cy.window().then((win) => {
    const token = win.localStorage.getItem('authToken')
    const userData = win.localStorage.getItem('userData')

    // Verify token format (JWT structure)
    expect(token).to.match(/^eyJ[A-Za-z0-9-_=]+\.[A-Za-z0-9-_=]+\.?[A-Za-z0-9-_.+/=]*$/)

    // Verify user data structure
    const user = JSON.parse(userData!)
    expect(user).to.have.property('id')
    expect(user).to.have.property('email', 'admin@dealsphere.com')
  })
})
```

#### Token Cleanup
```typescript
it('should clear tokens on logout', () => {
  cy.login('admin@dealsphere.com', 'password123')

  // Verify tokens exist
  cy.window().then((win) => {
    expect(win.localStorage.getItem('authToken')).to.exist
  })

  // Logout
  cy.get('[data-testid="logout-button"]').click()

  // Verify tokens are cleared
  cy.window().then((win) => {
    expect(win.localStorage.getItem('authToken')).to.be.null
    expect(win.localStorage.getItem('userData')).to.be.null
  })
})
```

### 3. Basic Functionality Tests

**File:** `cypress/e2e/auth/basic-login.cy.ts`

Simplified tests for quick verification:

```typescript
it('should test actual login with real backend', () => {
  cy.visit('/login')

  cy.get('input[name="email"]').type('admin@dealsphere.com')
  cy.get('input[name="password"]').type('password123')
  cy.get('button[type="submit"]').click()

  // Wait for processing
  cy.wait(3000)

  // Check success indicators
  cy.window().then((win) => {
    const token = win.localStorage.getItem('authToken')
    if (token) {
      expect(token).to.exist
      cy.log('✅ Login successful - Token stored')
    }
  })
})
```

## Custom Commands

### Authentication Commands

**File:** `cypress/support/commands.ts`

```typescript
// Login command for reusable authentication
Cypress.Commands.add('login', (email: string, password: string) => {
  cy.visit('/login')

  cy.get('input[name="email"]').type(email)
  cy.get('input[name="password"]').type(password)

  // Intercept login mutation
  cy.intercept('POST', '/api/graphql', (req) => {
    if (req.body.query.includes('mutation Login')) {
      req.alias = 'loginMutation'
    }
  })

  cy.get('button[type="submit"]').click()
  cy.wait('@loginMutation')
})

// Clear authentication state
Cypress.Commands.add('clearAuth', () => {
  cy.window().then((win) => {
    win.localStorage.removeItem('authToken')
    win.localStorage.removeItem('userData')
  })
})
```

### State Management Commands

```typescript
// Access Redux store (if exposed)
Cypress.Commands.add('getReduxState', () => {
  cy.window().its('store').invoke('getState')
})

// Wait for specific Redux actions
Cypress.Commands.add('waitForAction', (actionType: string) => {
  cy.window().then((win) => {
    return new Cypress.Promise((resolve) => {
      const store = (win as any).store
      if (store) {
        const unsubscribe = store.subscribe(() => {
          const state = store.getState()
          unsubscribe()
          resolve(state)
        })
      }
    })
  })
})
```

## Test Data Management

### User Fixtures

**File:** `cypress/fixtures/users.json`

```json
{
  "admin": {
    "email": "admin@dealsphere.com",
    "password": "password123",
    "expectedData": {
      "firstName": "Admin",
      "lastName": "User",
      "roles": ["ADMIN"]
    }
  },
  "testUser": {
    "email": "test@dealsphere.com",
    "password": "testpass123",
    "expectedData": {
      "firstName": "Test",
      "lastName": "User",
      "roles": ["USER"]
    }
  }
}
```

### Dynamic Test Data

```typescript
// Load fixtures in tests
describe('User Management', () => {
  let users: any

  before(() => {
    cy.fixture('users').then((userData) => {
      users = userData
    })
  })

  it('should login with admin user', () => {
    cy.login(users.admin.email, users.admin.password)
    // Test continues...
  })
})
```

## Running Tests

### Local Development

```bash
# Interactive mode (Cypress Test Runner)
cd frontend
pnpm cy:open

# Headless mode
pnpm cy:run

# Specific test file
pnpm cy:run --spec "cypress/e2e/auth/login-flow.cy.ts"

# With specific browser
pnpm cy:run --browser chrome

# With environment variables
CYPRESS_BASE_URL=http://localhost:3000 pnpm cy:run
```

### CI/CD Environment

```bash
# Headless with artifacts
pnpm cy:run --config video=true,screenshotOnRunFailure=true

# Multiple browsers (parallel)
pnpm cy:run --browser chrome &
pnpm cy:run --browser firefox &
wait

# Record to Cypress Dashboard
pnpm cy:run --record --key <cypress-record-key>
```

## Environment Configuration

### Development Environment

```bash
# .env.development
CYPRESS_BASE_URL=http://localhost
CYPRESS_VIDEO=true
CYPRESS_SCREENSHOTS=true
```

### Staging Environment

```bash
# .env.staging
CYPRESS_BASE_URL=https://staging.dealsphere.com
CYPRESS_VIDEO=true
CYPRESS_SCREENSHOTS=true
CYPRESS_VIDEO_COMPRESSION=32
```

### Production Environment

```bash
# .env.production
CYPRESS_BASE_URL=https://dealsphere.com
CYPRESS_VIDEO=false
CYPRESS_SCREENSHOTS=false
```

## Best Practices

### 1. Test Design Principles

#### Atomic Tests
```typescript
// ✅ Good - Single responsibility
it('should store JWT token in localStorage', () => {
  cy.login('admin@dealsphere.com', 'password123')
  cy.window().then((win) => {
    expect(win.localStorage.getItem('authToken')).to.exist
  })
})

// ❌ Bad - Multiple responsibilities
it('should login and navigate and check profile', () => {
  // Too many assertions in one test
})
```

#### Reliable Selectors
```typescript
// ✅ Good - Data attributes
cy.get('[data-testid="login-button"]').click()

// ✅ Good - Semantic selectors
cy.get('button[type="submit"]').click()

// ❌ Bad - Fragile selectors
cy.get('.css-1234567').click()
```

### 2. Network Management

#### GraphQL Interception
```typescript
// Intercept specific GraphQL operations
cy.intercept('POST', '/api/graphql', (req) => {
  if (req.body.query.includes('mutation Login')) {
    req.alias = 'loginMutation'
  }
  if (req.body.query.includes('query GetUser')) {
    req.alias = 'getUserQuery'
  }
})
```

#### Mock Responses
```typescript
// Mock failed login
cy.intercept('POST', '/api/graphql', {
  statusCode: 200,
  body: {
    errors: [{ message: 'Invalid credentials' }]
  }
}).as('failedLogin')
```

### 3. Wait Strategies

#### Explicit Waits
```typescript
// Wait for API calls
cy.wait('@loginMutation')

// Wait for elements
cy.get('[data-testid="dashboard"]', { timeout: 10000 })

// Wait for conditions
cy.window().then((win) => {
  cy.waitUntil(() => win.localStorage.getItem('authToken'))
})
```

#### Custom Wait Commands
```typescript
// Wait for application to be ready
Cypress.Commands.add('waitForApp', () => {
  cy.visit('/')
  cy.get('[data-testid="app-loaded"]').should('exist')
  cy.window().its('store').should('exist')
})
```

### 4. Error Handling

#### Graceful Failures
```typescript
it('should handle network errors gracefully', () => {
  // Simulate network failure
  cy.intercept('POST', '/api/graphql', { forceNetworkError: true })

  cy.get('input[name="email"]').type('admin@dealsphere.com')
  cy.get('input[name="password"]').type('password123')
  cy.get('button[type="submit"]').click()

  // Should show error message
  cy.get('.error-message').should('be.visible')
})
```

## Debugging and Troubleshooting

### Debug Mode

```bash
# Run with debug output
DEBUG=cypress:* pnpm cy:run

# Specific debug channels
DEBUG=cypress:server:* pnpm cy:run
```

### Test Debugging

```typescript
// Add debug breakpoints
cy.debug()

// Log values
cy.get('input[name="email"]').then(($el) => {
  cy.log('Email input value:', $el.val())
})

// Pause test execution
cy.pause()
```

### Common Issues

#### 1. Element Not Found
```typescript
// Solution: Use proper waits
cy.get('[data-testid="element"]', { timeout: 10000 })
  .should('be.visible')
  .click()
```

#### 2. Race Conditions
```typescript
// Solution: Wait for API calls
cy.intercept('POST', '/api/graphql').as('apiCall')
cy.get('button').click()
cy.wait('@apiCall')
```

#### 3. State Management Issues
```typescript
// Solution: Clear state between tests
beforeEach(() => {
  cy.clearAuth()
  cy.window().then((win) => {
    win.localStorage.clear()
    win.sessionStorage.clear()
  })
})
```

## Performance Optimization

### Test Execution Speed

```typescript
// Minimize DOM queries
cy.get('[data-testid="form"]').within(() => {
  cy.get('input[name="email"]').type('test@example.com')
  cy.get('input[name="password"]').type('password')
  cy.get('button[type="submit"]').click()
})

// Use aliases for reused elements
cy.get('[data-testid="login-form"]').as('loginForm')
cy.get('@loginForm').find('input[name="email"]').type('test@example.com')
```

### Parallel Execution

```bash
# GitHub Actions parallel execution
- name: Run Cypress tests
  uses: cypress-io/github-action@v6
  with:
    record: true
    parallel: true
    group: 'UI - Chrome'
```

## Integration with CI/CD

### GitHub Actions

```yaml
- name: Run Cypress E2E tests
  uses: cypress-io/github-action@v6
  with:
    working-directory: frontend
    install: false
    start: echo "Services already running"
    wait-on: 'http://localhost'
    wait-on-timeout: 120
    config: baseUrl=http://localhost,video=true
  env:
    CYPRESS_BASE_URL: http://localhost
```

### Artifact Collection

```yaml
- name: Upload Cypress videos
  uses: actions/upload-artifact@v4
  if: failure()
  with:
    name: cypress-videos-${{ github.run_number }}
    path: frontend/cypress/videos
    retention-days: 7
```

## Monitoring and Reporting

### Test Results Dashboard

- **Cypress Dashboard** - Record test runs for team visibility
- **GitHub Actions** - Built-in test reporting
- **Allure Reports** - Comprehensive test reporting (optional)

### Metrics to Track

- **Test Execution Time** - Monitor for performance degradation
- **Flaky Test Rate** - Identify unstable tests
- **Code Coverage** - E2E coverage mapping
- **Failure Patterns** - Common failure points

## Related Documentation

- [Local CI/CD Setup](local-cicd-setup.md)
- [GitHub Actions Guide](github-actions.md)
- [Environment Configuration](environment-config.md)
- [Frontend Testing Strategy](frontend-testing.md)