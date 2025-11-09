# MercurJS Railway Boilerplate

Production-ready multi-vendor marketplace built with MedusaJS, Next.js, and optimized for Railway deployment.

## What You Get

This isn't just a starter template - it's a complete marketplace solution:

- ✅ **Multi-vendor marketplace** with commission rules and payouts
- ✅ **Stripe Connect** for split payments
- ✅ **Next.js 15 storefront** with App Router, i18n, and Tailwind
- ✅ **Customized admin panel** for vendors
- ✅ **Real-time chat** between customers and vendors (TalkJS)
- ✅ **Fast product search** with Algolia
- ✅ **Email notifications** via Resend
- ✅ **Monorepo** with PNPM workspaces + Turbo
- ✅ **CI/CD pipeline** ready to go
- ✅ **Complete docs** for deployment and operations

**Clone it. Configure it. Deploy it. You can be live in a day.**

## Quick Start

```bash
# Clone the repo
git clone https://github.com/dtoyoda10/mercur-railway-boilerplate.git
cd mercur-railway-boilerplate

# Install dependencies
pnpm install

# Start database & Redis
docker-compose up -d

# Set up environment variables (see docs/environment-variables.md)
cp apps/backend/.env.template apps/backend/.env
cp apps/storefront/.env.template apps/storefront/.env
cp apps/vendor-panel/.env.template apps/vendor-panel/.env
# Edit each .env file with your values

# Initialize database
pnpm run db:init
pnpm run db:admin  # Create admin user

# Start all services
pnpm run dev
```

**Access:**
- Backend API: http://localhost:9000
- Vendor Panel: http://localhost:7000
- Storefront: http://localhost:8000

## Documentation

### Getting Started
- 📘 [Environment Variables](docs/environment-variables.md) - Complete reference for all configuration
- 🚀 [Railway Deployment Guide](docs/deployment/railway.md) - Step-by-step production deployment
- 🐳 [Local Development Setup](CONTRIBUTING.md) - Detailed setup instructions

### Integration Guides
- 💳 [Stripe Setup](docs/guides/stripe-setup.md) - Configure payments and Connect
- 📧 [Resend Setup](docs/guides/resend-setup.md) - Email notifications
- 🔍 [Algolia Setup](docs/guides/algolia-setup.md) - Product search
- 💬 [TalkJS Setup](docs/guides/talkjs-setup.md) - Vendor-customer messaging

### Operations
- 🗄️ [Database Management](docs/database-management.md) - Migrations, backups, maintenance
- 🔐 [Security Best Practices](docs/security.md) - Keep your marketplace secure
- 📊 [Monitoring & Health Checks](docs/monitoring.md) - Stay ahead of issues
- 🛠️ [Troubleshooting Guide](docs/troubleshooting.md) - Common issues and fixes

### Deployment
- ✅ [Production Checklist](docs/deployment/production-checklist.md) - Don't go live without this
- 🤝 [Contributing Guide](CONTRIBUTING.md) - How to contribute

## Tech Stack

**Backend:**
- MedusaJS v2.7 - Headless commerce engine
- PostgreSQL 16 - Database
- Redis 7 - Caching & job queues
- TypeScript - Type safety

**Storefront:**
- Next.js 15 - React framework with App Router
- Tailwind CSS - Styling
- next-intl - Internationalization
- Storybook - Component documentation

**Vendor Panel:**
- Medusa Admin SDK - Customized admin UI
- Vite - Fast builds
- Tailwind CSS - Consistent styling

**Integrations:**
- Stripe Connect - Multi-party payments
- Resend - Transactional emails
- Algolia - Product search
- TalkJS - Real-time chat
- Cloudflare R2/AWS S3 - File storage (optional)

**DevOps:**
- PNPM - Fast, disk-efficient package manager
- Turbo - Monorepo task runner with caching
- GitHub Actions - CI/CD pipeline
- Railway - Serverless deployment platform
- Docker - Local development services

## Features

### For Marketplace Owners
- Commission rules and automatic splits
- Vendor approval workflow
- Analytics and reporting
- Product moderation
- Order management across all vendors

### For Vendors
- Dedicated dashboard
- Product management
- Order fulfillment
- Payout tracking
- Customer messaging
- Sales analytics

### For Customers
- Fast product search
- Multiple payment methods
- Order tracking
- Vendor reviews
- Wishlist
- Multi-language support

## Project Structure

```
mercur-railway-boilerplate/
├── apps/
│   ├── backend/              # MedusaJS API
│   │   ├── src/
│   │   │   ├── api/          # Custom REST endpoints
│   │   │   ├── modules/      # Custom modules (seller, reviews, etc.)
│   │   │   ├── workflows/    # Business logic workflows
│   │   │   └── subscribers/  # Event handlers
│   │   ├── integration-tests/ # API tests
│   │   └── Dockerfile        # Production Docker image
│   │
│   ├── storefront/           # Next.js customer app
│   │   └── src/
│   │       ├── app/          # App Router pages
│   │       └── components/   # Organized by atomic design
│   │
│   └── vendor-panel/         # Medusa Admin
│       └── src/
│           ├── routes/       # Custom admin pages
│           └── extensions/   # Admin UI widgets
│
├── packages/
│   └── http-client/          # Auto-generated API client
│
├── docs/                     # Complete documentation
├── .github/workflows/        # CI/CD pipelines
├── docker-compose.yml        # Local development services
├── turbo.json               # Monorepo task config
└── railway.toml             # Railway deployment config
```

