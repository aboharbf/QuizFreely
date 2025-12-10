# QuizFreely Local Deployment Guide

A complete guide to running QuizFreely locally with proper environment isolation.

---

## JavaScript vs Python Virtual Environments

If you're coming from Python/conda, here's what you need to know:

### Python (conda/virtualenv)
```bash
conda create -n myproject python=3.10  # Create isolated environment
conda activate myproject                # Activate it
pip install -r requirements.txt        # Install packages in environment
```

### JavaScript (npm/Node.js)
```bash
# No separate environment creation!
npm install                            # Installs packages in ./node_modules/
```

**Key Differences:**

| Aspect | Python/conda | JavaScript/Node.js |
|--------|--------------|-------------------|
| **Isolation** | Separate environments you activate | Each project has its own `node_modules/` folder automatically |
| **Package location** | Centralized in conda/venv folder | Local `node_modules/` in each project |
| **Activation** | Required (`conda activate`) | Not needed - project uses local packages automatically |
| **Version management** | `conda` manages Python versions | `nvm`/`volta` manage Node.js versions |
| **Lock files** | `requirements.txt` / `environment.yml` | `package-lock.json` / `pnpm-lock.yaml` |
| **Package manager** | `pip` / `conda` | `npm` / `yarn` / `pnpm` |

**Think of it this way:** Every JavaScript project is like having an automatically activated conda environment. When you run `npm install`, it creates a `node_modules/` folder that's similar to a conda environment, but it lives right in your project folder.

---

## System Requirements

### Required Software

1. **Node.js** (v18 or higher)
   - Includes `npm` (Node Package Manager)
   - Download: https://nodejs.org/

2. **Go** (v1.21 or higher) - For the backend API
   - Download: https://go.dev/dl/

3. **Git** - For cloning repositories

### Optional but Recommended

- **nvm (Node Version Manager)** - Manage multiple Node.js versions
  - Windows: [nvm-windows](https://github.com/coreybutler/nvm-windows)
  - macOS/Linux: [nvm](https://github.com/nvm-sh/nvm)

---

## Installation Steps

### 1. Install Node.js Version Manager (Recommended)

**Why?** Like conda for Python, nvm lets you switch between Node.js versions per project.

**Windows:**
```powershell
# Download and install nvm-windows from:
# https://github.com/coreybutler/nvm-windows/releases

# After installation:
nvm install 18.19.0
nvm use 18.19.0
node --version  # Verify: should show v18.19.0
```

**macOS/Linux:**
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Restart terminal, then:
nvm install 18.19.0
nvm use 18.19.0
node --version  # Verify
```

**Without nvm:** Just install Node.js from https://nodejs.org/ (LTS version recommended)

### 2. Set Up Directory Structure

Create a parent folder for both repositories:

```bash
mkdir quizfreely-local
cd quizfreely-local
```

Your final structure will be:
```
quizfreely-local/
├── quizfreely/          # Web app (this repository)
└── api/                 # Backend API (separate repository)
```

---

## Part 1: Backend API Setup

The web app needs the backend API running first.

### Clone and Setup

```bash
cd quizfreely-local
git clone https://github.com/quizfreely/api.git
cd api
```

### Configure Environment

```bash
# Copy example environment file
cp .env.example .env
```

Edit `.env` with your preferred text editor:

```bash
# Required settings
PORT=8008
DB_TYPE=sqlite                    # Options: sqlite, postgres
DB_PATH=./quizfreely.db          # For sqlite
SECRET_KEY=your-secret-key-here  # Generate a random string

# Optional settings
ENABLE_REGISTRATION=true
ENABLE_OAUTH_GOOGLE=false
```

**Generate a secure SECRET_KEY:**

```bash
# On Windows (PowerShell):
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }))

# On macOS/Linux:
openssl rand -base64 32
```

### Install Go Dependencies

```bash
# Go manages dependencies differently - it downloads them on first build/run
# No separate "environment" needed, just ensure you're in the api/ folder

