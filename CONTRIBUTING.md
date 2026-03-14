# 🤝 Contributing to the Real Estate Fractional Ownership Platform

Thank you for your interest in contributing! This document covers everything you need to know to get started — whether you're fixing a bug, proposing a feature, or improving documentation.

---

## 📌 Table of Contents

- [Project Overview](#project-overview)
- [Repositories](#repositories)
- [Before You Start](#before-you-start)
- [Development Setup](#development-setup)
- [Branching Strategy](#branching-strategy)
- [Making a Contribution](#making-a-contribution)
- [Commit Message Convention](#commit-message-convention)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Code Standards](#code-standards)
- [Testing Requirements](#testing-requirements)
- [Issue Guidelines](#issue-guidelines)
- [Areas Where Help Is Needed](#areas-where-help-is-needed)
- [Code of Conduct](#code-of-conduct)

---

## Project Overview

This is a **hybrid fractional real estate investment platform** consisting of:

- A **NestJS backend** that manages ownership records, trading, and dividend logic
- A **Next.js frontend** that gives investors a dashboard, marketplace, and wallet
- **Stellar (USDC)** for financial settlement — dividends and trade payments

The two repos work together. Most features will require changes in both. Read the README for each repo before working on a feature:

- [`BACKEND_README.md`](./BACKEND_README.md)
- [`FRONTEND_README.md`](./FRONTEND_README.md)

---

## Repositories

| Repo | Description |
|---|---|
| `realestate-platform-backend` | NestJS API — share ledger, trading, dividends, Stellar |
| `realestate-platform-frontend` | Next.js app — investor dashboard, marketplace, wallet |

---

## Before You Start

1. **Check open issues** — your idea might already be tracked
2. **Open an issue first** for any non-trivial change so it can be discussed before you invest time building it
3. **One feature or fix per PR** — keep changes focused and reviewable
4. **Financial logic belongs in the backend** — never add financial calculations to the frontend

---

## Development Setup

### Backend

```bash
git clone https://github.com/your-org/realestate-platform-backend.git
cd realestate-platform-backend

cp .env.example .env
# Fill in local values (see README for variable descriptions)

npm install
npx prisma migrate dev
npm run start:dev
```

### Frontend

```bash
git clone https://github.com/your-org/realestate-platform-frontend.git
cd realestate-platform-frontend

cp .env.local.example .env.local
# Set NEXT_PUBLIC_API_URL=http://localhost:3001

npm install
npm run dev
```

### Running Both Together

Start the backend first (`localhost:3001`), then the frontend (`localhost:3000`). The frontend will connect to the backend automatically via the `NEXT_PUBLIC_API_URL` env variable.

---

## Branching Strategy

We use **GitHub Flow** with the following branch naming convention:

```
main              ← stable, production-ready
dev               ← integration branch for next release

Feature branches (cut from dev):
  feature/share-ledger-audit-log
  feature/order-book-ui

Bug fixes (cut from dev, or main if hotfix):
  fix/dividend-rounding-error
  fix/wallet-balance-display

Documentation:
  docs/stellar-integration-guide

Hotfixes (cut from main):
  hotfix/stellar-transaction-failure
```

**Never commit directly to `main` or `dev`.**

---

## Making a Contribution

1. Fork the repository and clone your fork
2. Cut a branch from `dev`
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b feature/your-feature-name
   ```
3. Write your code following the [Code Standards](#code-standards) below
4. Add or update tests
5. Run the test suite — it must pass before opening a PR
6. Push your branch and open a Pull Request against `dev`

---

## Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

**Types:**

| Type | When to use |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation changes only |
| `refactor` | Code change with no feature or fix |
| `test` | Adding or updating tests |
| `chore` | Build process, tooling, deps |
| `perf` | Performance improvement |

**Examples:**

```
feat(dividends): add proportional USDC payout calculation
fix(trading): resolve order matching off-by-one error
docs(stellar): add wallet generation guide
test(shares): add unit tests for share transfer edge cases
```

---

## Pull Request Guidelines

A good PR:

- **Has a clear title** using the commit convention above
- **Describes what changed and why** — not just what files were touched
- **Links the related issue** using `Closes #123` or `Relates to #123`
- **Is focused** — one feature or fix, not a bundle of unrelated changes
- **Includes tests** for any new logic
- **Does not break existing tests**
- **Updates documentation** if behavior changed

### PR Template

When opening a PR, please fill out the following:

```markdown
## What does this PR do?
<!-- Brief description of the change -->

## Why is this needed?
<!-- Link to issue or explain the motivation -->

## How was it tested?
<!-- Describe your testing approach -->

## Checklist
- [ ] Tests added or updated
- [ ] No financial logic added to the frontend
- [ ] `.env.example` updated if new env vars were added
- [ ] Docs updated if behavior changed
- [ ] PR targets the `dev` branch (not `main`)
```

---

## Code Standards

### Backend (NestJS / TypeScript)

- Follow the existing NestJS module structure — one folder per module
- Use **Prisma** for all database access — no raw SQL queries
- All service methods must have JSDoc comments for public-facing logic
- Avoid business logic in controllers — keep it in services
- Financial calculations must be handled with exact decimal math (use `decimal.js`)
- All Stellar transactions must be verified on-chain before updating the DB

```typescript
// ✅ Good
@Injectable()
export class DividendService {
  /** Calculates net payout per share after deducting management fees */
  async calculateNetPayout(propertyId: string): Promise<Decimal> { ... }
}

// ❌ Bad — business logic in controller
@Get('payout')
async getPayout(@Param('id') id: string) {
  const shares = await this.db.shares.findMany(...);
  const total = shares.reduce(...); // ← move this to a service
}
```

### Frontend (Next.js / TypeScript)

- All API calls go through the `services/` layer — never call `fetch` or `axios` directly from a component
- Use **React Query** for all server state — avoid `useEffect` for data fetching
- Use **Zustand** only for global client state (auth, UI preferences)
- No financial logic in the frontend — display values from the API as-is
- All user-facing strings should be in English; i18n support planned for Phase 4

```typescript
// ✅ Good — data fetching via React Query + service layer
const { data: portfolio } = useQuery({
  queryKey: ['portfolio'],
  queryFn: () => portfolioService.getHoldings()
});

// ❌ Bad — fetching directly in component
useEffect(() => {
  fetch('/api/portfolio').then(...);
}, []);
```

### General

- TypeScript strict mode is enabled — no `any` types unless absolutely necessary
- Run `npm run lint` before committing — PRs with lint errors won't be reviewed
- Run `npm run format` to auto-format with Prettier

---

## Testing Requirements

### Backend

- **Unit tests** required for all service methods with business logic
- **Integration tests** required for critical flows: share purchase, dividend calculation, trade settlement
- Use **Jest** (already configured)

```bash
npm run test          # unit tests
npm run test:e2e      # integration tests
npm run test:cov      # coverage report
```

Target: **>80% coverage** on `shares/`, `trading/`, and `dividends/` modules.

### Frontend

- **Component tests** for all interactive UI components
- Use **React Testing Library**
- Mock all API calls in tests — never hit a real backend in tests

```bash
npm run test
npm run test:coverage
```

---

## Issue Guidelines

### Reporting a Bug

Use the **Bug Report** template. Include:
- What you expected to happen
- What actually happened
- Steps to reproduce
- Environment (OS, Node version, browser if frontend)
- Relevant logs or screenshots

### Requesting a Feature

Use the **Feature Request** template. Include:
- The problem you're trying to solve
- Your proposed solution
- Any alternatives you considered
- Which phase this fits into (MVP, Marketplace, Stellar, etc.)

### Labels

| Label | Meaning |
|---|---|
| `good first issue` | Great for new contributors |
| `help wanted` | We'd especially welcome a PR here |
| `backend` | Backend repo work |
| `frontend` | Frontend repo work |
| `stellar` | Stellar integration related |
| `security` | Security-sensitive — discuss before PRing |
| `phase-1` / `phase-2` / `phase-3` | Development phase tag |

---

## Areas Where Help Is Needed

We'd especially welcome contributions in the following areas:

- **📊 Share Ledger** — audit log improvements, edge case handling for partial fills
- **💱 Trading Engine** — order matching algorithm optimization
- **🌐 Stellar Integration** — transaction retry logic, failed payment recovery
- **📱 Frontend** — mobile responsiveness, accessibility (WCAG 2.1)
- **🧪 Tests** — increasing coverage in trading and dividend modules
- **📖 Documentation** — API docs, architecture diagrams, developer guides
- **🔒 Security** — input validation, rate limiting, audit logging

Look for issues tagged `good first issue` if you're getting started.

---

## Code of Conduct

All contributors are expected to:

- Be respectful and constructive in all communications
- Welcome and support new contributors
- Focus feedback on code and ideas — not people
- Disclose any conflicts of interest (especially for financial logic changes)

Unacceptable behaviour will result in removal from the project. If you experience or witness a conduct issue, please contact the maintainers directly.

---

**Thank you for contributing.** Every PR, issue, test, and documentation improvement makes this platform better for investors everywhere. 🏡
