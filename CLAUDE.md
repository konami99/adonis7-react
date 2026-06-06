# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**adonis7-react** is a full-stack web application combining:
- **Backend**: AdonisJS 7 (Node.js framework) with TypeScript
- **Frontend**: React 19 + Inertia.js (server-driven rendering)
- **Database**: SQLite (via better-sqlite3) with Lucid ORM
- **Build Tool**: Vite (frontend bundling)
- **Authentication**: Session-based (via @adonisjs/auth)
- **Validation**: VineJS for schema validation
- **Styling**: Custom CSS (in `inertia/css/app.css`)

The application uses Inertia.js to seamlessly bridge server-side routing with React components, eliminating traditional API boundaries while maintaining a full-stack architecture.

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Runtime | Node.js | >=24.0.0 |
| Backend Framework | AdonisJS | ^7.3.3 |
| Frontend Library | React | ^19.2.6 |
| SSR/Page Framework | Inertia.js | ^4.2.0 / ^2.3.24 (React) |
| Database Driver | better-sqlite3 | ^12.10.0 |
| ORM | Lucid (Adonis) | ^22.4.2 |
| Validation | VineJS | ^4.4.0 |
| Type Language | TypeScript | ~6.0.3 |
| Build System | Vite | ^7.3.3 |
| Routing Library | Tuyau | ^1.2.2 |
| Notifications | sonner | ^2.0.7 |

## Project Structure

```
adonis7-react/
├── app/                          # Backend application code
│   ├── controllers/              # HTTP request handlers
│   ├── models/                   # Lucid ORM models
│   ├── middleware/               # HTTP middleware
│   ├── validators/               # VineJS validation schemas
│   ├── transformers/             # Data serialization (for APIs)
│   ├── exceptions/               # Custom exception classes
│   ├── mails/                    # Email templates (if needed)
│   ├── services/                 # Business logic services
│   └── policies/                 # Authorization policies
├── inertia/                      # Frontend (React) code
│   ├── app.tsx                   # React app entry point
│   ├── client.ts                 # Tuyau HTTP client setup
│   ├── layouts/                  # Layout components (default.tsx)
│   ├── pages/                    # Page components (routed via Inertia)
│   │   ├── home.tsx              # Homepage
│   │   ├── auth/                 # Authentication pages
│   │   │   ├── login.tsx
│   │   │   └── signup.tsx
│   │   └── errors/               # Error pages (404, 500)
│   ├── css/                      # Stylesheets
│   └── tsconfig.json             # Frontend TypeScript config
├── config/                       # Configuration files
│   ├── app.ts                    # HTTP & app settings
│   ├── auth.ts                   # Authentication guards & providers
│   ├── database.ts               # Database connections & migrations
│   ├── session.ts                # Session configuration
│   ├── cors.ts                   # CORS policy
│   ├── shield.ts                 # CSRF & security headers
│   └── ...other configs
├── database/                     # Database layer
│   ├── migrations/               # Schema migration files
│   └── schema.ts                 # Generated schema definitions (auto-updated)
├── start/                        # Application startup hooks
│   ├── routes.ts                 # HTTP route definitions
│   ├── kernel.ts                 # Middleware registration
│   ├── validator.ts              # Validator setup
│   └── env.ts                    # Environment variable validation
├── providers/                    # Custom service providers
│   └── api_provider.ts           # Custom API serializer
├── tests/                        # Test suites
│   ├── unit/                     # Unit tests
│   ├── functional/               # Functional tests
│   └── browser/                  # Browser-based tests
├── public/                       # Static assets
├── adonisrc.ts                   # AdonisJS app config & hooks
├── vite.config.ts                # Vite frontend build config
├── tsconfig.json                 # Backend TypeScript config
├── package.json                  # Dependencies & scripts
└── .env.example                  # Environment template

Key generated directories (auto-created):
├── .adonisjs/                    # Generated code (types, routes, controllers)
└── build/                        # Production build output
```

## Key Architecture Patterns

