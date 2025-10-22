# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Strapi 5.28.0 headless CMS application written in TypeScript. It's configured as a blog template with content types for articles, authors, categories, and global settings. The application uses better-sqlite3 by default but supports MySQL and PostgreSQL.

## Development Commands

```bash
# Start development server with auto-reload
yarn develop

# Start production server (no auto-reload)
yarn start

# Build admin panel
yarn build

# Seed example data (runs once per database)
yarn seed:example

# Access Strapi console
yarn console

# Upgrade to latest Strapi version
yarn upgrade          # Apply upgrade
yarn upgrade:dry      # Preview upgrade changes
```

## Architecture

### Content Type Structure

Content types are organized in `src/api/` with a consistent pattern:
- `content-types/[name]/schema.json` - Database schema and field definitions
- `controllers/[name].ts` - Request handlers
- `routes/[name].ts` - Route definitions
- `services/[name].ts` - Business logic

Key content types:
- **article** (collectionType) - Blog posts with dynamic zone blocks, relations to author/category
- **author** (collectionType) - Author profiles with avatar media
- **category** (collectionType) - Article categorization
- **global** (singleType) - Site-wide settings like siteName, favicon, SEO defaults
- **about** (singleType) - About page with dynamic zone blocks

### Dynamic Zone Components

Reusable content blocks in `src/components/shared/`:
- **media** - Single file upload (images/videos/files)
- **quote** - Text quotes
- **rich-text** - Formatted text content
- **slider** - Multiple file carousel
- **seo** - Meta tags and social sharing configuration

These components are used in dynamic zones on Article and About content types.

### Database Configuration

Database client is set via `DATABASE_CLIENT` env var (defaults to sqlite). The `config/database.ts` file provides connection configurations for:
- **sqlite** - Development default (`.tmp/data.db`)
- **postgres** - Production option with connection pooling and SSL support
- **mysql** - Alternative production option with connection pooling and SSL support

### Application Lifecycle

`src/index.ts` exports register and bootstrap functions:
- **register()** - Runs before initialization, for extending code
- **bootstrap()** - Runs before startup, for data setup or async initialization

### Seed Data System

The `scripts/seed.js` implements a sophisticated seeding system that:
- Tracks whether seed has run via plugin store (`initHasRun` flag)
- Sets public permissions for content type endpoints
- Uploads files from `data/uploads/` directory
- Creates content entries from `data/data.json`
- Handles dynamic zone blocks with file references
- Prevents duplicate file uploads by checking existing files
- Only runs once per database lifecycle

### Admin Panel Customization

Admin UI can be customized via:
- `src/admin/app.example.tsx` - React app configuration (rename to `app.tsx` to activate)
- `src/admin/vite.config.example.ts` - Vite build configuration (rename to `vite.config.ts` to activate)
- Separate TypeScript config at `src/admin/tsconfig.json`

### TypeScript Configuration

The project uses two TypeScript configurations:
- Root `tsconfig.json` - Server-side code (excludes `src/admin/`)
- `src/admin/tsconfig.json` - Admin panel code

Server config targets ES2019 with CommonJS modules and excludes test files, build artifacts, and plugins.

## Environment Setup

Copy `.env.example` to `.env` and configure:
- `HOST` / `PORT` - Server binding (default: 0.0.0.0:1337)
- `APP_KEYS` - Session encryption keys (comma-separated)
- `API_TOKEN_SALT` - API token hashing
- `ADMIN_JWT_SECRET` - Admin authentication
- `TRANSFER_TOKEN_SALT` - Transfer token hashing
- `JWT_SECRET` - JWT signing
- `ENCRYPTION_KEY` - Field-level encryption
- `DATABASE_CLIENT` - Database type (sqlite/postgres/mysql)
- `DATABASE_*` - Database connection parameters (host, port, credentials, SSL)

All secret values in `.env.example` use placeholder values and must be changed for production.

## Common Workflows

### Creating a New Content Type

1. Use Strapi CLI: `yarn strapi generate`
2. Select "content-type" and follow prompts
3. Schema files generate in `src/api/[name]/content-types/[name]/schema.json`
4. Controllers, services, and routes auto-generate with CRUD operations

### Modifying Content Type Schema

Edit the `schema.json` file directly. Changes apply on server restart. Key schema properties:
- `kind`: "collectionType" (multiple entries) or "singleType" (single entry)
- `options.draftAndPublish`: Enable draft/publish workflow
- `attributes`: Field definitions with types (string, text, media, relation, component, dynamiczone)

### Working with Relations

Relations are defined in schema attributes:
```json
{
  "author": {
    "type": "relation",
    "relation": "manyToOne",
    "target": "api::author.author",
    "inversedBy": "articles"
  }
}
```

### Adding Shared Components

Create JSON schema in `src/components/shared/[name].json`. Components can be used in:
- Single-use component fields
- Repeatable component fields
- Dynamic zones (multiple component types)

### Database Operations

Using Strapi Document Service API (v5 pattern):
```typescript
// Create
await strapi.documents('api::article.article').create({ data: {...} });

// Query with filters
await strapi.query('api::article.article').findMany({ where: {...} });
```

### Resetting Database

SQLite development databases can be reset by:
1. Stop server
2. Delete `.tmp/data.db`
3. Restart server
4. Run `yarn seed:example` to repopulate

## File Organization

- `config/` - Server, database, middleware, plugin configurations
- `src/api/` - Content type definitions and API logic
- `src/components/` - Reusable content components
- `src/admin/` - Admin panel customizations (excluded from server build)
- `data/` - Seed data JSON and upload files
- `scripts/` - Utility scripts (seeding, migrations)
- `public/` - Static files served by Strapi
- `types/generated/` - Auto-generated TypeScript definitions