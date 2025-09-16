# API Documentation

## Overview

The DealSphere platform provides a comprehensive GraphQL API for authentication, user management, and business operations. This document covers the complete API specification, authentication mechanisms, and usage examples.

## GraphQL API Endpoint

- **Production**: `https://api.dealsphere.com/graphql`
- **Staging**: `https://staging-api.dealsphere.com/graphql`
- **Development**: `http://localhost:8080/graphql`

## Authentication

### JWT Token-Based Authentication

All API requests (except login and public queries) require a valid JWT token in the Authorization header:

```http
Authorization: Bearer <jwt_token>
```

### Token Lifecycle

1. **Obtain Token**: Use `login` mutation with valid credentials
2. **Use Token**: Include in Authorization header for protected operations
3. **Refresh Token**: Use `refreshToken` mutation before expiration
4. **Token Expiration**: Production tokens expire in 1 hour, development in 24 hours

## Schema Definition

### Core Types

```graphql
# User Profile
type UserProfile {
  id: ID!
  email: String!
  firstName: String!
  lastName: String!
  organization: Organization!
  roles: [Role!]!
  isActive: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Organization
type Organization {
  id: ID!
  name: String!
  description: String
  isActive: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Role
type Role {
  id: ID!
  name: String!
  description: String
  organization: Organization!
  permissions: [String!]!
  isActive: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Authentication Payload
type AuthPayload {
  token: String!
  user: UserProfile!
}

# Invitation Payload
type InvitationPayload {
  success: Boolean!
  message: String!
  invitationId: String
}

# Password Reset Payload
type PasswordResetPayload {
  success: Boolean!
  message: String!
}
```

### Input Types

```graphql
# Login Input
input LoginInput {
  email: String!
  password: String!
}

# User Invitation Input
input InviteUserInput {
  email: String!
  firstName: String!
  lastName: String!
  organizationId: ID!
}

# Registration Completion Input
input CompleteRegistrationInput {
  invitationToken: String!
  password: String!
}

# Password Reset Request Input
input PasswordResetRequestInput {
  email: String!
}

# Password Reset Confirmation Input
input PasswordResetConfirmInput {
  resetToken: String!
  newPassword: String!
}

# User Update Input
input UserUpdateInput {
  firstName: String
  lastName: String
  isActive: Boolean
}
```

## Mutations

### Authentication Mutations

#### Login

Authenticate user with email and password.

```graphql
mutation Login($input: LoginInput!) {
  login(input: $input) {
    token
    user {
      id
      email
      firstName
      lastName
      organization {
        id
        name
      }
      roles {
        id
        name
        permissions
      }
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "email": "user@example.com",
    "password": "password123"
  }
}
```

**Response:**
```json
{
  "data": {
    "login": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "user": {
        "id": "user-123",
        "email": "user@example.com",
        "firstName": "John",
        "lastName": "Doe",
        "organization": {
          "id": "org-456",
          "name": "Acme Corp"
        },
        "roles": [
          {
            "id": "role-789",
            "name": "USER",
            "permissions": ["READ_PROFILE", "UPDATE_PROFILE"]
          }
        ]
      }
    }
  }
}
```

**Error Responses:**
```json
{
  "errors": [
    {
      "message": "Invalid email or password",
      "extensions": {
        "code": "AUTHENTICATION_ERROR"
      }
    }
  ]
}
```

#### Refresh Token

Refresh an existing JWT token.

```graphql
mutation RefreshToken {
  refreshToken {
    token
    user {
      id
      email
    }
  }
}
```

**Headers:**
```http
Authorization: Bearer <current_jwt_token>
```

#### Logout

Invalidate current session (client-side token removal).

```graphql
mutation Logout {
  logout
}
```

### User Management Mutations

#### Invite User

Send invitation email to new user (Admin only).

```graphql
mutation InviteUser($input: InviteUserInput!) {
  inviteUser(input: $input) {
    success
    message
    invitationId
  }
}
```

**Variables:**
```json
{
  "input": {
    "email": "newuser@example.com",
    "firstName": "Jane",
    "lastName": "Smith",
    "organizationId": "org-456"
  }
}
```

**Response:**
```json
{
  "data": {
    "inviteUser": {
      "success": true,
      "message": "Invitation sent successfully",
      "invitationId": "inv-789"
    }
  }
}
```

#### Complete Registration

Complete user registration from invitation.

