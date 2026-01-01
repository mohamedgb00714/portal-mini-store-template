# AutoPlans Project

This project is managed with [autoplans.dev](https://autoplans.dev) for AI-assisted development.

## Project Type

**Portal: Mini Store Template** - Full-featured e-commerce platform with CMS, database, and authentication.

## Getting Started

1. Install dependencies: `pnpm install` or `npm install --legacy-peer-deps`
2. Set up environment variables (see `.env.example`)
3. Initialize database: `pnpm payload migrate` or `npm run payload migrate`
4. Start development: `pnpm run dev --port 32100` or `npm run dev -- --port 32100`

## Tech Stack

- **Framework**: Next.js (App Router)
- **CMS**: Payload CMS
- **Database**: Neon PostgreSQL (Serverless)
- **ORM**: Drizzle ORM
- **UI**: React.js + Tailwind CSS + Shadcn UI
- **Language**: TypeScript
- **Authentication**: Payload Auth

## Critical Components

### Neon Database
- Serverless PostgreSQL database
- Connection pooling enabled
- Requires `DATABASE_URL` environment variable

### Payload CMS
- Admin panel at `/admin`
- Collections: Products, Categories, Orders, Media, Users
- Rich content management

### E-commerce Features
- Product catalog with categories
- Shopping cart functionality
- Order management
- User authentication
- Payment integration ready

## AI Development Workflow

1. **Read Context**: Check `.autoplans/tasks.md` for current work
2. **Database Changes**: Always create migrations, never modify schema directly
3. **CMS Collections**: Update Payload config for new collections
4. **Follow Architecture**: Review `.autoplans/architecture.md` before major changes
5. **Update Tasks**: Use AutoPlans tools to track progress

## Available AutoPlans Tools

- `autoplans_list_tasks` - View all tasks
- `autoplans_create_task` - Add new tasks
- `autoplans_update_task` - Update task status/details
- `autoplans_bulk_update_tasks` - Update multiple tasks at once
- `autoplans_sync_project_to_repo` - Sync project files to repository

## Environment Setup

Required environment variables:
```
DATABASE_URL=          # Neon PostgreSQL connection string
PAYLOAD_SECRET=        # Secret for Payload CMS
NEXT_PUBLIC_SERVER_URL= # Your server URL
```

---
*This project uses AutoPlans.dev for AI-powered project management.*
