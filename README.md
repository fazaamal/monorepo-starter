# Monorepo Starter

A [Bun workspaces](https://bun.com/docs/install/workspaces) monorepo template for full-stack TypeScript projects. The frontend ships ready to run; the backend is yours to add — pick the framework that fits your team.

## Prerequisites

- [Bun](https://bun.com) 1.x

## Quick start

```bash
bun install
bun --cwd apps/website run dev
```

The website dev server runs at `http://localhost:3000`.

## Repository structure

```
monorepo-starter/
├── apps/
│   ├── website/          # Nuxt 4 frontend (included)
│   └── api/              # Backend service (you add this)
├── packages/             # Shared libraries (add as needed)
│   └── <name>/           # e.g. core, db, validators
├── package.json          # Workspace root
└── tsconfig.json         # Root TypeScript config
```

| Directory | Purpose |
|-----------|---------|
| `apps/` | Runnable services — frontend, API, workers. Each app is `"private": true` and has its own `package.json`. |
| `packages/` | Shared code consumed by apps — types, validation, business logic, database clients. Published internally via `workspace:*`. |

### What belongs where

- **Frontend UI, routing, and client state** → `apps/website`
- **HTTP APIs, auth, webhooks, background jobs** → `apps/api` (or additional apps under `apps/`)
- **Code used by more than one app** → `packages/<name>`

Keep domain logic in `packages/` so both the website and API can import it without duplicating code.

## Frontend (`apps/website`)

Nuxt 4 + Vue 3 + Tailwind CSS.

```bash
bun --cwd apps/website run dev       # development
bun --cwd apps/website run build     # production build
bun --cwd apps/website run preview   # preview production build
```

See [apps/website/README.md](apps/website/README.md) for Nuxt-specific docs.

## Backend (`apps/api`)

There is no backend app in the starter — add one under `apps/api` and wire it into the workspace. All options below live in the same place; only the internal layout changes with the framework.

### Shared setup (all backends)

1. Create the app:

   ```bash
   mkdir -p apps/api
   ```

2. Add `apps/api/package.json` with `"private": true` and workspace dependencies:

   ```json
   {
     "name": "api",
     "private": true,
     "type": "module",
     "scripts": {
       "dev": "…",
       "start": "…"
     },
     "dependencies": {
       "@org/core": "workspace:*"
     }
   }
   ```

3. Add shared packages under `packages/` and reference them with `"workspace:*"`.
4. Add `apps/api/.env.example` for required env vars (never commit `.env`).

---

### Option A — Bun.serve (Bun-native)

Best when you want a minimal HTTP server with no extra framework. Bun handles routing, WebSockets, and static files out of the box.

```
apps/api/
├── package.json
├── src/
│   ├── index.ts          # Bun.serve() entrypoint
│   ├── routes/           # route handlers
│   └── middleware/       # auth, logging, etc.
└── .env.example
```

```bash
# apps/api/package.json scripts
"dev": "bun --watch src/index.ts"
"start": "bun src/index.ts"
```

Put route handlers in `src/routes/`, shared validation in `packages/`, and use `bun:sqlite` / `Bun.sql` for data access.

---

### Option B — Express

Familiar middleware ecosystem; runs on Bun without Node.

```
apps/api/
├── package.json
├── src/
│   ├── index.ts          # create app, listen
│   ├── routes/           # express.Router() modules
│   ├── middleware/
│   └── controllers/      # request/response logic
└── .env.example
```

```bash
bun add express
bun add -d @types/express
```

```bash
# apps/api/package.json scripts
"dev": "bun --watch src/index.ts"
"start": "bun src/index.ts"
```

Keep Express-specific wiring (routers, middleware) in `apps/api`. Move reusable business rules and types into `packages/`.

---

### Option C — AdonisJS

Full MVC framework with built-in ORM, auth, validation, and CLI scaffolding.

```
apps/api/
├── package.json
├── adonisrc.ts
├── bin/
│   └── server.ts         # HTTP server entry
├── start/
│   ├── env.ts
│   ├── kernel.ts         # middleware
│   └── routes.ts
├── app/
│   ├── controllers/
│   ├── models/
│   ├── middleware/
│   └── validators/
├── config/               # database, auth, cors, etc.
├── database/
│   └── migrations/
└── .env.example
```

Scaffold with the Adonis CLI inside `apps/api`:

```bash
cd apps/api
bunx create-adonisjs@latest . --kit=api
```

Use `packages/` for code that Adonis apps and the Nuxt frontend both need (shared DTOs, constants, utilities). Keep Adonis-specific pieces — models, controllers, migrations — inside `apps/api`.

---

### Option D — Hono

Lightweight, works well on Bun and edge runtimes. Good middle ground between raw `Bun.serve` and Express.

```
apps/api/
├── package.json
├── src/
│   ├── index.ts          # Hono app + export
│   ├── routes/
│   └── middleware/
└── .env.example
```

```bash
bun add hono
```

---

## Shared packages (`packages/`)

Create a package when logic is reused across apps:

```
packages/core/
├── package.json          # "name": "@org/core"
├── tsconfig.json
└── src/
    └── index.ts
```

```bash
mkdir -p packages/core/src
```

Build shared packages before running apps that depend on them, or add a `build:deps` script in the app that builds referenced packages first.

## Workspace commands

Run scripts from the repo root:

```bash
bun install                              # install all workspaces
bun --cwd apps/website run dev           # run a specific app
bun --filter website dev                 # alternative: filter by package name
```

## Environment variables

- Each app manages its own `.env` file.
- Commit `.env.example` with dummy/placeholder values.
- Bun loads `.env` automatically — no `dotenv` needed.

## Adding more apps or packages

**New package** — copy the `packages/core` layout; add `"@org/<name>": "workspace:*"` to consuming apps.

**New app** — copy an existing app layout; set `"private": true` and list workspace dependencies.

**Build order** — apps should build their `packages/` dependencies first. For many packages, add a root script:

```json
"scripts": {
  "build": "bun --filter './packages/*' run build && bun --filter './apps/*' run build"
}
```
