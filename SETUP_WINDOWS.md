# Windows Setup Guide

Two paths: **Docker (recommended, Steps A1–A6)** or **Native (Steps
B1–B20, no Docker)**. Pick one. Docker is far less error-prone on
Windows because it sidesteps PostgreSQL/Python/Node version conflicts.

---

## Path A: Docker (recommended)

### A1. Install required applications
- Install **Docker Desktop for Windows**: https://www.docker.com/products/docker-desktop/
  (requires WSL2 — the installer will prompt you to enable it if it isn't already).
- Install **Git for Windows**: https://git-scm.com/download/win
- Install **VS Code**: https://code.visualstudio.com/

**Expected result**: `docker --version`, `docker compose version`, and
`git --version` all print a version number in PowerShell.

**If it fails**: Docker Desktop requires Windows 10/11 64-bit with
virtualization enabled in BIOS. If `docker` is not recognized, restart
your terminal (and possibly your PC) after installation — PATH changes
often need a fresh shell.

### A2. Extract the ZIP and open in VS Code
- Right-click `AI-Sales-Revenue-Intelligence-Platform.zip` → Extract All.
- Open VS Code → File → Open Folder → select the extracted folder.

**Expected result**: VS Code's Explorer panel shows `backend/`,
`frontend/`, `docker-compose.yml`, etc.

### A3. Create your `.env` file
In VS Code's terminal (`` Ctrl+` ``):
```powershell
Copy-Item backend\.env.example backend\.env
```
Open `backend\.env` and set at minimum:
- `SECRET_KEY` — any long random string.
- `GROQ_API_KEY` (or `OPENAI_API_KEY`, or set `LLM_PROVIDER=ollama` if
  you have Ollama running) — needed only for the AI chatbot feature.

**Expected result**: `backend\.env` exists and no longer contains the
literal placeholder text for `SECRET_KEY`.

**If it fails**: If `Copy-Item` errors with "file not found", confirm
you're in the project root (`Get-Location` should end in
`...AI-Sales-Revenue-Intelligence-Platform`).

### A4. Start the full stack
```powershell
docker compose up --build
```

**Expected result**: after a few minutes (first build downloads/compiles
several large ML dependencies — Prophet, XGBoost, sentence-transformers
— this can take **10–20 minutes** on first run), you should see log
lines from `sales-intel-backend`, `sales-intel-frontend`, and
`sales-intel-db` with no `Exited` or repeated `Restarting` messages.

**If it fails**:
- `port is already allocated` → something else on your PC is using port
  3000, 8000, or 5432. Stop that process or edit the port mappings in
  `docker-compose.yml`.
- Build fails downloading a package → check your internet connection;
  corporate VPNs/firewalls sometimes block PyPI/npm registries.

### A5. Open the application
- Frontend: http://localhost:3000
- Backend API docs: http://localhost:8000/api/v1/docs
- Grafana: http://localhost:3001 (login `admin` / `admin`)

**Expected result**: the login page loads at localhost:3000.

### A6. Test login and core features
- Login with `admin@sales-intel.ai` / `Admin@12345` (auto-seeded on
  first startup, along with a synthetic demo dataset).
- You should land on the Executive Dashboard with real charts (not
  blank/error states) — the app auto-generates ~365 days of synthetic
  hospitality data on first boot.
- Try the **Bookings** and **AI Analyst** pages from the sidebar.

**If it fails**: check `docker compose logs backend` for errors. If the
dashboard is blank, confirm the demo data seeded: `docker compose logs
backend | Select-String "Generated"` should show seeding log lines.

### Stopping the application
```powershell
docker compose down          # stop and remove containers
docker compose down -v       # also delete the database volume (full reset)
```

---

## Path B: Native (no Docker)

### B1. Install required applications
Install, in order: **Python 3.11** (check "Add python.exe to PATH"
during install), **Git for Windows**, **PostgreSQL 16** (remember the
password you set for the `postgres` superuser), **Node.js 20 LTS**, **VS
Code**.

**Expected result**: `python --version`, `git --version`, `psql
--version`, `node --version`, `npm --version` all print version numbers.