go mod download  # Downloads dependencies listed in go.mod
```

**Note:** Unlike Python's `pip install`, Go's `go mod download` doesn't install to a separate environment. Dependencies are cached globally but versioned per-project in `go.mod` (similar to `package.json`).

### Run the API

```bash
# Development mode (with hot reload if configured)
go run .

# Or build and run
go build -o api
./api  # On Windows: api.exe
```

The API should now be running at `http://localhost:8008`

**Verify it's working:**
```bash
# Open another terminal and test:
curl http://localhost:8008/health

# Or visit in browser: http://localhost:8008/graphql
# You should see the GraphQL playground
```

---

## Part 2: Web App Setup (This Repository)

### Clone and Navigate

```bash
cd quizfreely-local
git clone https://github.com/quizfreely/quizfreely.git
cd quizfreely/web
```

### Install Dependencies

```bash
# This creates node_modules/ folder with all dependencies
# Similar to: conda install --file requirements.txt
npm install

# Alternative package managers (faster):
# npm install -g pnpm && pnpm install  # Use pnpm (faster, more efficient)
# npm install -g yarn && yarn install  # Use yarn (alternative to npm)
```

**What just happened?**
- `npm install` read `package.json` (like `requirements.txt`)
- Created `node_modules/` folder in `web/` (like a local venv)
- Generated `package-lock.json` (exact versions, like `conda list --export`)

**Understanding node_modules:**
```
web/
├── node_modules/          # All dependencies (like site-packages in venv)
│   ├── svelte/
│   ├── vite/
│   └── ... (500+ folders - includes all dependencies and their dependencies)
├── package.json           # Dependency list (like requirements.txt)
├── package-lock.json      # Exact versions lock file (like environment.yml)
└── src/
```

### Configure Environment

```bash
# Still in quizfreely/web/ directory
cp .env.example .env
```

Edit `.env`:

```bash
# Server Configuration
PORT=8080
HOST=0.0.0.0

# Backend API Connection (matches API setup)
API_URL=http://localhost:8008
REALTIME_SERVER_URL=http://localhost:8000
REALTIME_SERVER_WS_URL=ws://localhost:8000

# Optional Features
ENABLE_OAUTH_GOOGLE=false
ENABLE_CLASSES=false

# Public Variables
PUBLIC_DOMAIN=localhost
```

### Run Development Server

```bash
# From quizfreely/web/ directory
npm run dev

# What this does:
# 1. Runs vite dev server (fast development server with hot reload)
# 2. Proxies /api/* requests to http://localhost:8008 (your backend API)
# 3. Watches for file changes and auto-reloads browser
```

The web app should now be running at `http://localhost:8080`

**Verify it's working:**
- Open browser to `http://localhost:8080`
- You should see the QuizFreely landing page
- Try signing up for an account
- Create a local studyset (works without API)

---

## Understanding the Setup

### What's Running?

```
┌─────────────────────────────────────────────────────────┐
│ Your Computer                                            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Terminal 1: Backend API                                │
│  ┌──────────────────────────────────┐                  │
│  │ Go API Server                    │                  │
│  │ Port: 8008                       │                  │
│  │ Database: ./quizfreely.db        │                  │
│  └──────────────────────────────────┘                  │
│                      ▲                                   │
│                      │ GraphQL requests                 │
│                      │                                   │
│  Terminal 2: Web App                                    │
│  ┌──────────────────────────────────┐                  │
│  │ Vite Dev Server                  │                  │
│  │ Port: 8080                       │                  │
│  │ Proxies /api → localhost:8008    │                  │
│  └──────────────────────────────────┘                  │
│                      ▲                                   │
│                      │ HTTP requests                    │
│                      │                                   │
│  Browser: http://localhost:8080                         │
│  ┌──────────────────────────────────┐                  │
│  │ QuizFreely UI                    │                  │
│  │ + IndexedDB (local storage)      │                  │
│  └──────────────────────────────────┘                  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Data Flow Example: Creating an Online Studyset

1. **User clicks "Create Studyset"** in browser
2. Browser sends request to `http://localhost:8080/studyset/create`
3. SvelteKit loads page, user fills form
4. User submits → JavaScript makes GraphQL request to `/api/graphql`
5. Vite dev server proxies request to `http://localhost:8008/graphql`
6. Go API processes request, saves to database
7. Response flows back: API → Vite → Browser
8. Browser updates UI with new studyset

