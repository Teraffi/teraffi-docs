# ADR-020: User Authentication & Authorization

**Status:** Proposed  
**Date:** 2025-10-01  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001 (Runtime), ADR-019 (Multi-Tenancy)

---

## Context

TERAFFI requires secure authentication and fine-grained authorization for multiple user types across tenant boundaries. The system must support:

**User Types:**
- **Brand Executives**: Access to their brand's data and partnerships
- **IP Owners/Creators**: Access to their content and deals
- **TERAFFI Activators**: Cross-tenant access to manage partnerships
- **Admins**: Tenant-level administration
- **Platform Admins**: System-wide access (TERAFFI employees)

**Authentication Needs:**
- Email/password login
- SSO (SAML/OAuth) for enterprise tenants
- Multi-factor authentication (MFA)
- Session management
- Password reset flows

**Authorization Needs:**
- Role-Based Access Control (RBAC)
- Resource-level permissions
- Tenant isolation enforcement
- API key authentication for programmatic access

### Current Gap

Without proper authentication/authorization:
- **Security Risk**: No identity verification, anyone can access anything
- **Compliance Issues**: Cannot meet SOC 2 requirements for access control
- **User Experience**: No SSO = friction for enterprise users
- **Operational Risk**: No audit trail of who did what

### Requirements

**Functional:**
1. Secure password-based authentication
2. JWT-based session management
3. SSO integration (SAML 2.0, OAuth 2.0)
4. Multi-factor authentication (TOTP)
5. Role-based access control
6. Resource-level permissions
7. API key management
8. Session revocation

**Non-Functional:**
1. Authentication latency <100ms
2. Support 10,000+ concurrent sessions
3. SOC 2 compliant audit logging
4. Password storage with bcrypt (cost factor 12)
5. JWT expiry: 1 hour (with refresh tokens)
6. MFA recovery codes

---

## Decision

**Implement JWT-based authentication with RBAC authorization** using:
1. **Auth Service** for identity management
2. **JWT tokens** for stateless authentication
3. **Refresh tokens** stored in Redis
4. **Role-based permissions** with hierarchical roles
5. **SSO integration** via Auth0 or custom SAML/OAuth
6. **MFA** using TOTP (Time-based One-Time Password)
7. **API keys** for programmatic access

**Architecture:**
```
Client
    ↓
[Auth Gateway]
    ↓ (email/password or SSO)
Auth Service
    ↓
Issue JWT (access token + refresh token)
    ↓
API Requests with JWT
    ↓
[Auth Middleware] → Verify JWT → Extract user/tenant → Check permissions
    ↓
Business Logic (if authorized)
```

---

## Rationale

### JWT vs Session Cookies

**JWT (Stateless):**
- **Pros**: Scalable (no server-side session store), works across services, mobile-friendly
- **Cons**: Cannot revoke before expiry (mitigated with short expiry + refresh tokens)
- **Decision**: JWT for stateless auth, Redis for refresh token management

**Session Cookies (Stateful):**
- **Pros**: Easy to revoke, traditional web pattern
- **Cons**: Requires shared session store, sticky sessions, less mobile-friendly
- **Decision**: Not chosen due to microservices architecture

### Role-Based vs Attribute-Based Access Control

**RBAC (Role-Based):**
- Simpler to reason about and implement
- Users have roles, roles have permissions
- Sufficient for TERAFFI's needs

**ABAC (Attribute-Based):**
- More flexible but complex
- Overkill for current requirements
- Can migrate later if needed

**Decision**: Start with RBAC, migrate to ABAC if needed

---

## Implementation Details

### 1. User & Role Schema

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  email_verified BOOLEAN DEFAULT false,
  password_hash VARCHAR(255),  -- NULL for SSO users
  tenant_id UUID REFERENCES tenants(id),
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  avatar_url VARCHAR(500),
  
  -- MFA
  mfa_enabled BOOLEAN DEFAULT false,
  mfa_secret VARCHAR(255),  -- Encrypted TOTP secret
  mfa_recovery_codes TEXT[],  -- Encrypted recovery codes
  
  -- SSO
  sso_provider VARCHAR(50),  -- 'saml' | 'oauth' | null
  sso_subject VARCHAR(255),  -- External user ID from SSO provider
  
  -- Account status
  active BOOLEAN DEFAULT true,
  locked_until TIMESTAMPTZ,
  failed_login_attempts INTEGER DEFAULT 0,
  
  -- Timestamps
  last_login_at TIMESTAMPTZ,
  password_changed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_tenant ON users(tenant_id);
