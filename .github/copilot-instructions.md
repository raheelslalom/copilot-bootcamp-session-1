# Copilot Instructions for `copilot-bootcamp-session-1`

## Big Picture
- This is an npm workspaces monorepo with two packages: `packages/backend` (Express API) and `packages/frontend` (Create React App UI).
- End-to-end flow: React loads/submits items via `/api/items` in `packages/frontend/src/App.js`, backend handles routes in `packages/backend/src/app.js`.
- Frontend-to-backend integration is local-dev proxy based (`"proxy": "http://localhost:3030"` in `packages/frontend/package.json`), so client code should call relative API paths (for example `fetch('/api/items')`).

## Backend Architecture (`packages/backend`)
- Entrypoint `src/index.js` starts server on `PORT` default `3030` and imports `app` from `src/app.js`.
- `src/app.js` owns Express setup, middleware (`cors`, `express.json`, `morgan`), SQLite setup, schema creation, seed data, and route handlers.
- Data layer is intentionally in-memory SQLite (`new Database(':memory:')`), so all data resets on process restart.
- Existing API contract:
  - `GET /api/items` → array of items sorted by `created_at DESC`
  - `POST /api/items` with `{ name }` → `201` + created item
  - Invalid/missing/empty `name` → `400` with `{ error: 'Item name is required' }`
- `app`, `db`, and `insertStmt` are exported for testability (`module.exports = { app, db, insertStmt }`).

## Frontend Architecture (`packages/frontend`)
- Main UI is a single component in `src/App.js` using React hooks (`useState`, `useEffect`) and native `fetch`.
- Current state model: `data`, `loading`, `error`, `newItem`.
- Initial load calls `fetchData()` once on mount; submit handler posts new item then appends response to local state.
- Keep UX strings and behavior aligned with tests (examples: `Loading data...`, `No items found. Add some!`, input placeholder `Enter item name`).

## Workflows & Commands
- Install all dependencies from repo root: `npm install`.
- Run both apps concurrently from root: `npm start`.
- Run one package only:
  - `npm run start:backend`
  - `npm run start:frontend`
- Run tests from root: `npm test` (frontend then backend).
- Package-specific tests:
  - `npm run test --workspace=frontend`
  - `npm run test --workspace=backend`

## Testing Patterns to Follow
- Backend tests (`packages/backend/__tests__/app.test.js`) use `supertest` against the exported `app` instance.
- Backend tests close the shared DB in `afterAll`; preserve this lifecycle if changing DB setup.
- Frontend tests (`packages/frontend/src/__tests__/App.test.js`) use React Testing Library + `msw` handlers for `/api/items` GET/POST.
- Prefer updating/adding MSW handlers instead of mocking `fetch` directly in component tests.

## Codebase-Specific Conventions
- Backend uses CommonJS (`require`/`module.exports`), not ESM.
- Keep API error payload shape stable (`{ error: string }`) unless tests and UI are updated together.
- Keep changes small and package-local when possible; root scripts orchestrate package scripts via workspaces.