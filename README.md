# Mercur Railway Boilerplate



**Serverless MedusaJS E-commerce Made Easy on Railway**

Mercur is a powerful, opinionated boilerplate designed to jumpstart your headless e-commerce project with MedusaJS, optimized for seamless serverless deployment on Railway. It provides a pre-configured monorepo structure with PNPM workspaces, allowing you to manage your Medusa API, Admin Dashboard, and Storefront as interconnected but independently deployable services.

This boilerplate significantly reduces setup time, letting you focus on building features rather than wrestling with infrastructure.



## Features

* **MedusaJS Backend API:** The core headless commerce engine, ready for customization.
* **Medusa Admin Dashboard (Vendor):** A pre-configured frontend for managing your store data.
* **Next.js Storefront (Customer-facing):** A modern, performant e-commerce frontend example.
* **Monorepo Structure (PNPM Workspaces):** Efficiently manage multiple interdependent applications in one repository.
* **PostgreSQL Integration:** Seamlessly connect to a robust relational database (e.g., Railway PostgreSQL plugin).
* **S3-compatible File Storage (Optional):** Easy integration for product images and assets (e.g., AWS S3, Cloudflare R2).
* **Redis Integration (Optional):** Ready for caching and job queues to enhance performance.
* **Nixpacks for Zero-Config Deployment:** Optimized build system automatically detected by Railway, no Dockerfiles needed.
* **Preconfigured Environment Files:** Simplified setup for local development and deployment.



## Prerequisites

Before you begin, ensure you have the following installed:

* **Node.js:** v18+ (LTS recommended)
* **PNPM:** v8+ (`npm install -g pnpm`)
* **Git:** For cloning the repository.
* **Railway Account:** Sign up at [railway.app](https://railway.app/).
* **GitHub Account:** For seamless deployment integration with Railway.



## Getting Started (Local Development)

Follow these steps to get Mercur up and running on your local machine.

### 1. Clone the Repository

```bash
git clone [https://github.com/dtoyoda10/mercur-railway-boilerplate.git](https://github.com/dtoyoda10/mercur-railway-boilerplate.git)
cd mercur-railway-boilerplate
````

### 2\. Install Dependencies

Mercur uses PNPM for efficient dependency management:

```bash
pnpm install
```

### 3\. Environment Variables

Create `.env` files in the root of each `apps/` directory and configure them.

#### `apps/backend/.env`

```env
DATABASE_URL=postgres://user:password@localhost:5432/medusa_db # Your local PostgreSQL connection string
NODE_ENV=development
JWT_SECRET=your_jwt_secret_here # Must be a strong, random string
COOKIE_SECRET=your_cookie_secret_here # Must be a strong, random string
MEDUSA_ADMIN_CORS=http://localhost:7000,http://localhost:8000 # Admin and Storefront URLs for CORS
MEDUSA_STORE_CORS=http://localhost:8000,http://localhost:7000 # Storefront and Admin URLs for CORS

# Optional Redis Configuration (for caching/jobs)
REDIS_URL=redis://localhost:6379
```

#### `apps/vendor/.env` (Medusa Admin)

```env
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000 # Your local Medusa backend URL
```

#### `apps/store/.env` (Next.js Storefront)

```env
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000 # Your local Medusa backend URL
```

### 4\. Database Setup

Ensure you have a local PostgreSQL server running. Then, initialize your database:

```bash
# Run migrations (creates tables)
pnpm run db:init

# Create an admin user (interactive prompt)
pnpm run db:admin
```

### 5\. Run Services Locally

Open three separate terminal windows and run each application:

#### Run Backend API

```bash
pnpm run dev:api
# Runs on http://localhost:9000
```

#### Run Medusa Admin Dashboard

```bash
pnpm run dev:vendor
# Runs on http://localhost:7000
```

#### Run Next.js Storefront

```bash
pnpm run dev:store
# Runs on http://localhost:8000
```

You should now have your MedusaJS backend, admin panel, and storefront running locally\!



## Deployment to Railway

Mercur is pre-configured for a smooth, multi-service deployment on Railway using Nixpacks.

For a detailed, step-by-step guide on deploying this boilerplate to Railway, including service configuration (Backend, Vendor, Storefront) and environment variable setup, please refer to my comprehensive blog post:

**[How to Deploy Mercur to Railway: A Professional Guide for Serverless MedusaJS](https://dtoyoda10.medium.com/how-to-deploy-mercur-to-railway-a-professional-guide-for-serverless-medusajs-5a30f8e3dcdb)**

**Key deployment steps covered in the guide:**

1.  **Clone/Fork** the repository to your GitHub.
2.  **Create a New Railway Project** and connect it to your GitHub repo.
3.  **Set up three distinct services** (Backend, Vendor, Storefront) within your Railway project, each with specific "Watch Paths", "Build Commands", and "Start Commands".
4.  **Add a PostgreSQL Plugin** to your Railway project.
5.  **Configure Environment Variables** for each service in Railway (ensuring `DATABASE_URL`, `JWT_SECRET`, `COOKIE_SECRET`, `CORS` origins, and optional S3/Redis settings are correct).
6.  Trigger deployment and verify your live API endpoints.



## Project Structure

This boilerplate adopts a monorepo structure to streamline development and deployment of multiple related applications.

```
.
├── apps/
│   ├── backend/   # MedusaJS API backend
│   ├── vendor/    # Medusa Admin Dashboard (Next.js/Medusa Admin)
│   └── store/     # Next.js customer-facing storefront
├── data/          # Contains seed.json for database seeding
├── packages/      # Optional: For shared utilities, UI components, etc.
│   └── ui/
└── package.json   # Root PNPM workspace configuration
```



## Contributing

Contributions are welcome\! If you have suggestions, bug reports, or want to contribute code, please open an issue or pull request.