CREATE INDEX idx_users_sso ON users(sso_provider, sso_subject);

-- Roles table
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  tenant_id UUID REFERENCES tenants(id),  -- NULL for platform-wide roles
  description TEXT,
  permissions TEXT[],  -- Array of permission strings
  is_system_role BOOLEAN DEFAULT false,  -- Cannot be deleted
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(name, tenant_id)
);

-- User-Role assignments
CREATE TABLE user_roles (
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  assigned_at TIMESTAMPTZ DEFAULT NOW(),
  assigned_by UUID REFERENCES users(id),
  PRIMARY KEY (user_id, role_id)
);

CREATE INDEX idx_user_roles_user ON user_roles(user_id);
CREATE INDEX idx_user_roles_role ON user_roles(role_id);

-- API keys for programmatic access
CREATE TABLE api_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  tenant_id UUID REFERENCES tenants(id),
  name VARCHAR(255),
  key_hash VARCHAR(255) UNIQUE NOT NULL,  -- Hashed key
  key_prefix VARCHAR(20),  -- First 8 chars for identification
  permissions TEXT[],
  last_used_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_api_keys_user ON api_keys(user_id);
CREATE INDEX idx_api_keys_hash ON api_keys(key_hash);

-- Refresh tokens
CREATE TABLE refresh_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  token_hash VARCHAR(255) UNIQUE NOT NULL,
  device_info JSONB,  -- User agent, IP, etc.
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_refresh_tokens_user ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_hash ON refresh_tokens(token_hash);
CREATE INDEX idx_refresh_tokens_expiry ON refresh_tokens(expires_at);

-- Auth audit log
CREATE TABLE auth_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID,
  tenant_id UUID,
  event VARCHAR(100),  -- 'login' | 'logout' | 'mfa_success' | 'password_reset' | etc.
  ip_address INET,
  user_agent TEXT,
  success BOOLEAN,
  failure_reason TEXT,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_auth_audit_user ON auth_audit_log(user_id, timestamp DESC);
CREATE INDEX idx_auth_audit_event ON auth_audit_log(event, timestamp DESC);
```

### 2. Authentication Service

```typescript
// packages/auth-service/src/auth.ts

import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { authenticator } from 'otplib';

interface LoginRequest {
  email: string;
  password: string;
  mfa_code?: string;
}

interface JWTPayload {
  user_id: string;
  tenant_id: string;
  email: string;
  roles: string[];
  permissions: string[];
}

export class AuthService {
  private readonly JWT_SECRET = process.env.JWT_SECRET!;
  private readonly JWT_EXPIRY = '1h';
  private readonly REFRESH_TOKEN_EXPIRY_DAYS = 30;

  async login(request: LoginRequest): Promise<{ access_token: string; refresh_token: string }> {
    // 1. Find user by email
    const user = await this.db
      .selectFrom('users')
      .selectAll()
      .where('email', '=', request.email.toLowerCase())
      .executeTakeFirst();

    if (!user || !user.active) {
      await this.logAuthEvent(null, 'login', false, 'Invalid credentials');
      throw new UnauthorizedError('Invalid credentials');
    }

    // 2. Check account lock
    if (user.locked_until && user.locked_until > new Date()) {
      await this.logAuthEvent(user.id, 'login', false, 'Account locked');
      throw new ForbiddenError('Account temporarily locked');
    }

    // 3. Verify password
    if (user.password_hash) {
      const validPassword = await bcrypt.compare(request.password, user.password_hash);
      if (!validPassword) {
        await this.handleFailedLogin(user.id);
        throw new UnauthorizedError('Invalid credentials');
      }
    } else {
      // SSO user trying to use password
      throw new UnauthorizedError('Please use SSO to login');
    }

    // 4. Check MFA if enabled
    if (user.mfa_enabled) {
      if (!request.mfa_code) {
        return { requires_mfa: true } as any;
      }

      const validMFA = authenticator.verify({
        token: request.mfa_code,
        secret: this.decrypt(user.mfa_secret)
      });

      if (!validMFA) {
        await this.logAuthEvent(user.id, 'mfa_failed', false, 'Invalid MFA code');
        throw new UnauthorizedError('Invalid MFA code');
      }

      await this.logAuthEvent(user.id, 'mfa_success', true);
    }

    // 5. Load permissions
    const permissions = await this.getUserPermissions(user.id);

    // 6. Generate tokens
    const accessToken = this.generateAccessToken({
      user_id: user.id,
      tenant_id: user.tenant_id,
      email: user.email,
      roles: permissions.roles,
      permissions: permissions.permissions
    });

    const refreshToken = await this.generateRefreshToken(user.id);

    // 7. Update last login
    await this.db
      .updateTable('users')
      .set({
        last_login_at: new Date(),
        failed_login_attempts: 0,
        locked_until: null
      })
      .where('id', '=', user.id)
      .execute();

    await this.logAuthEvent(user.id, 'login', true);

    return {
      access_token: accessToken,
      refresh_token: refreshToken
    };
  }

