# Portal Mini Store - AI Development Instructions

## Project Context

This is a **full-featured e-commerce platform** using:
- Next.js App Router
- Payload CMS (headless CMS)
- Neon PostgreSQL (serverless database)
- Drizzle ORM
- Tailwind CSS + Shadcn UI

## Critical Rules for AI Agents

### 1. Database Management (CRITICAL)

**ALWAYS use migrations for schema changes:**

```bash
# Create a new migration
pnpm payload migrate:create

# Run migrations
pnpm payload migrate

# NEVER modify database schema directly
# NEVER skip migrations
```

**Database Connection:**
- Uses Neon PostgreSQL (serverless)
- Connection string in `DATABASE_URL` environment variable
- Automatic connection pooling enabled

### 2. Payload CMS Patterns

**Collections are defined in `src/payload/collections/`:**

```typescript
// src/payload/collections/Products.ts
import type { CollectionConfig } from 'payload/types'

export const Products: CollectionConfig = {
  slug: 'products',
  admin: {
    useAsTitle: 'name',
  },
  access: {
    read: () => true,
    create: ({ req: { user } }) => !!user?.roles?.includes('admin'),
    update: ({ req: { user } }) => !!user?.roles?.includes('admin'),
    delete: ({ req: { user } }) => !!user?.roles?.includes('admin'),
  },
  fields: [
    {
      name: 'name',
      type: 'text',
      required: true,
    },
    {
      name: 'price',
      type: 'number',
      required: true,
    },
    // ... more fields
  ],
}
```

**Register collections in `payload.config.ts`:**
```typescript
export default buildConfig({
  collections: [Products, Categories, Orders, Users, Media],
  // ...
})
```

### 3. Access Control

**Always implement proper access control:**

```typescript
access: {
  // Public read
  read: () => true,
  
  // Admin-only write
  create: ({ req: { user } }) => {
    return !!user?.roles?.includes('admin')
  },
  update: ({ req: { user } }) => {
    return !!user?.roles?.includes('admin')
  },
  delete: ({ req: { user } }) => {
    return !!user?.roles?.includes('admin')
  },
  
  // User can only access their own data
  read: ({ req: { user } }) => {
    if (user?.roles?.includes('admin')) return true
    return { user: { equals: user?.id } }
  }
}
```

### 4. Data Fetching Patterns

**Server Component (fetch Payload data):**
```typescript
import { getPayloadHMR } from '@payloadcms/next/utilities'
import config from '@payload-config'

export default async function ProductsPage() {
  const payload = await getPayloadHMR({ config })
  
  const { docs: products } = await payload.find({
    collection: 'products',
    where: {
      isActive: {
        equals: true
      }
    },
    limit: 10,
    sort: '-createdAt'
  })
  
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}
```

**Client Component (use API):**
```typescript
'use client'
import { useEffect, useState } from 'react'

export default function ProductsList() {
  const [products, setProducts] = useState([])
  
  useEffect(() => {
    fetch('/api/products?limit=10')
      .then(res => res.json())
      .then(data => setProducts(data.docs))
  }, [])
  
  return <div>{/* render products */}</div>
}
```

### 5. Route Organization

```
app/
├── (storefront)/          # Public store pages
│   ├── page.tsx          # Homepage
│   ├── products/
│   │   ├── page.tsx      # Product listing
│   │   └── [slug]/
│   │       └── page.tsx  # Product detail
│   ├── cart/
│   │   └── page.tsx      # Shopping cart
│   └── checkout/
│       └── page.tsx      # Checkout
├── (admin)/              # Admin-only pages (protected)
│   └── dashboard/
│       └── page.tsx
└── api/                  # Custom API routes
    ├── cart/
    └── checkout/
```

### 6. Working with Media

**Upload configuration in Payload:**
```typescript
{
  name: 'image',
  type: 'upload',
  relationTo: 'media',
  required: true,
}
```