### Data Flow Example: Creating a Local Studyset

1. **User clicks "Create Local Studyset"** in browser
2. Browser loads local studyset form
3. User fills form and submits
4. JavaScript calls `idb-api-layer.js` functions
5. Data saved directly to IndexedDB in browser (no network request)
6. No API involved - completely offline

---

## Daily Development Workflow

### Starting Your Work Session

```bash
# Like "conda activate myenv" but for both services

# Terminal 1: Start backend API
cd quizfreely-local/api
go run .

# Terminal 2: Start web app
cd quizfreely-local/quizfreely/web
npm run dev

# Now code away! Changes auto-reload in browser.
```

### Stopping Services

```bash
# In each terminal, press: Ctrl+C
# That's it! No "deactivate" needed like conda.
```

### Installing New Packages

**Backend API (Go):**
```bash
cd api/
go get github.com/some/package  # Adds to go.mod automatically
go mod tidy                      # Cleans up unused dependencies
```

**Web App (JavaScript):**
```bash
cd quizfreely/web/
npm install package-name         # Adds to package.json and node_modules/
# Equivalent to: pip install package-name
```

### Updating Dependencies

**Backend API (Go):**
```bash
cd api/
go get -u ./...     # Update all dependencies
go mod tidy
```

**Web App (JavaScript):**
```bash
cd quizfreely/web/
npm update          # Update to latest within semver ranges in package.json
# Or for major updates:
npm install package-name@latest
```

---

## Common Commands Reference

### Python/conda → JavaScript/npm Equivalents

| Python/conda | JavaScript/npm | Description |
|-------------|---------------|-------------|
| `conda create -n myenv` | *(not needed)* | Each project has its own `node_modules/` |
| `conda activate myenv` | `cd project/` | Just navigate to project - no activation |
| `pip install package` | `npm install package` | Install a package |
| `pip install -r requirements.txt` | `npm install` | Install all dependencies |
| `pip list` | `npm list` | List installed packages |
| `pip freeze > requirements.txt` | `npm shrinkwrap` | Lock dependency versions |
| `pip uninstall package` | `npm uninstall package` | Remove a package |
| `python script.py` | `node script.js` | Run a script |
| `conda env list` | `nvm list` | List installed versions |
| `conda deactivate` | *(not needed)* | No deactivation required |

### Project-Specific Commands

**Backend API:**
```bash
go run .              # Run in development mode
go build              # Compile to executable
go test ./...         # Run tests
go mod tidy           # Clean dependencies
```

**Web App:**
```bash
npm run dev           # Start development server (hot reload)
npm run build         # Build for production
npm run preview       # Preview production build
npm list              # List installed packages
npm outdated          # Check for updates
npm audit             # Check for security vulnerabilities
```

---

## Troubleshooting

### Port Already in Use

**Error:** `Port 8080 is already in use`

**Solution:**
```bash
# Find and kill the process
# Windows (PowerShell):
Get-Process -Id (Get-NetTCPConnection -LocalPort 8080).OwningProcess | Stop-Process

# macOS/Linux:
lsof -ti:8080 | xargs kill
```

**Or change the port in `.env`:**
```bash
PORT=3000  # Use a different port
```

### API Connection Refused

**Error:** `Failed to fetch` or `Connection refused to localhost:8008`

**Solution:**
1. Ensure backend API is running (check Terminal 1)
2. Verify `API_URL=http://localhost:8008` in `web/.env`
3. Test API directly: `curl http://localhost:8008/health`

### node_modules Issues