  async register(request: {
    email: string;
    password: string;
    first_name: string;
    last_name: string;
    tenant_id: string;
  }): Promise<User> {
    // 1. Validate password strength
    this.validatePasswordStrength(request.password);

    // 2. Hash password
    const passwordHash = await bcrypt.hash(request.password, 12);

    // 3. Create user
    const user = await this.db
      .insertInto('users')
      .values({
        email: request.email.toLowerCase(),
        password_hash: passwordHash,
        first_name: request.first_name,
        last_name: request.last_name,
        tenant_id: request.tenant_id,
        email_verified: false
      })
      .returning('*')
      .executeTakeFirstOrThrow();

    // 4. Assign default role
    await this.assignDefaultRole(user.id, request.tenant_id);

    // 5. Send verification email
    await this.emailService.sendVerificationEmail(user.email);

    await this.logAuthEvent(user.id, 'register', true);

    return user;
  }

  async refreshAccessToken(refreshToken: string): Promise<{ access_token: string }> {
    // 1. Verify refresh token exists and not expired
    const tokenHash = this.hashToken(refreshToken);
    
    const storedToken = await this.db
      .selectFrom('refresh_tokens')
      .selectAll()
      .where('token_hash', '=', tokenHash)
      .where('expires_at', '>', new Date())
      .executeTakeFirst();

    if (!storedToken) {
      throw new UnauthorizedError('Invalid refresh token');
    }

    // 2. Load user and permissions
    const user = await this.db
      .selectFrom('users')
      .selectAll()
      .where('id', '=', storedToken.user_id)
      .executeTakeFirstOrThrow();

    if (!user.active) {
      throw new UnauthorizedError('User account inactive');
    }

    const permissions = await this.getUserPermissions(user.id);

    // 3. Generate new access token
    const accessToken = this.generateAccessToken({
      user_id: user.id,
      tenant_id: user.tenant_id,
      email: user.email,
      roles: permissions.roles,
      permissions: permissions.permissions
    });

    return { access_token: accessToken };
  }

  async logout(refreshToken: string): Promise<void> {
    const tokenHash = this.hashToken(refreshToken);
    
    await this.db
      .deleteFrom('refresh_tokens')
      .where('token_hash', '=', tokenHash)
      .execute();

    await this.logAuthEvent(null, 'logout', true);
  }

  private generateAccessToken(payload: JWTPayload): string {
    return jwt.sign(payload, this.JWT_SECRET, {
      expiresIn: this.JWT_EXPIRY,
      issuer: 'teraffi-platform'
    });
  }

  private async generateRefreshToken(userId: string): Promise<string> {
    const token = this.generateSecureToken();
    const tokenHash = this.hashToken(token);

    await this.db
      .insertInto('refresh_tokens')
      .values({
        user_id: userId,
        token_hash: tokenHash,
        expires_at: addDays(new Date(), this.REFRESH_TOKEN_EXPIRY_DAYS)
      })
      .execute();

    return token;
  }