**If it fails**: If `python` isn't recognized, reinstall and check "Add
to PATH", or use `py` instead of `python` on Windows.

### B2. Open VS Code
Launch VS Code.

### B3. Extract the ZIP
Right-click the zip → Extract All → choose a destination.

### B4. Open the project folder in VS Code
File → Open Folder → select the extracted folder.

**Expected result**: `backend/`, `frontend/` visible in Explorer.

### B5. Create a Python virtual environment
```powershell
cd backend
python -m venv venv
```
**Expected result**: a `backend\venv\` folder appears.

### B6. Activate the virtual environment
```powershell
.\venv\Scripts\Activate.ps1
```
**Expected result**: your prompt now starts with `(venv)`.

**If it fails**: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy
Bypass` then retry — PowerShell blocks script execution by default.

### B7. Install Python dependencies
```powershell
pip install -r requirements.txt
```
**Expected result**: completes with no red `ERROR` lines (this installs
~30 packages including Prophet/XGBoost and can take 5–15 minutes).

**If it fails**: Prophet's `cmdstanpy` dependency needs a C++ build
toolchain on some systems — if it fails to build, install "Build Tools
for Visual Studio" (C++ workload) from Microsoft, or fall back to Path A
(Docker), which avoids this entirely since the Linux container image
already has the needed toolchain.

### B8. Install frontend dependencies
```powershell
cd ..\frontend
npm install
```
**Expected result**: a `node_modules\` folder appears, no fatal errors.

### B9–B10. Configure and create the PostgreSQL database
Open a terminal and run:
```powershell
psql -U postgres
```
Then inside the `psql` prompt:
```sql
CREATE USER sales_admin WITH PASSWORD 'sales_secret_pw';
CREATE DATABASE sales_revenue_intelligence OWNER sales_admin;
\q
```
**Expected result**: no error messages from `psql`.

**If it fails**: if `psql` isn't recognized, add PostgreSQL's `bin`
folder (e.g. `C:\Program Files\PostgreSQL\16\bin`) to your PATH.

### B11. Create the schema
There are no Alembic migrations in this project (see
`FINAL_VERIFICATION.md`) — the schema is created automatically the first
time the backend starts (Step B15), via SQLAlchemy's `create_all()`. No
separate migration step is needed.

### B12. Load sample data
Also automatic on first backend startup (see `AUTO_SEED_DEMO_DATA` in
`.env`, default `true`) — generates ~365 days of synthetic bookings,
revenue, restaurant/spa sales, reviews, and employee records.

### B13. Create `.env`
```powershell
cd ..\backend
Copy-Item .env.example .env
```
Edit `.env`: set `POSTGRES_SERVER=localhost`, and the
`POSTGRES_USER`/`POSTGRES_PASSWORD`/`POSTGRES_DB` to match Step B9-10.

### B14. Configure API keys
In `.env`, set `GROQ_API_KEY` (or `OPENAI_API_KEY`, or
`LLM_PROVIDER=ollama`) if you want the AI chatbot to work. Everything
else in the app functions without this.

### B15. Start the backend
```powershell
# from backend\, with venv activated
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
**Expected result**: log output ending in `Application startup
complete.`, with lines about seeding the admin user and generating
demo data on first run.

**If it fails**: `could not connect to server` → PostgreSQL isn't
running or the credentials in `.env` don't match Step B9-10.

### B16. Start the frontend
Open a **second** terminal:
```powershell
cd frontend
npm run dev
```
**Expected result**: `Local: http://localhost:5173/` printed.

### B17. Open the application
Browse to http://localhost:5173

### B18. Test login
`admin@sales-intel.ai` / `Admin@12345`.

### B19. Test the dashboard
Confirm KPI cards and charts render with real numbers.

### B20. Test other major features
Bookings page, AI Analyst chat (if you configured an LLM key), and
`http://localhost:8000/api/v1/docs` to try any endpoint directly.

### Running tests (either path, requires `requirements-dev.txt`)
```powershell
cd backend
pip install -r requirements-dev.txt
pytest -v
```

### Stopping the application (native path)
Press `Ctrl+C` in both the backend and frontend terminals.