### 1. **Inertia.js Server-Driven Rendering**
Pages are rendered by backend controllers and passed to React components via props. Controllers use `inertia.render()` to send data to frontend pages:

```typescript
// Backend (controller)
export default class HomeController {
  async show({ inertia }: HttpContext) {
    return inertia.render('home', { /* props */ })
  }
}

// Frontend (React page)
export default function Home(props) { /* ... */ }
```

Pages are automatically matched to files in `inertia/pages/` (e.g., `pages/home.tsx` → `home` route).

### 2. **Type-Safe Routing with Tuyau**
Routes are type-checked through Tuyau, providing type-safe route helpers in React:

```typescript
// Frontend
<Link route="session.create">Login</Link>
```

Route names are defined in `start/routes.ts` using `.as('session.create')` suffixes.

### 3. **Middleware Stack**
- **Server middleware** (all requests): Static files, CORS, Vite dev server, Inertia
- **Router middleware** (only routed requests): Body parser, session, CSRF, auth initialization
- **Named middleware** (selectively applied): `guest`, `auth`

Defined in `start/kernel.ts`.

### 4. **Validation with VineJS**
VineJS validators are defined in `app/validators/` and used in controllers:

```typescript
export const signupValidator = vine.create({
  email: vine.string().email().unique({ table: 'users', column: 'email' }),
  password: vine.string().minLength(8).confirmed(),
})

// In controller
const payload = await request.validateUsing(signupValidator)
```

### 5. **Authentication & Session Management**
- **Guard**: `web` (session-based, uses `sessionGuard` from @adonisjs/auth)
- **Provider**: `sessionUserProvider` with User model
- **User Model**: Lucid ORM with auth mixin (`withAuthFinder`)
- **Session Driver**: Cookie-based (configured in `.env` as `SESSION_DRIVER=cookie`)

Login/logout flows in `SessionController` and `NewAccountController`.

### 6. **Data Serialization**
Custom `ApiSerializer` in `providers/api_provider.ts` wraps API responses under a `data` key:
```json
{ "data": [...] }
```

Applied to transformers for consistent API response structure.

### 7. **Frontend Layout & Flash Messages**
`inertia/layouts/default.tsx` is the shared layout for all pages. It:
- Displays header with user info (via `children.props.user`)
- Shows flash messages (success/error) via sonner toasts
- Handles logout form

Flash messages are passed from backend via Inertia shared props.

## Common Commands

### Development
```bash
npm run dev          # Start dev server (HMR enabled on port 3333)
npm run typecheck    # Type-check backend & frontend TypeScript
npm run lint         # Run ESLint across all code
npm run format       # Auto-format code with Prettier
```

### Building & Running
```bash
npm run build        # Build for production (backend + frontend)
npm start            # Run production build
```

### Database
```bash
node ace migration:run        # Run pending migrations
node ace migration:rollback   # Rollback migrations
node ace migration:refresh    # Reset database & re-run migrations
```

### Testing
```bash
npm test                           # Run all test suites (unit, functional, browser)
npm test -- --suites=unit         # Run only unit tests
npm test -- --suites=functional   # Run only functional tests
npm test -- --files=path/file.ts  # Run a specific test file
```

### Other Useful Commands
```bash
node ace generate:migration name_here    # Create a new migration
node ace list:routes                     # List all registered routes
node ace tinker                          # REPL for interactive development
```

## Database & ORM