**Error:** `Module not found` or `Cannot find module`

**Solution:**
```bash
cd quizfreely/web/
rm -rf node_modules package-lock.json  # Clean slate
npm install                             # Reinstall everything
```

This is like deleting and recreating a conda environment.

### Go Build Errors

**Error:** `package X is not in GOROOT`

**Solution:**
```bash
cd api/
go mod download   # Re-download dependencies
go mod tidy       # Clean up
```

### Different Node.js Version

**Error:** `The engine "node" is incompatible with this module`

**Solution with nvm:**
```bash
nvm install 18.19.0
nvm use 18.19.0
```

**Check required version:**
```bash
# Look in package.json for "engines" field
cat web/package.json | grep -A 2 "engines"
```

---

## Environment Isolation Best Practices

### 1. Use .env Files (Never Commit!)

```bash
# quizfreely/web/.env - Local configuration
PORT=8080
API_URL=http://localhost:8008
SECRET_TOKEN=abc123  # Never commit this!
```

**Always in .gitignore:**
```gitignore
.env
.env.local
.env.*.local
```

### 2. Use nvm for Node.js Version Isolation

Create `.nvmrc` file in project root:
```
18.19.0
```

Then anyone can run:
```bash
cd quizfreely/web
nvm use  # Automatically uses version from .nvmrc
```

### 3. Lock Your Dependencies

**package-lock.json** (npm) locks exact versions:
```bash
# Commit this file! It ensures everyone gets same versions
git add package-lock.json
```

### 4. Separate Development vs Production

**Development:**
```bash
NODE_ENV=development npm run dev
```

**Production:**
```bash
NODE_ENV=production npm run build
NODE_ENV=production node build/
```

---

## Docker Alternative (Full Isolation)

If you want Python-style complete isolation, use Docker:

### docker-compose.yml

Create in `quizfreely-local/`:

```yaml
version: '3.8'

services:
  api:
    build: ./api
    ports:
      - "8008:8008"
    environment:
      - PORT=8008
      - DB_TYPE=sqlite
      - DB_PATH=/data/quizfreely.db
    volumes:
      - api-data:/data

  web:
    build: ./quizfreely/web
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - API_URL=http://api:8008
    depends_on:
      - api

volumes:
  api-data:
```

### Usage

```bash
# Start everything (completely isolated in containers)
docker-compose up

# Stop everything
docker-compose down

# This is the MOST similar to conda environments:
# - Complete isolation
# - Specific versions locked
# - Reproducible across machines
```

---

## Quick Start Checklist

- [ ] Install Node.js (v18+) and npm
- [ ] Install Go (v1.21+)
- [ ] Clone both repositories into `quizfreely-local/`
- [ ] Backend API:
  - [ ] `cd api && cp .env.example .env`
  - [ ] Edit `.env` with SECRET_KEY
  - [ ] `go mod download`
  - [ ] `go run .` → Running on port 8008
- [ ] Web App:
  - [ ] `cd quizfreely/web && cp .env.example .env`
  - [ ] Edit `.env` with API_URL
  - [ ] `npm install`
  - [ ] `npm run dev` → Running on port 8080
- [ ] Open browser: `http://localhost:8080`
- [ ] Test local studyset (offline) and online studyset (needs API)

---

## Key Takeaways for Python Developers

1. **No virtual environment activation** - Just `cd` into project and run commands
2. **Dependencies are local** - `node_modules/` is like a venv inside each project
3. **Lock files are critical** - `package-lock.json` should be committed (like `environment.yml`)
4. **Version managers exist** - `nvm` is the equivalent of `conda` for Node versions
5. **Multiple package managers** - `npm`, `yarn`, `pnpm` (like `pip` vs `conda`)
6. **Global installs are rare** - Almost everything installs per-project

**Bottom line:** JavaScript projects are inherently isolated. You don't need to remember to activate/deactivate environments - just navigate to the project and run commands. It's simpler in that way, but you trade explicit control for convention.
