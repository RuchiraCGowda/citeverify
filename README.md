# CiteVerify

Academic manuscript and bibliographic reference verification workbench powered by open scholarly registries (**Crossref**, **OpenAlex**, and **arXiv**).

**No Gemini API key or paid LLM dependency is required.**

## Architecture

- **Frontend (`/`)**: React + Vite + Tailwind CSS + TypeScript SPA deployed on **Vercel** (`dist/`).
- **Backend (`/backend`)**: FastAPI + Uvicorn + SQLAlchemy + `pypdf` + `httpx` + `rapidfuzz` deployed separately on **Render** (or any persistent Python host).
- **Database & Persistence**: Hosted **PostgreSQL** (`DATABASE_URL`) stores verification runs, citation comparison records, and uploaded PDF binaries (`BYTEA`, or optional persistent disk via `STORAGE_DIR`).

## Environment Variables

### Frontend (Vercel)

| Variable | Required | Description |
| :--- | :--- | :--- |
| `VITE_API_BASE_URL` | Yes (Production) | Public URL of your FastAPI backend (e.g. `https://citeverify-api.onrender.com`), with no trailing slash. Leave empty locally to use Vite's `/api` dev proxy. |
| `VITE_DEV_PROXY_TARGET` | No | Local Vite dev proxy target (defaults to `http://127.0.0.1:8000`). Ignored in production builds. |

### Backend (Render / Python Host)

| Variable | Required | Description |
| :--- | :--- | :--- |
| `DATABASE_URL` | Yes | Hosted PostgreSQL connection string (Neon, Supabase, Render Postgres, AWS RDS). Both `postgres://` and `postgresql://` are supported. |
| `CORS_ORIGINS` | Yes | Comma-separated allowed frontend origins (e.g. `https://citeverify.vercel.app,http://localhost:3000`). |
| `CORS_ORIGIN_REGEX` | No | Optional regex for Vercel preview URLs (e.g. `https://.*\.vercel\.app`). |
| `MAX_UPLOAD_MB` | No | Maximum uploaded PDF size in MB (default `25`). |
| `CROSSREF_MAILTO` | No | Optional email included in Crossref `User-Agent` header for polite pool routing. |
| `STORAGE_MODE` | No | Persistence strategy (`postgres` stores PDFs in PostgreSQL `BYTEA`). |
| `STORAGE_DIR` | No | Optional persistent disk mount directory (e.g. `/var/data/citeverify`) for storing PDFs on disk alongside DB metadata. |

## Deployment Steps

### 1. Provision Hosted PostgreSQL
Create a PostgreSQL database on Neon, Supabase, or Render Postgres and copy the connection string (`DATABASE_URL`). Tables (`verification_documents`, `citation_entries`) are created automatically on FastAPI startup.

### 2. Deploy FastAPI Backend on Render
1. Create a new **Web Service** connected to your GitHub repository.
2. Set **Root Directory** to `backend`.
3. Set **Build Command** to:
   ```bash
   pip install -r requirements.txt
   ```
4. Set **Start Command** to:
   ```bash
   uvicorn app.main:app --host 0.0.0.0 --port $PORT
   ```
5. Set **Health Check Path** to `/api/health`.
6. Add `DATABASE_URL` and `CORS_ORIGINS` (including your Vercel URL) in Render Environment Variables.

### 3. Deploy Frontend on Vercel
1. Import your GitHub repository in Vercel with **Root Directory** set to `.` (repository root).
2. Vercel auto-detects Vite (configured in `vercel.json`):
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
3. Add `VITE_API_BASE_URL` pointing to your Render FastAPI URL (e.g. `https://citeverify-api.onrender.com`).
4. Redeploy so Vite injects `VITE_API_BASE_URL` into the static bundle.

## Local Development

```bash
# Terminal 1: FastAPI backend
cd backend
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export DATABASE_URL="postgresql://postgres:postgres@localhost:5432/citeverify"
export CORS_ORIGINS="http://localhost:3000"
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 2: Frontend
npm install
npm run dev
```
