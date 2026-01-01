# Portal Mini Store Architecture

## Project Overview

Full-featured e-commerce platform combining Next.js, Payload CMS, and Neon PostgreSQL for a scalable mini store solution.

## Technology Stack

### Core Framework
- **Next.js**: App Router with React Server Components
- **Payload CMS**: Headless CMS for content and data management
- **Neon PostgreSQL**: Serverless PostgreSQL database
- **Drizzle ORM**: Type-safe database queries
- **TypeScript**: End-to-end type safety

### Frontend
- **React**: Component-based UI
- **Tailwind CSS**: Utility-first styling
- **Shadcn UI**: Pre-built accessible components
- **React Hook Form**: Form management

### Backend
- **Payload CMS**: Admin panel and API
- **Next.js API Routes**: Custom endpoints
- **Payload Auth**: User authentication
- **Uploadthing/S3**: Media storage

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Browser                        │
├─────────────────────────────────────────────────────────────┤
│                      Next.js Frontend                        │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐   │
│  │   Storefront │  │  Shopping    │  │  User Account   │   │
│  │   Pages      │  │  Cart        │  │  Dashboard      │   │
│  └──────────────┘  └──────────────┘  └─────────────────┘   │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│                    Next.js API Layer                         │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Payload CMS Admin                       │   │
│  │  /admin - Content Management Interface              │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Payload API                             │   │
│  │  /api - RESTful endpoints for collections           │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│                   Data Layer                                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │          Neon PostgreSQL Database                    │   │
│  │  - Products     - Media                              │   │
│  │  - Categories   - Users                              │   │
│  │  - Orders       - payload_preferences                │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Folder Structure

```
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (storefront)/      # Public-facing store
│   │   │   ├── page.tsx       # Homepage
│   │   │   ├── products/      # Product listings
│   │   │   └── cart/          # Shopping cart
│   │   ├── (admin)/           # Admin routes (protected)
│   │   └── api/               # API routes
│   ├── payload/
│   │   ├── collections/       # Payload collections
│   │   │   ├── Products.ts
│   │   │   ├── Categories.ts
│   │   │   ├── Orders.ts
│   │   │   ├── Users.ts
│   │   │   └── Media.ts
│   │   ├── access/           # Access control
│   │   └── payload.config.ts # Payload configuration
│   ├── components/           # React components
│   │   ├── ui/              # Shadcn components
│   │   ├── cart/            # Cart components
│   │   └── products/        # Product components
│   ├── lib/                 # Utilities
│   └── db/                  # Database schema (Drizzle)
├── public/                  # Static assets
└── .autoplans/             # AutoPlans project management
```

## Data Model

### Core Collections

#### Products
```typescript
{
  id: string;
  name: string;
  description: richText;
  price: number;
  images: Media[];
  categories: Category[];
  inventory: number;
  sku: string;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

#### Categories
```typescript
{
  id: string;
  name: string;
  slug: string;
  description: string;
  image: Media;
  parent?: Category;
}
```

#### Orders
```typescript
{
  id: string;
  orderNumber: string;
  customer: User;
  items: OrderItem[];
  total: number;
  status: 'pending' | 'processing' | 'shipped' | 'delivered';
  shippingAddress: Address;
  createdAt: Date;
}
```

#### Users
```typescript
{
  id: string;
  email: string;
  roles: ('admin' | 'customer')[];
  orders: Order[];
  addresses: Address[];
}
```

## Key Features

### 1. Content Management (Payload CMS)
- Admin panel at `/admin`
- Rich text editor for product descriptions
- Media library management
- User and role management
- Collection-based data structure

### 2. E-commerce Functionality
- Product catalog with filtering
- Category navigation
- Shopping cart (session-based)
- Order processing
- User accounts
- Order history

### 3. Database Management
- Neon PostgreSQL (serverless)
- Drizzle ORM for type-safe queries
- Automatic connection pooling
- Migration system

### 4. Authentication
- Payload built-in auth
- Role-based access control
- Admin vs. customer roles
- Protected routes

## API Endpoints

### Payload Generated
- `GET /api/products` - List products
- `GET /api/products/:id` - Get product
- `POST /api/products` - Create product (admin)
- `PATCH /api/products/:id` - Update product (admin)
- `DELETE /api/products/:id` - Delete product (admin)

Similar patterns for: categories, orders, users, media

### Custom Endpoints
- `POST /api/cart/add` - Add to cart
- `POST /api/cart/remove` - Remove from cart
- `POST /api/checkout` - Process order

## Development Workflow

### Adding New Features

1. **New Collection**:
   ```bash
   # Create collection file
   touch src/payload/collections/NewCollection.ts
   # Register in payload.config.ts
   # Run migration
   pnpm payload migrate:create
   pnpm payload migrate
   ```

2. **Frontend Changes**:
   - Update components in `src/components/`
   - Add routes in `src/app/`
   - Use Server Components for data fetching

3. **Database Changes**:
   - Update Drizzle schema
   - Create migration
   - Test migration

### Environment Configuration

```env
# Database
DATABASE_URL=postgresql://user:pass@host/db

# Payload
PAYLOAD_SECRET=your-secret-key
NEXT_PUBLIC_SERVER_URL=http://localhost:32100

# Media (optional)
UPLOADTHING_SECRET=
UPLOADTHING_APP_ID=

# Payment (optional)
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
```

## Performance Optimization

1. **Database**: Connection pooling via Neon
2. **Images**: Next.js Image optimization
3. **Caching**: Server Components cache by default
4. **Static Generation**: Build-time product pages
5. **API**: Payload built-in pagination

## Security Considerations

1. **Authentication**: Payload JWT-based auth
2. **Access Control**: Collection-level permissions
3. **Input Validation**: Payload field validation
4. **SQL Injection**: Drizzle ORM protection
5. **Environment Variables**: Never commit secrets

## Testing Strategy

- **Unit**: Component testing
- **Integration**: API route testing
- **E2E**: Checkout flow testing
- **Database**: Migration testing

---
*Architecture evolves with project needs. Update this document as the system grows.*