  private async handleFailedLogin(userId: string): Promise<void> {
    const user = await this.db
      .selectFrom('users')
      .select(['failed_login_attempts'])
      .where('id', '=', userId)
      .executeTakeFirstOrThrow();

    const attempts = (user.failed_login_attempts || 0) + 1;

    if (attempts >= 5) {
      // Lock account for 30 minutes
      await this.db
        .updateTable('users')
        .set({
          failed_login_attempts: attempts,
          locked_until: addMinutes(new Date(), 30)
        })
        .where('id', '=', userId)
        .execute();

      await this.logAuthEvent(userId, 'account_locked', true);
    } else {
      await this.db
        .updateTable('users')
        .set({ failed_login_attempts: attempts })
        .where('id', '=', userId)
        .execute();
    }

    await this.logAuthEvent(userId, 'login', false, `Failed attempt ${attempts}`);
  }

  private validatePasswordStrength(password: string): void {
    if (password.length < 12) {
      throw new ValidationError('Password must be at least 12 characters');
    }
    
    const hasUppercase = /[A-Z]/.test(password);
    const hasLowercase = /[a-z]/.test(password);
    const hasNumber = /[0-9]/.test(password);
    const hasSpecial = /[^A-Za-z0-9]/.test(password);

    if (!hasUppercase || !hasLowercase || !hasNumber || !hasSpecial) {
      throw new ValidationError('Password must contain uppercase, lowercase, number, and special character');
    }
  }
}
```

### 3. Authorization Middleware

```typescript
// packages/auth-service/src/authorization.ts

export class AuthorizationService {
  async checkPermission(
    userId: string,
    permission: string,
    resourceId?: string
  ): Promise<boolean> {
    // 1. Load user permissions
    const userPermissions = await this.getUserPermissions(userId);

    // 2. Check if user has permission
    if (userPermissions.permissions.includes(permission)) {
      return true;
    }

    // 3. Check wildcard permissions
    const permissionParts = permission.split(':');
    for (let i = permissionParts.length; i > 0; i--) {
      const wildcardPermission = permissionParts.slice(0, i).join(':') + ':*';
      if (userPermissions.permissions.includes(wildcardPermission)) {
        return true;
      }
    }

    // 4. If resource-specific, check resource ownership
    if (resourceId) {
      return await this.checkResourceOwnership(userId, resourceId);
    }

    return false;
  }

  private async getUserPermissions(userId: string): Promise<{
    roles: string[];
    permissions: string[];
  }> {
    const result = await this.db
      .selectFrom('users')
      .innerJoin('user_roles', 'users.id', 'user_roles.user_id')
      .innerJoin('roles', 'user_roles.role_id', 'roles.id')
      .select(['roles.name', 'roles.permissions'])
      .where('users.id', '=', userId)
      .execute();

    const roles = result.map(r => r.name);
    const permissions = [...new Set(result.flatMap(r => r.permissions))];

    return { roles, permissions };
  }
}

// Middleware
export function requireAuth(req: FastifyRequest, reply: FastifyReply, done: () => void) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    throw new UnauthorizedError('Missing authentication token');
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
    req.user = decoded;
    done();
  } catch (error) {
    throw new UnauthorizedError('Invalid or expired token');
  }
}

export function requirePermission(permission: string) {
  return async (req: FastifyRequest, reply: FastifyReply, done: () => void) => {
    if (!req.user) {
      throw new UnauthorizedError('Not authenticated');
    }

    const hasPermission = req.user.permissions.includes(permission) ||
                         req.user.permissions.some(p => p.endsWith(':*') && permission.startsWith(p.replace(':*', '')));

    if (!hasPermission) {
      throw new ForbiddenError(`Missing permission: ${permission}`);
    }

    done();
  };
}
```

### 4. Permission System

```typescript
// Permission format: resource:action
// Examples:
// - partnerships:read
// - partnerships:write
// - partnerships:delete
// - analytics:view
// - admin:*

