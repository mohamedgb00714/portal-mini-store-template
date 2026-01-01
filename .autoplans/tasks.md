# Project Tasks

This file tracks all tasks for the Portal Mini Store project.

## Task Organization

Tasks are managed through AutoPlans.dev and can be viewed/updated using:
- `autoplans_list_tasks({projectId})` - List all tasks
- `autoplans_create_task({projectId, title, description, priority, type})` - Create new task
- `autoplans_update_task({taskId, status, priority, ...})` - Update existing task

## Initial Setup Tasks

### Not Started
- [ ] Configure Neon database connection
- [ ] Set up Payload CMS admin account
- [ ] Configure Stripe/payment gateway
- [ ] Add product categories
- [ ] Create sample products
- [ ] Set up email service (orders, notifications)
- [ ] Configure media storage (Uploadthing/S3)
- [ ] Customize storefront theme

### Database Setup
- [ ] Run initial migrations
- [ ] Seed sample data
- [ ] Configure database backups

### Template Setup Complete
- [x] Next.js + Payload CMS initialized
- [x] Neon PostgreSQL configured
- [x] Drizzle ORM setup
- [x] Base collections created (Products, Categories, Orders, Users)
- [x] Tailwind CSS configured
- [x] Authentication system ready

## Development Guidelines

### Database Changes
- **Always create migrations**: `pnpm payload migrate:create`
- **Never edit schema directly**: Use Drizzle schema files
- **Test migrations**: Run `pnpm payload migrate` before pushing

### CMS Collections
- Define in `src/payload/collections/`
- Update `payload.config.ts` to register new collections
- Use access control for sensitive data

### Task Management
When creating tasks:
- Use descriptive titles
- Include acceptance criteria
- Set priority: `low`, `medium`, `high`, or `critical`
- Choose type: `coding`, `design`, `documentation`, `testing`, or `other`

## Subtasks

Individual task subtasks are stored in `.autoplans/tasks/{task-id}.md`

---
*Last Updated: Auto-generated on project creation*
