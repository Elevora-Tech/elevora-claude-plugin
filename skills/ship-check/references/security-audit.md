# Security Audit Reference Checklist

Detailed checklist for the Security Auditor agent. Evaluate every applicable item against the target codebase.

---

## 1. Authentication & Authorization

### Authentication Patterns
- [ ] Auth middleware exists and is applied to protected routes
- [ ] Session management is secure (httpOnly, secure, sameSite cookies)
- [ ] JWT tokens have reasonable expiration (< 24h for access, < 30d for refresh)
- [ ] Password hashing uses bcrypt/scrypt/argon2 (not MD5/SHA1/SHA256)
- [ ] OAuth/social login callbacks validate state parameter
- [ ] Login rate limiting prevents brute force (account lockout or exponential backoff)
- [ ] Password reset tokens are single-use and time-limited
- [ ] Email verification exists for new accounts

**Grep patterns:**
```
bcrypt|argon2|scrypt                    # password hashing
md5|sha1|sha256.*password              # weak password hashing (CRITICAL)
jwt.sign|jsonwebtoken                  # JWT usage
expiresIn|exp:                         # token expiration
httpOnly|secure|sameSite               # cookie security flags
```

### Authorization
- [ ] Role-based or attribute-based access control exists
- [ ] API routes check user permissions, not just authentication
- [ ] Admin routes have separate middleware/guards
- [ ] Object-level authorization (users can only access their own data)
- [ ] No direct object references without ownership check (IDOR vulnerability)

**Grep patterns:**
```
role|permission|isAdmin|isOwner|authorize     # RBAC patterns
userId.*===.*session|req.user.id              # ownership checks
where.*userId|findFirst.*userId               # DB-level scoping
```

---

## 2. OWASP Top 10

### A01: Broken Access Control
- [ ] No missing auth middleware on sensitive routes
- [ ] API endpoints validate authorization, not just authentication
- [ ] No directory traversal in file handling (`../` in paths)
- [ ] CORS is configured with specific origins (not `*` in production)

**Grep patterns:**
```
cors\(|Access-Control-Allow-Origin     # CORS config
origin:\s*['"]\*['"]                   # wildcard CORS (WARNING)
\.\.\/|path\.join.*req\.(body|query|params)  # path traversal risk
```

### A02: Cryptographic Failures
- [ ] No secrets in source code (API keys, passwords, tokens)
- [ ] Sensitive data encrypted at rest if stored
- [ ] HTTPS enforced (no HTTP in production URLs)
- [ ] No sensitive data in URL query parameters or logs

**Grep patterns:**
```
sk_live|sk_test|AKIA|AIza              # API key patterns
password\s*[:=]\s*['"][^'"]+['"]       # hardcoded passwords
secret\s*[:=]\s*['"][^'"]+['"]         # hardcoded secrets
http://(?!localhost|127\.0\.0\.1)      # non-HTTPS URLs
```

### A03: Injection
- [ ] SQL queries use parameterized queries / prepared statements
- [ ] No string concatenation in SQL queries
- [ ] NoSQL queries sanitize user input
- [ ] Command injection: no `exec()`, `spawn()` with unsanitized user input
- [ ] No `eval()` with user-controlled input
- [ ] Template injection: no raw user input in template strings

**Grep patterns:**
```
\$\{.*req\.(body|query|params).*\}.*query|execute  # SQL injection risk
eval\(|new Function\(                               # eval usage
exec\(|execSync\(|spawn\(                           # command execution
dangerouslySetInnerHTML                              # XSS via React
innerHTML\s*=                                        # XSS via DOM
```

### A04: Insecure Design
- [ ] Business logic has validation (not just UI validation)
- [ ] Rate limiting on expensive operations
- [ ] No reliance on client-side-only validation for security

### A05: Security Misconfiguration
- [ ] Debug mode disabled in production
- [ ] Default credentials removed
- [ ] Error messages don't leak stack traces or internal details in production
- [ ] Unnecessary HTTP methods disabled
- [ ] Server version headers suppressed

**Grep patterns:**
```
DEBUG\s*=\s*True|NODE_ENV.*development  # debug mode
stack.*trace|\.stack\b.*res\.(json|send)  # stack trace leaks
x-powered-by                             # server info leak
```

### A06: Vulnerable Components
- [ ] No known critical CVEs in dependencies
- [ ] Dependencies are reasonably up to date
- [ ] Lock file exists (package-lock.json, yarn.lock, pnpm-lock.yaml)
- [ ] No deprecated packages with known security issues

### A07: Authentication Failures
- Covered in Authentication section above

### A08: Data Integrity Failures
- [ ] Webhook endpoints verify signatures
- [ ] No `JSON.parse()` of untrusted data without try-catch
- [ ] CI/CD pipeline integrity (no arbitrary script execution from PRs)

