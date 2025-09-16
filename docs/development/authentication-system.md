# Authentication System

## Overview

The DealSphere platform implements a comprehensive, security-first authentication system using JWT tokens, invitation-based user registration, and external observability for security auditing.

## Architecture

### Components

```mermaid
graph TB
    A[Frontend React App] --> B[GraphQL API]
    B --> C[AuthService]
    C --> D[JwtService]
    C --> E[InvitationService]
    C --> F[PasswordResetService]
    C --> G[SecurityAuditService]

    H[Admin User] --> I[InviteUser Mutation]
    I --> E
    E --> J[Email Service]
    J --> K[SMTP Server]

    L[New User] --> M[CompleteRegistration]
    M --> E

    N[User] --> O[Login Mutation]
    O --> C

    C --> P[(PostgreSQL)]
    G --> Q[External Observability]

    R[nginx] --> B
    R --> S[Rate Limiting]
    R --> T[Security Headers]
```

## Core Features

### 1. JWT-Based Authentication

**Location**: `backend/commons/auth/src/main/java/com/dealsphere/backend/commons/auth/service/JwtServiceImpl.java`

- **Token Generation**: Creates secure JWT tokens with user claims
- **Token Validation**: Validates token signature and expiration
- **Automatic Refresh**: Frontend automatically refreshes tokens before expiration
- **Secure Storage**: Tokens stored in localStorage with proper cleanup

```java
@Service
public class JwtServiceImpl implements JwtService {

    public String generateToken(UserProfile userProfile) {
        return Jwts.builder()
            .setSubject(userProfile.getEmail())
            .claim("userId", userProfile.getId())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpirationMs))
            .signWith(getSigningKey())
            .compact();
    }
}
```

### 2. Invitation-Based Registration

**Flow**:
1. Admin invites user via email through `InviteUserForm`
2. System generates secure invitation token
3. Email sent with registration link
4. User completes registration with temporary password
5. Account activated and user can log in

**Key Files**:
- Backend: `InvitationServiceImpl.java`
- Frontend: `InviteUserForm.tsx`, `CompleteRegistrationForm.tsx`

### 3. Password Reset System

**Flow**:
1. User requests password reset via email
2. System generates time-limited reset token
3. Reset link sent via email
4. User confirms reset with new password
5. Old sessions invalidated

**Security Features**:
- Rate limiting (3 attempts per hour)
- Token expiration (15 minutes)
- Anti-enumeration protection
- Secure token generation

### 4. Security Auditing

**Location**: `SecurityAuditServiceImpl.java`

All security events are logged to external observability systems with structured data:

```java
@Override
@Async
public void logSecurityEvent(SecurityEvent event) {
    Map<String, Object> logFields = createStructuredLogFields(event);

    switch (event.getSeverity()) {
        case CRITICAL:
            log.error("SECURITY_EVENT: {}", event.getMessage(),
                StructuredArguments.entries(logFields));
            break;
        // ... other severity levels
    }
}
```

**Event Types**:
- Login success/failure
- Password changes
- Account modifications
- Access denied events
- XSS/SQL injection attempts
- Privilege escalation attempts
- Configuration changes

## Frontend Implementation

### Authentication Hook

**Location**: `frontend/src/hooks/useAuth.ts`

```typescript
export const useAuth = () => {
  const dispatch = useAppDispatch();
  const auth = useAppSelector(state => state.auth);

  const login = useCallback(async (email: string, password: string) => {
    dispatch(loginStart());
    try {
      const result = await loginMutation({ variables: { input: { email, password } } });
      const { token, user } = result.data.login;

      localStorage.setItem('auth_token', token);
      dispatch(loginSuccess({ user, token }));

      return { user, token };
    } catch (error) {
      dispatch(loginFailure(error.message));
      throw error;
    }
  }, [dispatch, loginMutation]);

  // ... other auth methods
};
```

### Form Validation & Security

**Location**: `frontend/src/utils/validation.ts`

All forms include comprehensive validation:

```typescript
export const sanitizeXSS = (input: string): string => {
  if (!input) return input;
  let sanitized = input;
  XSS_PATTERNS.forEach(pattern => {
    sanitized = sanitized.replace(pattern, '');
  });
  // HTML encode special characters
  return sanitized
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/\//g, '&#x2F;');
};

export const detectSQLInjection = (input: string): boolean => {
  if (!input) return false;
  const sqlPatterns = [
    /(\b(union|select|insert|update|delete|drop|create|alter|exec|execute)\b)/i,
    /(\bor\b.*\b=\b.*\bor\b)/i,
    /(;|\-\-|\*|\/\*)/,
    // ... more patterns
  ];
  return sqlPatterns.some(pattern => pattern.test(input));
};
```

## Security Features

### 1. Rate Limiting (nginx)

**Location**: `nginx/nginx.conf`

```nginx
# Rate limiting zones
limit_req_zone $binary_remote_addr zone=auth:10m rate=5r/m;
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/m;
limit_req_zone $binary_remote_addr zone=reset:10m rate=1r/m;

location /api/auth/ {
    limit_req zone=auth burst=10 nodelay;
    limit_req_status 429;
    # ... proxy configuration
}
```

### 2. Security Headers

```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

# Content Security Policy
add_header Content-Security-Policy "
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdnjs.cloudflare.com;
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    font-src 'self' https://fonts.gstatic.com;
    img-src 'self' data: https:;
    connect-src 'self' ws: wss: https://api.example.com;
" always;
```

### 3. Input Validation

**Backend** (`ValidationConfig.java`):
- XSS pattern detection and sanitization
- SQL injection detection
- Input length validation
- Pattern matching validation

**Frontend**:
- Real-time validation with debouncing
- XSS/SQL injection detection
- Email format validation
- Password strength requirements

### 4. CORS Configuration

```nginx
# CORS headers
add_header 'Access-Control-Allow-Origin' '$allowed_origin' always;
add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Authorization' always;
add_header 'Access-Control-Allow-Credentials' 'true' always;
add_header 'Access-Control-Max-Age' '86400' always;
```

## Database Schema

### Core Tables

```sql
-- User profiles
CREATE TABLE user_profiles (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    organization_id VARCHAR(36) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    roles TEXT[] -- Array of role IDs
);

-- User credentials
CREATE TABLE user_credentials (
    user_id VARCHAR(36) PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES user_profiles(id)
);

-- User invitations
CREATE TABLE user_invitations (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    organization_id VARCHAR(36) NOT NULL,
    invited_by_user_id VARCHAR(36) NOT NULL,
    invitation_token VARCHAR(255) UNIQUE NOT NULL,
    temporary_password VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    is_used BOOLEAN DEFAULT false,
    used_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Password reset tokens
CREATE TABLE password_reset_tokens (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    reset_token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    is_used BOOLEAN DEFAULT false,
    used_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES user_profiles(id)
);

-- Security events (for external observability reference)
CREATE TABLE security_events (
    id VARCHAR(36) PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    severity VARCHAR(20) NOT NULL,
    user_id VARCHAR(36),
    session_id VARCHAR(100),
    ip_address VARCHAR(45),
    user_agent VARCHAR(500),
    endpoint VARCHAR(200),
    message VARCHAR(1000) NOT NULL,
    risk_score INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## GraphQL API

### Mutations

```graphql
type Mutation {
    # Authentication
    login(input: LoginInput!): AuthPayload!
    refreshToken: AuthPayload!

    # User Management
    inviteUser(input: InviteUserInput!): InvitationPayload!
    completeRegistration(input: CompleteRegistrationInput!): AuthPayload!

    # Password Management
    requestPasswordReset(input: PasswordResetRequestInput!): PasswordResetPayload!
    confirmPasswordReset(input: PasswordResetConfirmInput!): String!
    validatePasswordResetToken(resetToken: String!): Boolean!
}

type AuthPayload {
    token: String!
    user: UserProfile!
}

