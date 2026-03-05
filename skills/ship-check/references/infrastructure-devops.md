# Infrastructure & DevOps Reference Checklist

Detailed checklist for the Infrastructure & DevOps Reviewer agent. Evaluate every applicable item against the target codebase.

---

## 1. Environment Configuration

### Environment Files
- [ ] `.env.example` exists with all required variables documented
- [ ] `.env.example` has descriptive comments for each variable
- [ ] `.env.example` uses placeholder values (not real secrets)
- [ ] `.env` / `.env.local` / `.env.production` are in `.gitignore`
- [ ] No `.env` files committed to git history
- [ ] Environment variables are validated at startup (fail fast if missing)

**Grep patterns:**
```
process\.env\.|import\.meta\.env\.                 # env var usage
z\.object.*env|env.*schema|createEnv              # env validation (e.g., @t3-oss/env)
dotenv|loadEnv                                     # env loading
```

### Configuration Management
- [ ] Separate config for development, staging, and production
- [ ] Database connection strings are environment-specific
- [ ] API endpoints are environment-specific (no hardcoded URLs)
- [ ] Feature flags are environment-aware

**Grep patterns:**
```
localhost:\d{4}|127\.0\.0\.1                       # hardcoded local URLs (check they're env-gated)
https?://[a-z].*\.(com|io|dev|app)                 # hardcoded production URLs
```

---

## 2. CI/CD Pipeline

### Pipeline Existence
- [ ] CI/CD configuration exists (GitHub Actions, GitLab CI, CircleCI, etc.)
- [ ] Pipeline runs on pull requests
- [ ] Pipeline runs on push to main/production branch

**File patterns to check:**
```
.github/workflows/*.yml                            # GitHub Actions
.gitlab-ci.yml                                     # GitLab CI
.circleci/config.yml                               # CircleCI
Jenkinsfile                                        # Jenkins
.travis.yml                                        # Travis CI
bitbucket-pipelines.yml                            # Bitbucket
vercel.json                                        # Vercel config
netlify.toml                                       # Netlify config
```

### Pipeline Steps
- [ ] **Lint**: Code linting runs (ESLint, Prettier, etc.)
- [ ] **Type check**: TypeScript compilation or type checking
- [ ] **Test**: Unit/integration tests run
- [ ] **Build**: Production build succeeds
- [ ] **Deploy**: Automated deployment to target environment
- [ ] **Notifications**: Build failures notify the team

**Grep patterns in CI config:**
```
lint|eslint|prettier                                # linting step
tsc|typecheck|type-check                            # type checking
test|jest|vitest|pytest|rspec                       # test step
build|next build|npm run build                      # build step
deploy|aws|vercel|netlify|fly                       # deploy step
```

### Pipeline Quality
- [ ] Pipeline caches dependencies (node_modules, pip cache)
- [ ] Pipeline uses specific versions (not `latest` tags)
- [ ] Secrets are in CI/CD secrets, not in config files
- [ ] Pipeline has reasonable timeout limits
- [ ] Branch protection requires CI to pass

---

## 3. Logging

### Structured Logging
- [ ] Structured logging library used (pino, winston, bunyan, structlog)
- [ ] Log levels are used appropriately (error, warn, info, debug)
- [ ] Log entries include context (request ID, user ID, action)
- [ ] No `console.log` in production code (or proper logger wrapping console)

**Grep patterns:**
```
pino|winston|bunyan|morgan|structlog               # logging libraries
logger\.(info|warn|error|debug)                    # structured log calls
console\.(log|warn|error|debug)                    # raw console (WARNING if in production paths)
requestId|correlationId|traceId                     # request tracing
```

### Log Hygiene
- [ ] No sensitive data in logs (passwords, tokens, full credit cards, PII)
- [ ] Error logs include stack traces
- [ ] Log volume is manageable (not logging every request body)
- [ ] Logs are shipped to a centralized service (if applicable)

**Grep patterns:**
```
log.*password|log.*token|log.*secret               # sensitive data in logs
JSON\.stringify.*req\.body.*log                     # request body logging
datadog|logflare|papertrail|logtail                # log shipping services
```

---

## 4. Monitoring & Alerting

### Error Tracking
- [ ] Error tracking service integrated (Sentry, Bugsnag, Datadog, etc.)
- [ ] Error tracking captures unhandled exceptions
- [ ] Error tracking includes user context (who was affected)
- [ ] Source maps uploaded for readable stack traces

**Grep patterns:**
```
@sentry/|Sentry\.init|DSN                          # Sentry
bugsnag|Bugsnag\.start                             # Bugsnag
datadog|dd-trace                                    # Datadog
newrelic|new-relic                                  # New Relic
logrocket|LogRocket\.init                          # LogRocket
```

### Health Checks
- [ ] Health check endpoint exists (`/health`, `/api/health`, `/healthz`)
- [ ] Health check verifies database connectivity
- [ ] Health check verifies critical service dependencies
- [ ] Health check returns structured response (not just 200 OK)

