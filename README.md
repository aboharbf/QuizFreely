# QuizFreely

QuizFreely is a free and open-source studying tool that helps students learn more effectively through flashcards, practice tests, spaced repetition, and multiplayer study games.

**Website:** https://quizfreely.org
**Repositories:** [Codeberg](https://codeberg.org/quizfreely/quizfreely) · [GitHub](https://github.com/quizfreely/quizfreely)
**License:** AGPL-3.0-or-later

---

## What is QuizFreely?

QuizFreely is a complete studying platform consisting of two main parts:

1. **This repository (`quizfreely/quizfreely`)** - The web application (frontend) that users interact with in their browser, written in JavaScript using SvelteKit
2. **Backend API (`quizfreely/api`)** - The server that handles user accounts and data storage, written in Go with GraphQL (a modern way for applications to request specific data from servers)

This README focuses on the web application.

---

## Key Features

### Study Modes
- **Flashcards** - Traditional card-based studying with rich text support
- **Practice Tests** - Quiz yourself with multiple choice, true/false, and free response questions
- **Review Mode** - Smart spaced repetition system that shows you terms at optimal intervals based on how well you know them
- **Statistics** - Track your progress, accuracy, and improvement over time with visual charts

### Collaboration & Competition
- **Multiplayer Games** - Host or join real-time study games with friends using game codes
- **Public Studysets** - Browse and use study materials created by other users
- **Categories & Subjects** - Organize and discover content by topic

### Offline-First Design
QuizFreely works even without an internet connection. Your study data is stored locally (on your own device using IndexedDB, a browser-based storage system), so you can study anywhere. When you're back online, your progress syncs automatically with the server.

### Rich Content Editing
Create study materials with formatted text including bold, italic, underline, superscript, and subscript using ProseMirror (a professional text editing framework).

---

## Architecture Overview

### Two-Repository Design

**Web App (This Repo)** ↔️ **Backend API (Separate Repo)**

The web app runs in your browser and communicates with the backend API in two ways:
- **GraphQL requests** - Fetching and saving user data (like a waiter taking your order to the kitchen)
- **WebSocket connections** - Real-time multiplayer games (like a phone call that stays open for instant communication)

### Offline-First Architecture

```
User's Browser
    ├── Online Mode: Studyset stored on server (/studysets/[id]/)
    └── Local Mode: Studyset stored in IndexedDB (/studyset/local/)
```

The app provides parallel experiences:
- **Online studysets** - Stored on the server, accessible from any device
- **Local studysets** - Stored only on your device, work without internet

This dual approach ensures you can always study, regardless of connectivity.

### Technology Stack

**Framework & Build Tools:**
- **SvelteKit** (v2.16) - Modern web framework that makes building interactive apps simpler
- **Svelte 5** (v5.19) - The UI library using "runes" (a new reactive programming pattern)
- **Vite** (v6.0) - Fast development server and build tool
- **Node.js** - JavaScript runtime for the production server

**Rich Text Editing:**
- **ProseMirror** - Professional text editor that powers the term/definition editing
  - `prosemirror-state` - Manages what's currently in the editor
  - `prosemirror-view` - Displays the editor on screen
  - `prosemirror-model` - Defines what types of content are allowed
  - `prosemirror-commands` - Handles keyboard shortcuts and formatting

**Data Storage:**
- **Dexie** (v4.0) - Wrapper around IndexedDB that makes browser storage easier to use
- **IndexedDB** - Built-in browser database for storing studysets, progress, and statistics offline

**Data Visualization:**
- **Chart.js** (v4.5) - Creates the graphs and charts in the statistics pages
- **Luxon** (v3.7) - Handles dates and times (like "studied 2 hours ago")

**UI Components:**
- **Floating UI** - Positions tooltips and dropdowns intelligently
- **svelte-confetti** - Celebration animations when you do well
- **nprogress** - Loading bar at the top when navigating between pages

---

## Project Structure

```
QuizFreely/
├── web/                          # Main web application
│   ├── src/
│   │   ├── routes/              # Pages and API endpoints (SvelteKit file-based routing)
│   │   │   ├── +layout.svelte   # Root layout (header, footer, navigation)
│   │   │   ├── +page.svelte     # Homepage (landing or dashboard)
│   │   │   ├── dashboard/       # User dashboard
│   │   │   ├── studysets/[id]/  # Online studyset pages (view, test, review, stats)
│   │   │   ├── studyset/local/  # Local studyset pages (offline versions)
│   │   │   ├── explore/         # Browse public studysets
│   │   │   ├── search/          # Search functionality
│   │   │   ├── sign-in/         # Authentication
│   │   │   ├── sign-up/         # Registration
│   │   │   ├── settings/        # User preferences
│   │   │   ├── host/            # Host multiplayer games
│   │   │   └── join/            # Join multiplayer games
│   │   │
│   │   ├── lib/                 # Reusable code and utilities
│   │   │   ├── components/      # UI components (Header, Footer, StudysetList, etc.)
│   │   │   ├── icons/           # 43 SVG icons
│   │   │   ├── questionComponents/  # MCQ, FRQ, TrueFalse question types
│   │   │   ├── multiplayer/     # Game, Lobby, Leaderboard components
│   │   │   ├── stores/          # Shared state (nprogress timeout)
│   │   │   ├── idb-api-layer/   # IndexedDB interface
│   │   │   │   ├── db.js        # Database schema (14 versions with migrations)
│   │   │   │   └── idb-api-layer.js  # Local studyset CRUD operations
│   │   │   ├── proseMirrorEditor.js  # Rich text editor factory
│   │   │   ├── proseMirrorSchema.js  # Document structure definition
│   │   │   ├── themes.js        # Available themes (auto, dark, light)
│   │   │   └── fetchAuthData.server.js  # Server-side auth fetching
│   │   │
│   │   ├── app.html             # HTML shell template
│   │   └── hooks.server.js      # SvelteKit server initialization & middleware
│   │
│   ├── static/                   # Static assets (favicon, fonts, CSS)
│   │   └── immutable/           # Cached assets (themes, fonts, CSS framework)
│   │
│   ├── package.json             # Dependencies and build scripts
│   ├── svelte.config.js         # SvelteKit configuration
│   ├── vite.config.js           # Dev server & proxy configuration
│   └── .env.example             # Environment variable template
│
├── config/                       # Production server configuration
│   ├── Caddyfile                # Reverse proxy configuration
│   └── quizfreely-web.service   # Systemd service for auto-start
│
├── docs/                        # Developer documentation
│   └── dev/                     # Setup guides and technical specs
│       ├── web/                 # Web app documentation
│       └── api/                 # Backend API documentation
│
└── .github/workflows/           # Automated deployment and mirroring
    ├── deploy-prod.yml          # Production deployment automation
    ├── push-to-codeberg.yml     # Mirror to Codeberg
    └── push-to-gh.yml           # Mirror to GitHub
```

### Key Files Explained

- **`web/src/hooks.server.js`** - Validates environment variables and injects the selected theme into every page
- **`web/src/lib/idb-api-layer/db.js`** - Defines the local database structure with 6 tables and 14 schema versions
- **`web/src/routes/+layout.svelte`** - The wrapper around all pages (includes header and footer)
- **`web/vite.config.js`** - Configures the development server to forward API requests to the backend
- **`config/Caddyfile`** - Production web server that serves the app and routes API requests

---

## Getting Started

### Prerequisites

- **Node.js** (v18 or higher recommended)
- **npm** (comes with Node.js)
- **Backend API** running (see [quizfreely/api](https://codeberg.org/quizfreely/api))

> **Coming from Python/conda?** See the [Local Deployment Guide](docs/dev/LOCAL_DEPLOYMENT_GUIDE.md) for a comprehensive explanation of how JavaScript dependency management compares to Python virtual environments, plus complete setup instructions for running both the backend API and web app locally.

### Development Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/quizfreely/quizfreely.git
   cd quizfreely
   ```

2. **Install dependencies**
   ```bash
   cd web
   npm install
   ```

3. **Configure environment variables**
   ```bash
   cp .env.example .env
   ```

   Edit `.env` and set:
   - `API_URL` - Your backend API URL (default: http://localhost:8008)
   - `REALTIME_SERVER_URL` - HTTP endpoint for multiplayer (default: http://localhost:8000)
   - `REALTIME_SERVER_WS_URL` - WebSocket endpoint (default: ws://localhost:8000)
   - `PORT` - Port to run the dev server (default: 8080)
   - `HOST` - Host address (default: 0.0.0.0)

4. **Start the development server**
   ```bash
   npm run dev
   ```

   The app will be available at `http://localhost:8080`

### Production Build

1. **Build the application**
   ```bash
   npm run build
   ```

2. **Run the production server**
   ```bash
   node --env-file=.env build/
   ```

---

## Deployment

### Traditional Server Deployment

QuizFreely uses a straightforward Node.js deployment:

1. Build the application with `npm run build`
2. Copy the `build/` folder, `package.json`, and `node_modules/` to your server
3. Run with `node --env-file=.env build/`

For production, use a reverse proxy like Caddy (see `config/Caddyfile`) or nginx to:
- Handle HTTPS
- Serve static files efficiently
- Forward API requests to the backend

The repository includes:
- **Systemd service** (`config/quizfreely-web.service`) - Auto-start on server boot
- **GitHub Actions** (`.github/workflows/deploy-prod.yml`) - Automated deployment via SSH

### Docker Deployment

While QuizFreely doesn't currently include Docker configuration files, you can containerize it:

**Recommended approach for Docker:**

1. **Multi-stage Dockerfile** for the web app:
   ```dockerfile
   # Build stage
   FROM node:18-alpine AS builder
   WORKDIR /app
   COPY web/package*.json ./
   RUN npm ci
   COPY web/ ./
   RUN npm run build

   # Production stage
   FROM node:18-alpine
   WORKDIR /app
   COPY --from=builder /app/build ./build
   COPY --from=builder /app/package*.json ./
   COPY --from=builder /app/node_modules ./node_modules
   ENV PORT=8080 HOST=0.0.0.0
   EXPOSE 8080
   CMD ["node", "build/"]
   ```

2. **Docker Compose** to run both frontend and backend together:
   - Frontend container (this web app)
   - Backend API container (quizfreely/api)
   - Shared network for communication
   - Volume for persistent data

3. **Environment variables** via Docker Compose or `.env` file mounted as a volume

**Note:** You'll need to set up the backend API separately. See the [API repository documentation](https://codeberg.org/quizfreely/api) for its Docker configuration.

---

## How It Works: Technical Flow

### 1. User Opens the App

1. Browser requests the page from the server
2. **`hooks.server.js`** runs on the server:
   - Checks required environment variables exist
   - Reads the user's theme preference from cookies
   - Injects the theme CSS into the HTML before sending to browser
3. **`+layout.svelte`** loads in the browser:
   - Renders the header and footer
   - Sets up navigation progress bar
4. **`+page.svelte`** determines if user should see landing page or dashboard

### 2. User Views a Studyset

**Online Studyset:**
1. User navigates to `/studysets/123`
2. **`+page.server.js`** runs on the server:
   - Reads the auth token from cookies
   - Makes a GraphQL request to the backend API
   - Fetches studyset data and all terms
3. Data is passed to **`+page.svelte`** which displays it

**Local Studyset:**
1. User navigates to `/studyset/local?id=456`
2. **`+page.svelte`** runs in the browser:
   - Calls `idb-api-layer.js` functions
   - Queries IndexedDB directly (no network request)
   - Displays the stored data

### 3. User Takes a Practice Test

1. Component loads terms from either server or IndexedDB
2. Renders appropriate question component (MCQ, FRQ, or TrueFalse)
3. User answers questions, component tracks correctness
4. Results are saved:
   - **Online:** GraphQL mutation to backend API
   - **Local:** Stored in IndexedDB `termProgress` and `termProgressHistory` tables
5. Statistics page queries this data and renders charts with Chart.js

### 4. User Plays a Multiplayer Game

1. Host navigates to `/host` and picks a studyset
2. Frontend opens WebSocket connection to `REALTIME_SERVER_WS_URL`
3. Backend generates a game code and creates a game room
4. Other players enter code at `/join` and connect to same room
5. WebSocket messages flow between players in real-time:
   - Questions appear simultaneously
   - Answers are collected
   - Leaderboard updates live
6. **`Game.svelte`** manages the WebSocket connection and game state

---

## Data Models

### IndexedDB Tables (Local Storage)

The app uses 6 tables with relationships:

1. **`studysets`** - Studyset metadata (title, folder, subject)
2. **`terms`** - Individual flashcards (term, definition, type, distractor options)
3. **`termProgress`** - Current learning state for spaced repetition (next review date, interval)
4. **`termProgressHistory`** - Historical performance records
5. **`termConfusionPairs`** - Tracks which terms you commonly confuse
6. **`practiceTests`** - Saved practice test results

**Schema version:** 14 (with automatic migration when database structure changes)

### Example: How Spaced Repetition Works

When you review a term:
1. You rate how well you knew it (e.g., "hard", "good", "easy")
2. Algorithm calculates the next review date based on:
   - Previous interval (time since last review)
   - Your rating
   - Total number of reviews
3. Updated `termProgress` record saves new interval and next review date
4. New entry added to `termProgressHistory` for statistics
5. Review Mode queries terms where next review date ≤ today

---

## Contributing

QuizFreely is open source and welcomes contributions!

### Areas to Contribute

- **Testing** - The project currently has no automated tests (great opportunity!)
- **Accessibility** - Improving keyboard navigation and screen reader support
- **Documentation** - Expanding guides in the `docs/` folder
- **Features** - Check issues on Codeberg or GitHub
- **Bug Fixes** - Report issues or submit fixes
- **Translations** - Internationalization isn't implemented yet

### Development Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test manually (no automated tests yet)
5. Commit with clear messages (`git commit -m 'Add amazing feature'`)
6. Push to your fork (`git push origin feature/amazing-feature`)
7. Open a Pull Request on Codeberg or GitHub

---

## Environment Variables Reference

Create a `.env` file in the `web/` directory (copy from `.env.example`):

```bash
# Server Configuration
PORT=8080                          # Port for development/production server
HOST=0.0.0.0                       # Host address (0.0.0.0 allows external connections)

# Backend API (Required)
API_URL=http://localhost:8008      # Backend API base URL
REALTIME_SERVER_URL=http://localhost:8000       # Multiplayer HTTP endpoint
REALTIME_SERVER_WS_URL=ws://localhost:8000      # Multiplayer WebSocket endpoint

# Optional Features
ENABLE_OAUTH_GOOGLE=false          # Enable Google OAuth sign-in
ENABLE_CLASSES=false               # Enable classes feature (experimental)
CLASSES_API_URL=http://localhost:3000  # Classes API URL if enabled

# Public Variables (Available in Browser)
PUBLIC_DOMAIN=quizfreely.org       # Your domain (for links and metadata)
```

**Important:** Variables prefixed with `PUBLIC_` are embedded in the client-side code and visible to users. Never put secrets in `PUBLIC_*` variables.

---

## Related Repositories

- **Backend API**: [quizfreely/api](https://codeberg.org/quizfreely/api) - GraphQL server in Go
- **Main Website**: [quizfreely.org](https://quizfreely.org) - Production instance

---

## License

This project is licensed under the **AGPL-3.0-or-later** license. This means:
- You can use, modify, and distribute this software freely
- If you run a modified version on a server accessible to others, you must make your source code available
- Any derivative work must also be licensed under AGPL-3.0

See the full license text in the repository.

---

## Questions or Issues?

- **Documentation**: Check the `docs/dev/` folder for detailed setup guides
- **Issues**: Report bugs or request features on [Codeberg](https://codeberg.org/quizfreely/quizfreely/issues) or [GitHub](https://github.com/quizfreely/quizfreely/issues)
- **Website**: Visit [quizfreely.org](https://quizfreely.org) to see the live app

---

**Built with ❤️ by the QuizFreely community**