type InvitationPayload {
    success: Boolean!
    message: String!
    invitationId: String
}
```

## Testing

### Backend Tests

**Coverage**: 90%+

- **AuthServiceImplTest**: Unit tests for authentication flows
- **JwtServiceImplTest**: Token generation/validation tests
- **SecurityAuditServiceImplTest**: Security logging tests
- **Integration Tests**: End-to-end authentication flows

### Frontend Tests

**Coverage**: 85%+

- **Login.test.tsx**: Login form and validation tests
- **InviteUserForm.test.tsx**: User invitation tests
- **PasswordReset.test.tsx**: Password reset flow tests
- **useAuth.test.ts**: Authentication hook tests
- **validation.test.ts**: Security validation tests
- **Integration Tests**: Complete authentication flows

### Test Commands

```bash
# Backend tests
cd backend
./gradlew test

# Frontend tests
cd frontend
pnpm test                    # Unit tests
pnpm test:coverage          # Coverage report
pnpm test:e2e              # End-to-end tests
```

## Deployment Considerations

### Environment Variables

```bash
# JWT Configuration
JWT_SECRET=your-super-secure-secret-key-here
JWT_EXPIRATION_MS=86400000  # 24 hours

# Database
DATABASE_URL=postgresql://user:pass@host:5432/dealsphere

# Email Configuration
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=noreply@dealsphere.com
SMTP_PASS=smtp-password

# Security
BCRYPT_ROUNDS=12
RATE_LIMIT_MAX_ATTEMPTS=5
PASSWORD_RESET_EXPIRY_MINUTES=15
```

### Security Checklist

- [ ] JWT secrets are properly secured
- [ ] Database connections use SSL
- [ ] SMTP credentials are encrypted
- [ ] Rate limiting is configured
- [ ] Security headers are enabled
- [ ] Input validation is comprehensive
- [ ] Audit logging is functional
- [ ] CORS is properly configured
- [ ] SSL certificates are valid
- [ ] Backup and recovery tested

## Monitoring & Observability

### Security Events

All security events are logged to external observability systems (ELK, Datadog, etc.) with structured data:

```json
{
  "message": "SECURITY_EVENT: User logged in successfully",
  "event_type": "LOGIN_SUCCESS",
  "severity": "LOW",
  "user_id": "user-123",
  "ip_address": "192.168.1.100",
  "session_id": "session-456",
  "endpoint": "/api/auth/login",
  "risk_score": 10,
  "service": "dealsphere-auth",
  "timestamp": "2025-01-16T10:30:00Z"
}
```

### Key Metrics to Monitor

1. **Authentication Metrics**:
   - Login success/failure rates
   - Token validation failures
   - Password reset requests
   - Account lockouts

2. **Security Metrics**:
   - XSS/SQL injection attempts
   - Rate limit violations
   - Suspicious IP activity
   - Geographic anomalies

3. **Performance Metrics**:
   - Authentication response times
   - Token generation latency
   - Database query performance
   - Email delivery success rates

## Troubleshooting

### Common Issues

1. **Token Validation Failures**
   - Check JWT secret configuration
   - Verify token expiration settings
   - Ensure clock synchronization

2. **Email Delivery Issues**
   - Verify SMTP configuration
   - Check spam folder policies
   - Monitor email service quotas

3. **Rate Limiting False Positives**
   - Review nginx rate limit zones
   - Check IP whitelisting
   - Monitor legitimate traffic patterns

4. **Database Connection Issues**
   - Verify connection pool settings
   - Check SSL certificate validity
   - Monitor connection timeouts

### Debug Commands

```bash
# Check JWT token validity
curl -H "Authorization: Bearer <token>" http://localhost:8080/api/auth/validate

# Test email configuration
curl -X POST -d '{"email":"test@example.com"}' \
  -H "Content-Type: application/json" \
  http://localhost:8080/api/auth/request-password-reset

# Monitor security events
tail -f /var/log/dealsphere/security.log | jq '.severity=="HIGH"'
```