# Contributing to MercurJS Railway Boilerplate

Thanks for considering contributing! This is a community project and we welcome improvements, bug fixes, and new features.

## How to Contribute

### Reporting Bugs

Found a bug? Here's how to report it:

1. **Check existing issues** first - someone might have already reported it
2. **Create a new issue** with:
   - Clear title describing the bug
   - Steps to reproduce
   - Expected vs actual behavior
   - Your environment (Node version, OS, etc.)
   - Screenshots if relevant

**Example:**
```
Title: Backend crashes when creating product without price

Steps to reproduce:
1. Log in to vendor panel
2. Go to Products → Create Product
3. Fill in title and description
4. Save without adding a price
5. Backend throws 500 error

Expected: Validation error shown to user
Actual: Server crashes with "Cannot read property 'amount' of undefined"

Environment: Node 20.11.0, macOS, PostgreSQL 16
```

### Suggesting Features

Have an idea? Open an issue with:

- What problem it solves
- How it would work
- Why it's useful for the boilerplate

We prioritize features that benefit most users of a marketplace boilerplate.

### Pull Requests

Ready to code? Here's the workflow:

## Development Setup

### 1. Fork and Clone

```bash
# Fork the repo on GitHub, then:
git clone https://github.com/YOUR_USERNAME/mercur-railway-boilerplate.git
cd mercur-railway-boilerplate
```

### 2. Install Dependencies

```bash
pnpm install
```

**Note:** This project uses **pnpm**, not npm or yarn. Install it with:
```bash
npm install -g pnpm
```

### 3. Set Up Local Environment

```bash
# Copy environment templates
cp apps/backend/.env.template apps/backend/.env
cp apps/storefront/.env.template apps/storefront/.env
cp apps/vendor-panel/.env.template apps/vendor-panel/.env

# Start database and Redis with Docker
docker-compose up -d

# Run migrations
pnpm run db:init

# Create an admin user
pnpm run db:admin
```

### 4. Start Development Servers

Open 3 terminal windows:

**Terminal 1 - Backend:**
```bash
cd apps/backend
pnpm run dev
# Runs on http://localhost:9000
```

**Terminal 2 - Vendor Panel:**
```bash
cd apps/vendor-panel
pnpm run dev
# Runs on http://localhost:7000
```

**Terminal 3 - Storefront:**
```bash
cd apps/storefront
pnpm run dev
# Runs on http://localhost:8000
```

**Or use Turbo to run all at once:**
```bash
pnpm run dev
```

## Making Changes

### Branch Naming

Create a branch for your changes:

```bash
git checkout -b feature/add-product-reviews
git checkout -b fix/cart-price-calculation
git checkout -b docs/improve-setup-guide
```

Use prefixes:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation updates
- `refactor/` - Code refactoring
- `test/` - Adding tests
- `chore/` - Maintenance tasks

### Code Style

We use ESLint and Prettier to keep code consistent.

**Before committing:**
```bash
# Format code
pnpm run format

# Check for linting errors
pnpm run lint

# Build to catch TypeScript errors
pnpm run build
```

**Pro tip:** Set up your editor to format on save:

**VSCode** - Install Prettier extension and add to `.vscode/settings.json`:
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

### Commit Messages

Write clear commit messages:

**Good:**
```
fix: prevent crash when product has no variants
feat: add bulk delete for products in vendor panel
docs: update Railway deployment guide with new UI
```

**Not so good:**
```
fixed bug
updates
changes
```

**Format:** `type: description`

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `style` - Code style (formatting, no logic change)
- `refactor` - Code restructuring
- `test` - Adding tests
- `chore` - Maintenance

### Testing

We're working on improving test coverage. If you're adding a feature:

**Backend:**
```bash
cd apps/backend
pnpm test
```

**Add tests for:**
- New API endpoints
- Business logic
- Database operations
- Integrations (Stripe, Algolia, etc.)

Tests go in `apps/backend/integration-tests/http/`

### Documentation

If your change affects how users set up or use the boilerplate, update the docs:

- `README.md` - Main setup guide
- `docs/deployment/railway.md` - Deployment instructions
- `docs/environment-variables.md` - New env vars
- `docs/guides/` - Integration guides

## Submitting a Pull Request

### 1. Push Your Branch

```bash
git push origin feature/your-feature-name
```

### 2. Create Pull Request

1. Go to GitHub and create a PR from your branch
2. Fill in the PR template (if we have one)
3. Describe what you changed and why
4. Link related issues (e.g., "Fixes #123")

**Good PR description:**
```markdown
## What

Adds product reviews functionality to the marketplace.

## Why

Customers need to see reviews before buying from vendors.

## Changes

- Added Review model and API endpoints
- Created review UI in storefront
- Added review management to vendor panel
- Updated docs with review setup

## Testing

- [ ] Tested creating reviews as customer
- [ ] Tested vendor can see and respond to reviews
- [ ] Tested average rating calculation
- [ ] Added unit tests for review endpoints

Fixes #456
```

### 3. Wait for Review

A maintainer will review your PR. They might:

- ✅ Approve and merge
- 💬 Request changes
- ❌ Close if it doesn't fit the project

**If changes are requested:**
1. Make the changes
2. Push to the same branch
3. PR updates automatically

### 4. After Merge

Once merged:
```bash
# Switch back to main
git checkout main

# Pull latest changes
git pull upstream main

# Delete your feature branch
git branch -d feature/your-feature-name
```

## Project Structure

Here's where things live:

```
mercur-railway-boilerplate/
├── apps/
│   ├── backend/              # MedusaJS API
│   │   ├── src/
│   │   │   ├── api/          # Custom API routes
│   │   │   ├── modules/      # Custom Medusa modules
│   │   │   ├── workflows/    # Business logic workflows
│   │   │   ├── jobs/         # Background jobs
│   │   │   └── subscribers/  # Event subscribers
│   │   └── integration-tests/ # API tests
│   │
│   ├── storefront/           # Next.js customer app
│   │   └── src/
│   │       ├── app/          # App router pages
│   │       ├── components/   # React components
│   │       └── lib/          # Utilities
│   │
│   └── vendor-panel/         # Medusa admin
│       └── src/
│           ├── routes/       # Admin pages
│           └── components/   # Admin UI components
│
├── docs/                     # Documentation
├── packages/                 # Shared packages
└── .github/                  # CI/CD workflows
```

### Common Tasks

**Add new backend module:**
```bash
cd apps/backend/src/modules
mkdir my-module
# Follow Medusa module structure
```

**Add new API endpoint:**
```bash
cd apps/backend/src/api
# Create route file following Medusa conventions
```

**Add new storefront page:**
```bash
cd apps/storefront/src/app
# Create new route folder (Next.js App Router)
```

**Add vendor panel page:**
```bash
cd apps/vendor-panel/src/routes
# Create new route component
```

## Code of Conduct

### Be Nice

- Be respectful and constructive
- No harassment, discrimination, or trolling
- Assume good intentions
- Give helpful feedback

### Be Patient

- Maintainers are volunteers
- Reviews take time
- Some PRs might not get merged (and that's okay)

## Getting Help

Stuck? Here's how to get help:

1. **Check the docs** - `docs/` folder
2. **Search existing issues** - Someone might have had the same problem
3. **Ask in Discussions** - GitHub Discussions tab
4. **Open an issue** - If you think it's a bug

## Recognition

Contributors are awesome! We'll add you to:
- `CONTRIBUTORS.md` (if we create one)
- GitHub contributors page (automatic)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

## Quick Reference

**Setup:**
```bash
pnpm install
docker-compose up -d
pnpm run db:init
pnpm run dev
```

**Before committing:**
```bash
pnpm run lint
pnpm run format
pnpm run build
```

**Run tests:**
```bash
pnpm --filter=backend run test
```

**Update docs if:**
- Adding new features
- Changing setup process
- Adding environment variables
- Changing API endpoints

---

Thanks for contributing! 🎉