## Prerequisites

Before you start:

- **Node.js** v20+ ([download](https://nodejs.org/))
- **PNPM** v10+ (`npm install -g pnpm`)
- **Docker** (for local PostgreSQL/Redis)
- **Git**

For deployment:
- **Railway account** ([sign up](https://railway.app/))
- **Stripe account** (for payments)
- **Resend account** (for emails)
- **Algolia account** (for search)
- **TalkJS account** (for chat)

All have free tiers to get started.

## Development Workflow

```bash
# Install dependencies
pnpm install

# Start local services (PostgreSQL, Redis)
docker-compose up -d

# Run database migrations
pnpm run db:init

# Start all apps in dev mode
pnpm run dev

# Or start individually:
pnpm run dev:api      # Backend only
pnpm run dev:vendor   # Vendor panel only
pnpm run dev:store    # Storefront only

# Build for production
pnpm run build

# Run tests
pnpm --filter=backend run test

# Lint and format
pnpm run lint
pnpm run format
```

## Deploying to Production

Follow our [complete Railway deployment guide](docs/deployment/railway.md) for step-by-step instructions.

**Quick summary:**

1. Fork this repository
2. Create Railway project and connect your repo
3. Add PostgreSQL database
4. Create 3 services (backend, vendor, storefront)
5. Configure environment variables for each service
6. Deploy!

See [production checklist](docs/deployment/production-checklist.md) before going live.

## Environment Variables

This boilerplate uses a lot of environment variables. We've documented every single one.

**Quick reference:**

```env
# Backend essentials
DATABASE_URL=postgres://...
JWT_SECRET=<generate with: openssl rand -base64 32>
COOKIE_SECRET=<generate with: openssl rand -base64 32>
STRIPE_SECRET_API_KEY=sk_...
RESEND_API_KEY=re_...

# Storefront essentials
MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_...
NEXT_PUBLIC_STRIPE_KEY=pk_...

# Vendor panel essentials
VITE_MEDUSA_BACKEND_URL=http://localhost:9000
```

**See [complete environment variables guide](docs/environment-variables.md) for all variables and configuration.**

## Common Tasks

### Add a New Product
1. Log in to vendor panel (http://localhost:7000)
2. Products → Create Product
3. Add title, description, price, images
4. Publish

### Create a Test Order
1. Visit storefront (http://localhost:8000)
2. Add product to cart
3. Checkout with Stripe test card: `4242 4242 4242 4242`

### Run Database Migrations
```bash
pnpm run db:migrate
```

### Seed Test Data
```bash
cd apps/backend
npm run seed
```

This creates sample products, categories, and a test seller account.

### Update Dependencies
```bash
pnpm update
pnpm run build  # Make sure nothing broke
```

## Troubleshooting

**Backend won't start?**
- Check DATABASE_URL is correct
- Make sure PostgreSQL is running: `docker ps | grep postgres`
- Verify .env file exists: `ls apps/backend/.env`

**Can't log in to admin?**
- Create admin user: `pnpm run db:admin`
- Check backend is running: `curl http://localhost:9000/health`

**Products not showing?**
- Check they're published (not draft)
- Verify publishable key is set in storefront

**More issues?** See our [complete troubleshooting guide](docs/troubleshooting.md).

## Contributing

We welcome contributions! Please read our [contributing guide](CONTRIBUTING.md) first.

**Quick start:**
1. Fork the repo
2. Create a branch: `git checkout -b feature/my-feature`
3. Make your changes
4. Run tests and linting
5. Open a pull request

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Support

- 📖 **Documentation:** [docs/](docs/)
- 🐛 **Bug Reports:** [GitHub Issues](https://github.com/dtoyoda10/mercur-railway-boilerplate/issues)
- 💬 **Questions:** [GitHub Discussions](https://github.com/dtoyoda10/mercur-railway-boilerplate/discussions)
- 🌐 **MedusaJS Community:** [Discord](https://discord.gg/medusajs)

## Roadmap

Planned improvements:

- [ ] E2E testing with Playwright
- [ ] Vendor onboarding wizard
- [ ] Advanced analytics dashboard
- [ ] Multi-currency support
- [ ] Mobile app (React Native)
- [ ] Marketplace templates/themes

## Acknowledgments

Built with:
- [MedusaJS](https://medusajs.com) - The headless commerce platform
- [Next.js](https://nextjs.org) - The React framework
- [Railway](https://railway.app) - The deployment platform
- [Stripe](https://stripe.com) - Payment processing
- And many other amazing open-source projects

---

**Ready to build your marketplace?** [Deploy to Railway](docs/deployment/railway.md) →

**Questions?** Open an [issue](https://github.com/dtoyoda10/mercur-railway-boilerplate/issues) or [discussion](https://github.com/dtoyoda10/mercur-railway-boilerplate/discussions)

**Star ⭐ this repo if you find it useful!**
