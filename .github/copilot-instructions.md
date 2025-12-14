<!-- Copilot / AI agent instructions for contributors working on this repo -->
# Event Management — Copilot Instructions

Purpose: Give AI coding agents the minimal, concrete knowledge needed to be productive in this repo.

- Project layout (core):
  - Backend: `backend/` — Express + Mongoose. Entrypoint: `backend/server.js` and `backend/package.json` (scripts: `start`, `dev`).
  - Frontend: `frontend/` — static HTML/CSS/JS pages (open in browser or serve with a static server).
  - Key folders: `backend/routes/`, `backend/controllers/`, `backend/models/`, `backend/middleware/`, `backend/socket.js`.

- Run / dev workflow (explicit):
  - Start backend (recommended dev):
    - `cd backend && npm install` (one-time)
    - `cd backend && npm run dev` (uses `nodemon`, restarts on file change)
  - Start backend in production-like mode: `cd backend && npm start`.
  - Frontend is static: open `frontend/index.html` in a browser or serve via `npx serve frontend` / `npx http-server frontend -p 5500` for proper CORS/localhost testing.
  - Note: `server.js` loads `.env` using `path.join(__dirname, '.env')` — ensure any env file is in `backend/` when starting from workspace root.

- Important env vars (backend/.env or backend/.env.example):
  - `MONGO_URI` — MongoDB connection string (required).
  - `JWT_SECRET` — used for signing/verifying JWT tokens.
  - `ADMIN_SECRET` — optional fallback admin secret (default visible in code: `event$phere2025` if not set).
  - `PORT` — server port (default 5000).

- Authentication patterns and gotchas (explicit):
  - Middleware: `backend/middleware/auth.js` accepts either:
    - Bearer JWT in `Authorization: Bearer <token>` or `x-auth-token` header (decoded id becomes `req.user._id`), or
    - Admin secret via `x-admin-secret` header (or `?adminSecret=`) — compare to `process.env.ADMIN_SECRET`.
  - Useful for AI agents: tests and API requests can bypass JWT by sending `x-admin-secret: <secret>`.

- Socket usage and events (explicit):
  - Initialization: `backend/socket.js` exports `init(server)` and `getIo()`; called from `server.js` after mongoose connects.
  - Emitted event: `eventsUpdated` — payload examples seen in `backend/routes/event.js`:
    - On create: `{ action: 'created', event }`
    - On update: `{ action: 'updated', event }`
    - On delete: `{ action: 'deleted', id }`
  - Agents should wrap `getIo()` calls in try/catch; socket may be uninitialized during some startup paths.

- Data flow & domain model highlights:
  - `backend/models/Event.js` — canonical event schema. Note `category` is a friendly slug used by frontend templates (e.g., `wedding`, `beach-wedding`).
  - `Registration`, `User`, and `payment` models exist under `backend/models/` — follow the same Mongoose patterns when adding fields.
  - Routes: `backend/routes/event.js` performs create/read/update/delete and already contains real examples of saving, emitting socket events, and error handling.

- Project-specific conventions and patterns:
  - Console-first debugging: many files include informative `console.log` / `[TAG]` messages — prefer following existing logging style when adding messages.
  - CORS policy in `server.js` intentionally permissive (`origin: '*'`) to support static local frontend testing — do not tighten without updating frontend dev workflow.
  - Controllers directory may contain placeholders (some files empty). Routes sometimes implement logic inline (see `routes/event.js`). Check both `controllers/` and `routes/` before refactoring.

- Integration points / external deps to be aware of:
  - MongoDB (via `mongoose`): ensure `MONGO_URI` points to a running MongoDB instance.
  - JWT (`jsonwebtoken`) for user auth flows.
  - `socket.io` — real-time updates emitted from backend routes.
  - Payments: there is a `payment` model and `backend/routes/paymentRoutes.js` — inspect before touching payment flows.

- Quick examples agents can use when suggesting code/tests:
  - Example backend start: `cd backend && npm run dev`.
  - Example API call using admin secret (curl):
    - `curl -X POST http://localhost:5000/api/events -H "x-admin-secret: event$phere2025" -H "Content-Type: application/json" -d '{"title":"Demo","date":"2026-01-01"}'`
  - Example socket listen (frontend): the frontend expects `eventsUpdated` and will refresh listings when received.

- Where to look first when debugging or extending features:
  - Authentication: `backend/middleware/auth.js`
  - Event CRUD: `backend/routes/event.js` and `backend/models/Event.js`
  - Server startup & env loading: `backend/server.js` (socket init occurs here)
  - Frontend templates: `frontend/` — `category` values map to filenames like `wedding.html`.

If anything here is unclear or you want me to include automated test run commands or CI details, tell me which area to expand and I will iterate.