**Grep patterns:**
```
stripe.*webhook.*verify|constructEvent   # Stripe webhook verification
svix.*verify|webhook.*verify             # generic webhook verification
JSON\.parse\(                            # unprotected JSON parsing
```

### A09: Logging & Monitoring Failures
- [ ] Authentication events are logged (login, logout, failed attempts)
- [ ] Authorization failures are logged
- [ ] No sensitive data in logs (passwords, tokens, PII)
- [ ] Logs have enough context for incident response

**Grep patterns:**
```
console\.log.*password|console\.log.*token|console\.log.*secret  # sensitive data in logs
logger|winston|pino|bunyan                                        # structured logging
```

### A10: Server-Side Request Forgery (SSRF)
- [ ] User-supplied URLs are validated before server-side fetching
- [ ] No internal network access via user-controlled URLs
- [ ] Redirect URLs are validated against allowlist

**Grep patterns:**
```
fetch\(.*req\.(body|query|params)|axios\(.*req\.(body|query|params)  # SSRF risk
redirect\(.*req\.(body|query|params)                                   # open redirect
```

---

## 3. Secrets Management

- [ ] `.gitignore` includes `.env`, `.env.local`, `.env.production`
- [ ] No `.env` files committed to git (check git history)
- [ ] `.env.example` exists with placeholder values (no real secrets)
- [ ] Client-side bundle doesn't contain server secrets
- [ ] Environment variables use `NEXT_PUBLIC_` / `VITE_` prefix only for public values

**Grep patterns:**
```
process\.env\.((?!NEXT_PUBLIC_|VITE_|NODE_ENV).)*  # server env vars (check they're not client-exposed)
NEXT_PUBLIC_.*SECRET|NEXT_PUBLIC_.*KEY              # secrets exposed to client (CRITICAL)
VITE_.*SECRET|VITE_.*KEY                            # secrets exposed to client (CRITICAL)
```

---

## 4. HTTP Security Headers

Check for these headers in middleware, server config, or next.config.js:

- [ ] `Content-Security-Policy` (CSP) — prevents XSS
- [ ] `X-Content-Type-Options: nosniff` — prevents MIME sniffing
- [ ] `X-Frame-Options: DENY` or `SAMEORIGIN` — prevents clickjacking
- [ ] `Strict-Transport-Security` (HSTS) — enforces HTTPS
- [ ] `Referrer-Policy` — controls referrer leakage
- [ ] `Permissions-Policy` — restricts browser features

**Grep patterns:**
```
Content-Security-Policy|contentSecurityPolicy    # CSP
X-Content-Type-Options|nosniff                   # MIME type
X-Frame-Options|DENY|SAMEORIGIN                  # Clickjacking
Strict-Transport-Security|hsts                   # HSTS
Referrer-Policy                                  # Referrer
Permissions-Policy                               # Feature policy
headers\(\)|middleware.*headers                   # header configuration
```

---

## 5. Input Validation

- [ ] API routes validate request body with a schema (zod, joi, yup, class-validator)
- [ ] File uploads validate type, size, and sanitize filenames
- [ ] Pagination parameters are bounded (max limit)
- [ ] Search inputs are sanitized
- [ ] URL parameters are validated before use

**Grep patterns:**
```
z\.object|z\.string|z\.number          # zod schemas
Joi\.object|Joi\.string                # joi schemas
yup\.object|yup\.string                # yup schemas
@IsString|@IsNumber|@ValidateNested    # class-validator
multer|formidable|busboy               # file upload handling
```

---

## 6. Rate Limiting

- [ ] Rate limiting on authentication endpoints
- [ ] Rate limiting on API endpoints
- [ ] Rate limiting on file upload endpoints
- [ ] Rate limiting uses appropriate window and max values

**Grep patterns:**
```
rateLimit|rate-limit|rateLimiter        # rate limiting library
upstash.*ratelimit|@upstash/ratelimit   # Upstash rate limiting
throttle|limiter                         # throttling
```

---

## 7. Dependency Security

- [ ] Run `npm audit` / `yarn audit` / `pnpm audit` mentally — check for patterns of outdated deps
- [ ] No `*` version ranges in package.json
- [ ] Lockfile is committed
- [ ] No packages with known major vulnerabilities in common use (check package names against known issues)

---

## Scoring Guide

- **CRITICAL**: Any finding from A01-A03 (access control, crypto, injection), hardcoded secrets, missing auth on sensitive routes, wildcard CORS in production
- **WARNING**: Missing security headers, no rate limiting, outdated dependencies, missing input validation on some routes
- **GOOD**: Auth implemented correctly, input validation present, secrets properly managed, basic security headers
- **EXCELLENT**: All OWASP items addressed, CSP in enforce mode, webhook verification, comprehensive rate limiting, regular dependency updates