**Display in Next.js:**
```typescript
import Image from 'next/image'

// product.image is a Media object
<Image 
  src={product.image.url} 
  alt={product.image.alt || product.name}
  width={product.image.width}
  height={product.image.height}
/>
```

### 7. Environment Variables

**Required for development:**
```env
# Database
DATABASE_URL=postgresql://username:password@host/database

# Payload CMS
PAYLOAD_SECRET=your-32-character-secret-key
NEXT_PUBLIC_SERVER_URL=http://localhost:32100

# Media Storage (optional)
UPLOADTHING_SECRET=
UPLOADTHING_APP_ID=

# Payment (optional)
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
```

### 8. TypeScript Types

**Payload auto-generates types:**
```typescript
import type { Product, Category, Order } from '@/payload-types'

// Use generated types
const product: Product = {
  id: '123',
  name: 'Sample Product',
  price: 29.99,
  // ... type-safe properties
}
```

### 9. Common Operations

**Create a product (admin only):**
```typescript
const payload = await getPayloadHMR({ config })

const newProduct = await payload.create({
  collection: 'products',
  data: {
    name: 'New Product',
    price: 49.99,
    description: 'Product description',
    isActive: true,
  },
  user: req.user, // for access control
})
```

**Query products with filters:**
```typescript
const { docs: products } = await payload.find({
  collection: 'products',
  where: {
    and: [
      { isActive: { equals: true } },
      { price: { less_than: 100 } },
      { category: { in: ['electronics', 'gadgets'] } }
    ]
  },
  limit: 20,
  sort: '-createdAt',
  page: 1,
})
```

**Update a product:**
```typescript
await payload.update({
  collection: 'products',
  id: productId,
  data: {
    inventory: currentInventory - quantity,
  },
  user: req.user,
})
```

### 10. Shopping Cart Implementation

**Session-based cart (recommended for guests):**
```typescript
// app/api/cart/add/route.ts
import { cookies } from 'next/headers'

export async function POST(request: Request) {
  const { productId, quantity } = await request.json()
  
  const cartCookie = cookies().get('cart')
  const cart = cartCookie ? JSON.parse(cartCookie.value) : { items: [] }
  
  // Add or update item
  const existingItem = cart.items.find(i => i.productId === productId)
  if (existingItem) {
    existingItem.quantity += quantity
  } else {
    cart.items.push({ productId, quantity })
  }
  
  cookies().set('cart', JSON.stringify(cart))
  
  return Response.json({ success: true, cart })
}
```

### 11. Common Mistakes to Avoid

❌ **Don't modify database directly** - Always use migrations
❌ **Don't skip access control** - Every collection needs proper permissions
❌ **Don't expose admin endpoints** - Use route groups for protection
❌ **Don't forget to validate input** - Use Payload field validation
❌ **Don't commit environment variables** - Use `.env.local`
❌ **Don't query database in client components** - Use Server Components or API routes

## AutoPlans Integration

When working on tasks:
1. Check `.autoplans/tasks.md` for current work
2. Use `autoplans_update_task` to mark progress
3. Create migrations for database changes
4. Update architecture.md when adding major features
5. Document new collections and their relationships

## Development Commands

```bash
# Install dependencies
pnpm install
npm install --legacy-peer-deps

# Run development server
pnpm run dev --port 32100
npm run dev -- --port 32100

# Database migrations
pnpm payload migrate:create  # Create new migration
pnpm payload migrate         # Run migrations

# Build for production
pnpm build
npm run build

# Access Payload admin
# Navigate to http://localhost:32100/admin
```

## Neon-Specific Considerations

1. **Connection Pooling**: Automatic, no configuration needed
2. **Serverless**: Database scales automatically
3. **Branching**: Use Neon branches for development
4. **Backups**: Automatic continuous backups

## Getting Help

- Payload Docs: https://payloadcms.com/docs
- Neon Docs: https://neon.tech/docs
- Next.js Docs: https://nextjs.org/docs
- Drizzle ORM: https://orm.drizzle.team/docs

---
*Follow these instructions for optimal AI-assisted Portal Mini Store development.*