**Grep patterns:**
```
\/health|\/healthz|\/api\/health                    # health endpoints
health.*check|readiness|liveness                    # health check logic
```

### Uptime Monitoring
- [ ] External uptime monitoring configured (or at least documented)
- [ ] Alerting rules exist for downtime
- [ ] Performance monitoring tracks response times

---

## 5. Database Management

### Migrations
- [ ] Database migration system exists (Prisma Migrate, Drizzle Kit, Alembic, Active Record, etc.)
- [ ] Migrations are version-controlled
- [ ] Migration files are sequential and non-conflicting
- [ ] Rollback migrations exist or are possible

**Grep patterns:**
```
prisma migrate|drizzle-kit|migrate|migration       # migration tools
schema\.prisma|drizzle\.config                     # ORM config
alembic|knex.*migrate|sequelize.*migration          # migration frameworks
```

### Seed Data
- [ ] Seed script exists for development setup
- [ ] Seed data is clearly separate from production data
- [ ] Seed script is idempotent (safe to run multiple times)

**Grep patterns:**
```
seed|db:seed|prisma.*seed                          # seed scripts
```

### Connection Management
- [ ] Connection pooling configured (not opening new connection per request)
- [ ] Connection limits are reasonable for the deployment target
- [ ] Database URL uses connection pooler if applicable (e.g., PgBouncer, Supabase pooler)

**Grep patterns:**
```
pool|connectionLimit|max_connections               # connection pooling
pgbouncer|pooler|connection_limit                  # connection pooler
PrismaClient|new Pool|createPool                   # client instantiation (check singleton pattern)
```

### Backups
- [ ] Database backup strategy exists or is documented
- [ ] Backup restoration has been tested (or at least documented)

---

## 6. Deployment

### Deployment Configuration
- [ ] Deployment target is configured (Vercel, AWS, Fly.io, Railway, etc.)
- [ ] Build command produces optimized production build
- [ ] Start command runs production server (not development server)
- [ ] Environment variables are configured in deployment target

**Grep patterns:**
```
Dockerfile|docker-compose|\.dockerignore            # Docker deployment
fly\.toml|Procfile|railway\.json                    # PaaS config
vercel\.json|netlify\.toml                          # JAMstack deployment
Caddyfile|nginx\.conf                               # reverse proxy
pm2|ecosystem\.config                               # process management
```

### Rollback Capability
- [ ] Deployment supports instant rollback (previous version available)
- [ ] Database migrations are backwards-compatible (or have rollback)
- [ ] Zero-downtime deployment (rolling update, blue-green, canary)

### Environment Parity
- [ ] Development setup mirrors production (same database type, same services)
- [ ] Docker or similar tool ensures consistent environments
- [ ] Staging environment exists (or documented how to test pre-production)

---

## 7. Security Infrastructure

### SSL/TLS
- [ ] HTTPS enforced in production
- [ ] SSL certificates auto-renew (Let's Encrypt, managed by hosting provider)
- [ ] HSTS header enabled with reasonable max-age

### Secure Cookies
- [ ] Session cookies use `httpOnly` flag
- [ ] Session cookies use `secure` flag in production
- [ ] Session cookies use `sameSite` attribute
- [ ] Cookie domain is scoped appropriately

**Grep patterns:**
```
httpOnly|http_only                                  # httpOnly cookie flag
secure:\s*true|secure=true                          # secure cookie flag
sameSite|same_site                                  # SameSite attribute
cookie.*domain|domain.*cookie                       # cookie domain
```

---

## 8. Developer Experience

### Setup Documentation
- [ ] README has setup instructions
- [ ] Setup requires fewer than 5 manual steps
- [ ] `package.json` scripts cover common operations (dev, build, test, lint)
- [ ] Docker Compose for local services (DB, Redis, etc.) or clear setup instructions

### Code Quality Tooling
- [ ] Linter configured (ESLint, Ruff, RuboCop, etc.)
- [ ] Formatter configured (Prettier, Black, etc.)
- [ ] Pre-commit hooks (husky, lint-staged, pre-commit)
- [ ] TypeScript strict mode (if TypeScript project)

**Grep patterns:**
```
eslint|\.eslintrc|eslint\.config                    # ESLint
prettier|\.prettierrc                               # Prettier
husky|lint-staged|pre-commit                        # git hooks
strict.*true|strictNullChecks                       # TypeScript strict mode
```

---

## Scoring Guide

- **CRITICAL**: No CI/CD pipeline, no error tracking, secrets committed to git, database with no migration system, production using development server
- **WARNING**: Missing health check endpoint, no structured logging, CI doesn't run tests, no connection pooling, no rollback strategy, missing `.env.example`
- **GOOD**: CI/CD runs lint + test + build, error tracking integrated, structured logging, migrations work, `.env.example` documented, deployment configured
- **EXCELLENT**: Full CI/CD with deployment, health checks with dependency verification, centralized logging, uptime monitoring, automated rollbacks, staging environment, comprehensive developer setup docs