**ORM**: Lucid (AdonisJS's query builder + ORM)

- Models extend `BaseModel` from `@adonisjs/lucid/orm`
- Define columns with decorators: `@column()`, `@column.dateTime()`, etc.
- Models automatically sync with database schema after migrations
- Schema is auto-generated in `database/schema.ts` after migrations (do not edit manually)

**Current Schema**:
- `users` table: `id` (PK), `full_name` (nullable), `email` (unique), `password`, `created_at`, `updated_at`

Migration files in `database/migrations/` use Lucid's schema builder.

## Frontend Patterns

### Shared Props
All pages receive shared props defined in Inertia middleware (`inertia_middleware.ts`):
- `user`: Current authenticated user (null if guest)
- `flash`: Object with `error` and `success` messages
- `csrf`: CSRF token for forms

Access via: `const { user, flash } = usePage().props`

### Forms
Use the `<Form>` component from `@adonisjs/inertia/react` for automatic CSRF protection and method spoofing:

```typescript
<Form route="session.destroy" method="post">
  <button type="submit">Logout</button>
</Form>
```

### Routing
Use `<Link>` component (Inertia) for client-side navigation:
```typescript
<Link route="home">Home</Link>
<Link href="/external">External Link</Link>
```

## Environment Variables

Key variables (see `.env.example`):
- `PORT`: Server port (default: 3333)
- `APP_KEY`: Encryption key (required for session middleware)
- `APP_URL`: Absolute URL of the app (used in email links, etc.)
- `SESSION_DRIVER`: `cookie` (default) or `file`
- `NODE_ENV`: `development`, `production`, or `test`

All env vars are validated in `start/env.ts` using VineJS.

## Code Generation & Hooks

AdonisJS auto-generates code during initialization:
- **Controllers index** (`@generated/controllers`): Type-safe controller imports
- **Routes metadata** (Tuyau): Type-safe route helpers
- **Page index** (Inertia): Automatic page → component resolution
- **Types** (`@generated/data`): Page props types for frontend

These are generated by hooks in `adonisrc.ts` and should never be edited manually.

## Testing Strategy

- **Unit tests** (`tests/unit/`): Isolated logic, mocked dependencies
- **Functional tests** (`tests/functional/`): Full request/response cycles, database operations
- **Browser tests** (`tests/browser/`): E2E browser automation (via @japa/browser-client)

Use `@japa/plugin-adonisjs` for test helpers (database transactions, assertions).

## File Naming Conventions

- Controllers: `{resource}_controller.ts` (e.g., `session_controller.ts`)
- Models: PascalCase (e.g., `User.ts`)
- Validators: `{model}.ts` (e.g., `user.ts`)
- Migrations: Timestamp prefix `{timestamp}_{description}.ts`
- React Pages: snake_case (e.g., `home.tsx`, `auth/login.tsx`, `errors/not_found.tsx`)
- Middleware: `{name}_middleware.ts`

## Key Imports & Aliases

Configured in `package.json` (backend) and `tsconfig.inertia.json` (frontend):

**Backend**:
- `#controllers/*` → `./app/controllers/`
- `#models/*` → `./app/models/`
- `#validators/*` → `./app/validators/`
- `#services/*` → `./app/services/`
- `#middleware/*` → `./app/middleware/`
- `#generated/*` → `./.adonisjs/server/` (generated code)
- `#start/*` → `./start/`
- `#config/*` → `./config/`
- `#database/*` → `./database/`

**Frontend**:
- `~/*` → `./inertia/`
- `@generated/*` → `./.adonisjs/client/`

## Important Notes for Future Work

1. **Do not manually edit generated files**: Schema definitions, controller/route indexes, and type files in `.adonisjs/` are auto-generated. Modify migrations and source files instead.

2. **Database migrations are immutable**: Once pushed/deployed, create new migrations for schema changes rather than modifying existing ones.

3. **Inertia page props must be serializable**: Props passed from controllers to React pages must be JSON-serializable (no functions, class instances, or circular references).

4. **Session-based auth only**: Current setup uses session authentication. For API endpoints requiring token auth, extend `config/auth.ts` with additional guards.

5. **TypeScript strict mode**: Backend uses AdonisJS's strict TypeScript config. Avoid `any` types; use proper type inference.

6. **Frontend build artifacts**: Vite output goes to `public/assets/`. These are generated on build and should not be committed to git.

7. **Middleware ordering matters**: Server middleware runs before router middleware. For auth-dependent logic, use router middleware or named middleware on specific routes.

8. **Hot-hook boundaries**: The `hotHook` setting in `package.json` specifies which files trigger full server reload vs. HMR. Controllers and middleware require full restart.