```graphql
mutation CompleteRegistration($input: CompleteRegistrationInput!) {
  completeRegistration(input: $input) {
    token
    user {
      id
      email
      firstName
      lastName
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "invitationToken": "inv-token-123",
    "password": "newpassword123"
  }
}
```

#### Update User Profile

Update user profile information.

```graphql
mutation UpdateUserProfile($id: ID!, $input: UserUpdateInput!) {
  updateUserProfile(id: $id, input: $input) {
    id
    firstName
    lastName
    isActive
    updatedAt
  }
}
```

### Password Management Mutations

#### Request Password Reset

Send password reset email.

```graphql
mutation RequestPasswordReset($input: PasswordResetRequestInput!) {
  requestPasswordReset(input: $input) {
    success
    message
  }
}
```

**Variables:**
```json
{
  "input": {
    "email": "user@example.com"
  }
}
```

#### Confirm Password Reset

Complete password reset with token.

```graphql
mutation ConfirmPasswordReset($input: PasswordResetConfirmInput!) {
  confirmPasswordReset(input: $input)
}
```

**Variables:**
```json
{
  "input": {
    "resetToken": "reset-token-123",
    "newPassword": "newpassword123"
  }
}
```

#### Validate Password Reset Token

Check if password reset token is valid.

```graphql
mutation ValidatePasswordResetToken($resetToken: String!) {
  validatePasswordResetToken(resetToken: $resetToken)
}
```

## Queries

### User Queries

#### Get Current User

Get current authenticated user profile.

```graphql
query GetCurrentUser {
  me {
    id
    email
    firstName
    lastName
    organization {
      id
      name
      description
    }
    roles {
      id
      name
      description
      permissions
    }
    isActive
    createdAt
    updatedAt
  }
}
```

#### Get User by ID

Get specific user by ID (Admin only).

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    email
    firstName
    lastName
    organization {
      id
      name
    }
    roles {
      id
      name
      permissions
    }
    isActive
    createdAt
    updatedAt
  }
}
```

#### List Users

Get paginated list of users (Admin only).

```graphql
query ListUsers($first: Int, $after: String, $organizationId: ID) {
  users(first: $first, after: $after, organizationId: $organizationId) {
    edges {
      node {
        id
        email
        firstName
        lastName
        isActive
        createdAt
      }
      cursor
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
    totalCount
  }
}
```

### Organization Queries

#### Get Organization

Get organization details.

```graphql
query GetOrganization($id: ID!) {
  organization(id: $id) {
    id
    name
    description
    isActive
    createdAt
    updatedAt
    userCount
  }
}
```

#### List Organizations

Get all organizations (Super Admin only).

```graphql
query ListOrganizations {
  organizations {
    id
    name
    description
    isActive
    userCount
    createdAt
  }
}
```

### Role Queries

#### List Roles

Get available roles for organization.

```graphql
query ListRoles($organizationId: ID!) {
  roles(organizationId: $organizationId) {
    id
    name
    description
    permissions
    isActive
    createdAt
  }
}
```

## Error Handling

### Error Types

The API uses standardized error codes and messages:

```typescript
interface GraphQLError {
  message: string;
  locations?: Array<{
    line: number;
    column: number;
  }>;
  path?: Array<string | number>;
  extensions: {
    code: string;
    details?: any;
  };
}
```

### Common Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `AUTHENTICATION_ERROR` | Invalid credentials or token | 401 |
| `AUTHORIZATION_ERROR` | Insufficient permissions | 403 |
| `VALIDATION_ERROR` | Input validation failed | 400 |
| `NOT_FOUND` | Resource not found | 404 |
| `RATE_LIMITED` | Too many requests | 429 |
| `INTERNAL_ERROR` | Server error | 500 |

### Error Examples

#### Authentication Error
```json
{
  "errors": [
    {
      "message": "Invalid email or password",
      "extensions": {
        "code": "AUTHENTICATION_ERROR"
      }
    }
  ]
}
```

#### Validation Error
```json
{
  "errors": [
    {
      "message": "Validation failed",
      "extensions": {
        "code": "VALIDATION_ERROR",
        "details": {
          "email": "Invalid email format",
          "password": "Password must be at least 8 characters"
        }
      }
    }
  ]
}
```

#### Rate Limiting Error
```json
{
  "errors": [
    {
      "message": "Too many requests",
      "extensions": {
        "code": "RATE_LIMITED",
        "details": {
          "retryAfter": 300,
          "limit": 5,
          "window": "15m"
        }
      }
    }
  ]
}
```

## Client Integration

### JavaScript/TypeScript

Using Apollo Client:

```typescript
import { ApolloClient, InMemoryCache, createHttpLink } from '@apollo/client';
import { setContext } from '@apollo/client/link/context';

// HTTP link
const httpLink = createHttpLink({
  uri: 'https://api.dealsphere.com/graphql',
});

// Auth link
const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('auth_token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    }
  };
});