const SYSTEM_ROLES = {
  platform_admin: {
    name: 'Platform Admin',
    permissions: ['*:*']  // Full access
  },
  
  tenant_admin: {
    name: 'Tenant Admin',
    permissions: [
      'users:*',
      'roles:*',
      'partnerships:*',
      'analytics:*',
      'settings:*'
    ]
  },
  
  brand_executive: {
    name: 'Brand Executive',
    permissions: [
      'partnerships:read',
      'partnerships:write',
      'analytics:view',
      'deals:*'
    ]
  },
  
  creator: {
    name: 'Content Creator',
    permissions: [
      'partnerships:read',
      'partnerships:write',
      'deals:*',
      'content:*'
    ]
  },
  
  teraffi_activator: {
    name: 'TERAFFI Activator',
    permissions: [
      'partnerships:*',  // Cross-tenant access
      'deals:*',
      'analytics:view'
    ]
  },
  
  viewer: {
    name: 'Viewer',
    permissions: [
      'partnerships:read',
      'analytics:view'
    ]
  }
};
```

### 5. MFA Implementation

```typescript
// packages/auth-service/src/mfa.ts

export class MFAService {
  async enableMFA(userId: string): Promise<{ secret: string; qr_code: string }> {
    // 1. Generate secret
    const secret = authenticator.generateSecret();

    // 2. Get user email for QR code
    const user = await this.db
      .selectFrom('users')
      .select('email')
      .where('id', '=', userId)
      .executeTakeFirstOrThrow();

    // 3. Generate QR code
    const otpauth = authenticator.keyuri(user.email, 'TERAFFI', secret);
    const qrCode = await QRCode.toDataURL(otpauth);

    // 4. Generate recovery codes
    const recoveryCodes = this.generateRecoveryCodes(10);

    // 5. Store encrypted secret and recovery codes
    await this.db
      .updateTable('users')
      .set({
        mfa_secret: this.encrypt(secret),
        mfa_recovery_codes: recoveryCodes.map(c => this.encrypt(c)),
        mfa_enabled: false  // Not enabled until verified
      })
      .where('id', '=', userId)
      .execute();

    return {
      secret,
      qr_code: qrCode,
      recovery_codes: recoveryCodes
    };
  }

  async verifyAndEnableMFA(userId: string, code: string): Promise<void> {
    const user = await this.db
      .selectFrom('users')
      .select('mfa_secret')
      .where('id', '=', userId)
      .executeTakeFirstOrThrow();

    const valid = authenticator.verify({
      token: code,
      secret: this.decrypt(user.mfa_secret)
    });

    if (!valid) {
      throw new ValidationError('Invalid MFA code');
    }

    await this.db
      .updateTable('users')
      .set({ mfa_enabled: true })
      .where('id', '=', userId)
      .execute();
  }

  async disableMFA(userId: string, password: string): Promise<void> {
    // Require password confirmation
    const user = await this.db
      .selectFrom('users')
      .select('password_hash')
      .where('id', '=', userId)
      .executeTakeFirstOrThrow();

    const validPassword = await bcrypt.compare(password, user.password_hash);
    if (!validPassword) {
      throw new UnauthorizedError('Invalid password');
    }

    await this.db
      .updateTable('users')
      .set({
        mfa_enabled: false,
        mfa_secret: null,
        mfa_recovery_codes: null
      })
      .where('id', '=', userId)
      .execute();
  }

  private generateRecoveryCodes(count: number): string[] {
    const codes: string[] = [];
    for (let i = 0; i < count; i++) {
      codes.push(randomBytes(4).toString('hex').toUpperCase());
    }
    return codes;
  }
}
```

---

## Migration Path

### Phase 1: Basic Auth (Months 1-2)
- User schema and registration
- Email/password login
- JWT tokens
- Basic RBAC

### Phase 2: Enhanced Security (Months 3-4)
- MFA implementation
- Password reset flows
- Account lockout
- Audit logging

### Phase 3: Enterprise Features (Months 5-6)
- SSO integration (SAML/OAuth)
- API key management
- Advanced permissions
- Session management

### Phase 4: Compliance (Months 7-8)
- SOC 2 audit trail
- Password policies
- Session timeout enforcement
- Security monitoring

---

## Success Metrics

**Security:**
- Zero authentication bypasses
- <0.1% fraudulent login attempts succeed
- 100% of admin actions audited
- MFA adoption >50% for enterprise users

**Performance:**
- Authentication <100ms (p95)
- Authorization check <10ms
- JWT verification <5ms
- Support 10,000+ concurrent sessions

**User Experience:**
- <5% login failure rate (excluding wrong passwords)
- <1% support tickets related to auth
- SSO success rate >99%

---

## References

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)
- [NIST Password Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [SAML 2.0 Specification](http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html)

---
