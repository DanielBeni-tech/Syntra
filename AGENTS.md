# Base44 Dev Environment — CommHQ

## Overview
CommHQ is a Slack/Discord-style chat app for technical teams. Fullstack:
- **Backend**: NestJS + MongoDB (Mongoose) + Socket.IO, REST under `/api`, WebSocket namespace `/realtime`.
- **Frontend**: React 19 + Vite 8 + TailwindCSS 4 + shadcn/ui, TanStack Query, Zustand.

## Running the app
```bash
docker compose -f docker-compose.base44.yml up -d
```
- Frontend (Vite dev server): host port **3000** → container 5173
- Backend (NestJS): host port **8000** → container 3000
- MongoDB: internal only (no host port exposed)

## Services
| Service | Image | Notes |
|---|---|---|
| `mongodb` | `mongo:7` | Healthchecked; app waits for it. |
| `backend` | `node:20-alpine` | Bind-mounts `./Backend`, runs `npm run start:dev` (NestJS watch mode). |
| `frontend` | `node:20-alpine` | Bind-mounts `./frontend`, runs `npm run dev --host 0.0.0.0` (Vite). |

## Environment
- JWT secrets (`JWT_SECRET`, `JWT_REFRESH_SECRET`) are **required at boot** with no fallback in code. Dev placeholders are generated via `generate_development_secrets` and delivered through `/run/base44/app.env`. Repo-level `.env.base44-defaults` provides fallbacks so the app boots even before secrets land.
- `AI_PROVIDER=mock` by default — no external API key needed. Set `AI_PROVIDER=openai` + `OPENAI_API_KEY` for real AI summaries.
- `SEED_DEMO=true` creates a demo account on boot: `camille@acme.dev` / `demo1234` with a workspace "Acme Demo" and sample messages in `#general`.
- `CORS_ORIGIN` and `VITE_API_URL`/`VITE_WS_URL` use `BASE44_PUBLIC_HOST_SUFFIX` — never hardcode these.
- Vite `host: true` added to `vite.config.ts` so the dev server binds 0.0.0.0; `__VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS` is passed bare for allowed-host matching.

## Verifying
1. `docker compose ps` — all three services should be up/healthy.
2. `curl -s http://localhost:8000/api/health` → should return a health response.
3. `curl -s http://localhost:3000` → should return the Vite HTML shell.
4. Open the preview; log in with `camille@acme.dev` / `demo1234`.

## Quirks
- The backend's own `docker-compose.yml` (in `Backend/`) builds a production image — do NOT use it for dev. The Base44 compose at the repo root uses runtime images with bind mounts instead.
- `nest start --watch` compiles to `dist/`; a named volume (`backend_dist`) keeps build artifacts out of the host mount.
- Frontend has an MSW mock mode (`VITE_USE_MOCKS=true`) for backendless dev — not used here; we point at the real NestJS backend.