// Apollo Client
const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache(),
});

// Usage example
import { gql } from '@apollo/client';

const LOGIN_MUTATION = gql`
  mutation Login($input: LoginInput!) {
    login(input: $input) {
      token
      user {
        id
        email
        firstName
        lastName
      }
    }
  }
`;

// Execute mutation
const [login] = useMutation(LOGIN_MUTATION);

const handleLogin = async (email: string, password: string) => {
  try {
    const result = await login({
      variables: { input: { email, password } }
    });

    const { token, user } = result.data.login;
    localStorage.setItem('auth_token', token);

    console.log('Login successful:', user);
  } catch (error) {
    console.error('Login failed:', error);
  }
};
```

### cURL Examples

#### Login
```bash
curl -X POST \
  https://api.dealsphere.com/graphql \
  -H 'Content-Type: application/json' \
  -d '{
    "query": "mutation Login($input: LoginInput!) { login(input: $input) { token user { id email } } }",
    "variables": {
      "input": {
        "email": "user@example.com",
        "password": "password123"
      }
    }
  }'
```

#### Authenticated Request
```bash
curl -X POST \
  https://api.dealsphere.com/graphql \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' \
  -d '{
    "query": "query GetCurrentUser { me { id email firstName lastName } }"
  }'
```

## Rate Limiting

### Limits by Endpoint

| Endpoint | Limit | Window |
|----------|--------|--------|
| Login | 5 requests | 15 minutes |
| Password Reset | 3 requests | 1 hour |
| General API | 100 requests | 1 hour |
| GraphQL Introspection | 10 requests | 1 minute |

### Rate Limit Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
Retry-After: 3600
```

### Handling Rate Limits

```typescript
const handleRateLimit = (error: ApolloError) => {
  if (error.graphQLErrors.some(e => e.extensions.code === 'RATE_LIMITED')) {
    const retryAfter = error.graphQLErrors[0].extensions.details.retryAfter;

    // Wait and retry
    setTimeout(() => {
      // Retry the request
    }, retryAfter * 1000);
  }
};
```

## Security Considerations

### Input Validation

All inputs are validated on the server side:

- **XSS Prevention**: HTML content is sanitized
- **SQL Injection**: Parameterized queries prevent injection
- **Input Length**: Maximum field lengths enforced
- **Format Validation**: Email, phone, URL format validation

### CORS Policy

```http
Access-Control-Allow-Origin: https://app.dealsphere.com, https://admin.dealsphere.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

### HTTPS Only

All API endpoints require HTTPS in production. HTTP requests are automatically redirected to HTTPS.

## WebSocket Subscriptions

### Real-time User Status

```graphql
subscription UserStatusUpdates($organizationId: ID!) {
  userStatusUpdates(organizationId: $organizationId) {
    user {
      id
      email
      isActive
    }
    status
    timestamp
  }
}
```

### Security Events

```graphql
subscription SecurityEvents {
  securityEvents {
    eventType
    severity
    message
    timestamp
    userAgent
    ipAddress
  }
}
```

## API Versioning

### Current Version: v1

The API follows semantic versioning. Breaking changes will result in a new major version.

### Version Headers

```http
API-Version: v1
```

### Deprecation Policy

- **Deprecation Notice**: 6 months before removal
- **Migration Guide**: Provided for breaking changes
- **Backward Compatibility**: Maintained for minor versions

## Testing & Development

### GraphQL Playground

Available at:
- Development: `http://localhost:8080/graphql-playground`
- Staging: `https://staging-api.dealsphere.com/graphql-playground`

### Schema Introspection

```graphql
query IntrospectionQuery {
  __schema {
    queryType {
      name
      fields {
        name
        type {
          name
        }
      }
    }
    mutationType {
      name
      fields {
        name
        args {
          name
          type {
            name
          }
        }
      }
    }
  }
}
```

### Mock Data

For development and testing, mock data is available:

```bash
# Start with mock data
docker-compose -f docker-compose.dev.yml up -d

# Import test data
npm run db:seed
```

This comprehensive API documentation provides all the information needed to integrate with the DealSphere platform's authentication and user management system.