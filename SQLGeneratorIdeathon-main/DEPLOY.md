# Deploy SQL Generator to Render

Your app runs as **one Web Service**: the backend serves the API and the built frontend. Use either the **Dashboard** flow or the **Blueprint** (render.yaml).

---

## Option A: Deploy with Dashboard (step-by-step)

### 1. Push code to GitHub/GitLab/Bitbucket

Make sure your repo is pushed (including `Dockerfile`, `backend/`, and `frontend/`).

### 2. Create a Render account and connect the repo

- Go to [render.com](https://render.com) and sign up (or log in).
- **Dashboard** → **New** → **Web Service**.
- Connect your Git provider and select this repository.
- Choose the branch to deploy (e.g. `main`).

### 3. Configure the Web Service

| Field | Value |
|-------|--------|
| **Name** | `sql-generator` (or any name) |
| **Region** | Choose one (e.g. Oregon) |
| **Runtime** | **Docker** |
| **Dockerfile path** | `Dockerfile` (root of repo) |
| **Instance type** | Free (or paid if you prefer) |

Leave **Build Command** and **Start Command** empty; the Dockerfile defines the build and start.

### 4. Environment variables

In **Environment** add:

| Key | Value | Notes |
|-----|--------|--------|
| `JWT_SECRET` | (generate a long random string) | Required for auth; use “Generate” in Render |
| `OPENAI_API_KEY` | your key | If you use OpenAI |
| `ANTHROPIC_API_KEY` | your key | If you use Anthropic |
| `GEMINI_API_KEY` | your key | If you use Gemini |

Do **not** set `PORT`; Render sets it automatically.

Optional: for persistent data, create a **PostgreSQL** database in Render and set:

- `DATABASE_URL` = the **Internal Database URL** from the database’s Render dashboard.

(Default is SQLite; on the free tier the filesystem is ephemeral, so data is lost on redeploy.)

### 5. Deploy

Click **Create Web Service**. Render will build the Docker image (frontend + backend) and start the app. When the build finishes, open the service URL (e.g. `https://sql-generator-xxxx.onrender.com`).

---

## Option B: Deploy with Blueprint (render.yaml)

1. Push the repo (including `render.yaml` and `Dockerfile`).
2. In Render: **Dashboard** → **New** → **Blueprint**.
3. Connect the repo; Render will read `render.yaml` and create the Web Service.
4. After the service is created, open the **Environment** tab and add:
   - `JWT_SECRET` (or use “Generate”).
   - Any API keys you use: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`.
5. Trigger a deploy if needed; then open the service URL.

---

## After deploy

- **URL**: One URL serves both the app and the API, e.g. `https://your-service.onrender.com`.
- **API**: Backend is under `/api` (e.g. `https://your-service.onrender.com/api/health`).
- **Free tier**: The service may spin down after ~15 minutes of no traffic; the first request after that can take 30–60 seconds (cold start).

---

## Troubleshooting

- **Build fails on frontend**: The Dockerfile uses `npm install` so it works even without `package-lock.json`. If the frontend build fails, check the **Build logs** for the exact error (e.g. missing env, Node version).
- **Build fails on pip**: Check that `backend/requirements.txt` is committed and has no typos. If a package fails to install, you may need to pin versions.
- **502 / service not starting**: 
  - In Render, open your service → **Logs** and check the **Deploy log** and **Runtime log** for errors.
  - The app uses `backend/run.py` as the start command and reads `PORT` from the environment; do **not** set a custom Start Command in Render when using Docker.
  - Ensure **Runtime** is **Docker** and **Dockerfile path** is `Dockerfile` (at repo root).
- **Blank page or "Cannot GET /"**: The backend serves the built frontend from `frontend/dist`. If the Docker build didn’t copy `frontend/dist`, the build step may have failed; check that the frontend build step in the Docker image completed successfully.
- **Database**: For persistent users and data, add a Render PostgreSQL database and set `DATABASE_URL` to its internal URL.
- **Still stuck**: Copy the exact error from Render **Logs** (build or runtime) and use it to debug or ask for help.
