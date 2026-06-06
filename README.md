# adonis7-react

A full-stack web application built with [AdonisJS 7](https://adonisjs.com/) and [React 19](https://react.dev/), connected via [Inertia.js](https://inertiajs.com/) for server-driven rendering without a separate API layer.

## Tech Stack

- **Backend**: AdonisJS 7, TypeScript, Lucid ORM, VineJS, session-based auth
- **Frontend**: React 19, Inertia.js, Tuyau (type-safe routing), Vite
- **Database**: SQLite (via better-sqlite3)

## Requirements

- Node.js >= 24.0.0

## Getting Started

```bash
# Install dependencies
npm install

# Copy environment file and set APP_KEY
cp .env.example .env
node ace generate:key   # paste output into APP_KEY in .env

# Run database migrations
node ace migration:run

# Start development server (with HMR)
npm run dev
```

The app will be available at `http://localhost:3333`.

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server with hot module replacement |
| `npm run build` | Build for production |
| `npm start` | Run production build |
| `npm test` | Run all tests |
| `npm run lint` | Lint with ESLint |
| `npm run format` | Format with Prettier |
| `npm run typecheck` | Type-check backend and frontend |

## Database

```bash
node ace migration:run        # Run pending migrations
node ace migration:rollback   # Rollback last migration
node ace generate:migration <name>  # Create a new migration
```

## Testing

```bash
npm test                               # All suites
npm test -- --suites=unit             # Unit tests only
npm test -- --suites=functional       # Functional tests only
npm test -- --files=tests/path/file.ts  # Single file
```
