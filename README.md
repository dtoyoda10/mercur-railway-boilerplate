# MercurJS Railway Boilerplate

A ready-to-use multi-vendor marketplace built with MedusaJS and Next.js. Deploy to Railway in minutes.

## What's Included

This is a complete marketplace solution with:

- Multi-vendor marketplace with commission management
- Stripe Connect for vendor payments
- Next.js 15 storefront (App Router, i18n, Tailwind)
- Customized admin panel for vendors
- Real-time chat between customers and vendors
- Product search with Algolia
- Email notifications via Resend
- Monorepo setup with PNPM and Turbo
- CI/CD pipeline with GitHub Actions

## Quick Setup

```bash
git clone https://github.com/dtoyoda10/mercur-railway-boilerplate.git
cd mercur-railway-boilerplate
pnpm install

cp apps/backend/.env.template apps/backend/.env
cp apps/storefront/.env.template apps/storefront/.env
cp apps/vendor-panel/.env.template apps/vendor-panel/.env
# Edit the .env files with your config

pnpm run db:init
pnpm run db:admin
pnpm run dev
```

Access at http://localhost:8000 (storefront) and http://localhost:7000 (admin)

For detailed instructions, see [QUICKSTART.md](QUICKSTART.md)

## Documentation

**Getting Started**
- [Quickstart](QUICKSTART.md) - Get running in 5 minutes
- [Complete Setup Guide](docs/GETTING-STARTED.md) - Local development and Railway deployment
- [Environment Variables](docs/environment-variables.md) - All configuration options

**Integrations**
- [Stripe Setup](docs/guides/stripe-setup.md) - Payments and vendor payouts
- [Resend Setup](docs/guides/resend-setup.md) - Email notifications
- [Algolia Setup](docs/guides/algolia-setup.md) - Product search
- [TalkJS Setup](docs/guides/talkjs-setup.md) - Customer chat

**Operations**
- [Database Management](docs/database-management.md) - Migrations and backups
- [Security](docs/security.md) - Best practices
- [Monitoring](docs/monitoring.md) - Health checks and alerts
- [Troubleshooting](docs/troubleshooting.md) - Common issues

**Before Going Live**
- [Production Checklist](docs/deployment/production-checklist.md) - Pre-launch verification

## Requirements

Before you start:
- Node.js v20 or higher
- PNPM v10 or higher
- Docker (for local development)
- Git

For deployment you'll need accounts on:
- Railway (hosting)
- Stripe (payments)
- Resend (emails)
- Algolia (search)
- TalkJS (chat)

All have free tiers you can start with.

## Tech Stack

Backend: MedusaJS v2.7, PostgreSQL 16, Redis 7, TypeScript

Storefront: Next.js 15, Tailwind CSS, next-intl

Vendor Panel: Medusa Admin SDK, Vite, Tailwind CSS

DevOps: PNPM, Turbo, GitHub Actions, Railway, Docker

## Project Structure

```
mercur-railway-boilerplate/
├── apps/
│   ├── backend/              MedusaJS API server
│   ├── storefront/           Next.js customer-facing app
│   └── vendor-panel/         Admin dashboard for vendors
├── packages/
│   └── http-client/          Generated API client
├── docs/                     Documentation
├── .github/workflows/        CI/CD pipelines
└── docker-compose.yml        Local development services
```

## Common Commands

```bash
# Start all services
pnpm run dev

# Start individual services
pnpm run dev:api      # Backend only
pnpm run dev:vendor   # Vendor panel only
pnpm run dev:store    # Storefront only

# Database operations
pnpm run db:init      # Initialize database
pnpm run db:migrate   # Run migrations
pnpm run db:admin     # Create admin user

# Build for production
pnpm run build

# Run tests
pnpm --filter=backend run test

# Code quality
pnpm run lint
pnpm run format
```

## Deploying to Railway

We have a complete step-by-step guide in [docs/deployment/railway.md](docs/deployment/railway.md).

Quick version:
1. Fork this repository
2. Create a Railway project and connect your fork
3. Add PostgreSQL database
4. Create 3 services (backend, vendor panel, storefront)
5. Configure environment variables
6. Deploy

Check the [production checklist](docs/deployment/production-checklist.md) before launching.

## Environment Variables

The boilerplate needs several environment variables. Here's what's required:

**Backend (apps/backend/.env)**
```env
DATABASE_URL=postgres://...
JWT_SECRET=your-secret-here
COOKIE_SECRET=your-secret-here
STRIPE_SECRET_API_KEY=sk_...
RESEND_API_KEY=re_...
ALGOLIA_APP_ID=...
ALGOLIA_API_KEY=...
```

**Storefront (apps/storefront/.env)**
```env
MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_...
NEXT_PUBLIC_STRIPE_KEY=pk_...
```

**Vendor Panel (apps/vendor-panel/.env)**
```env
VITE_MEDUSA_BACKEND_URL=http://localhost:9000
```

See [docs/environment-variables.md](docs/environment-variables.md) for complete documentation.

## Troubleshooting

**Backend won't start**
- Make sure PostgreSQL is running: `docker ps`
- Check your DATABASE_URL in .env
- Verify migrations ran: `pnpm run db:migrate`

**Can't log in to admin**
- Create an admin user: `pnpm run db:admin`
- Check backend is running: `curl http://localhost:9000/health`

**Products not showing on storefront**
- Verify products are published in the admin panel
- Check NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY is set correctly

More solutions in [docs/troubleshooting.md](docs/troubleshooting.md).

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Quick steps:
1. Fork the repo
2. Create a feature branch
3. Make your changes
4. Run tests and linting
5. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) file.

## Support

- Documentation: [docs/](docs/)
- Issues: [GitHub Issues](https://github.com/dtoyoda10/mercur-railway-boilerplate/issues)
- Discussions: [GitHub Discussions](https://github.com/dtoyoda10/mercur-railway-boilerplate/discussions)

## Credits

Built with MedusaJS, Next.js, Railway, Stripe, and other open-source projects.
