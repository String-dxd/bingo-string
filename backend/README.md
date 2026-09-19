# Bingo Backend API

Express.js backend for the String Bingo application with PostgreSQL/NeonDB and Prisma ORM.

## Setup

### 1. Install Dependencies
```bash
cd backend
npm install
```

### 2. Environment Variables
Copy `.env.example` to `.env` and configure:
```env
DATABASE_URL="your_neondb_connection_string"
GOOGLE_CLIENT_ID="your_google_oauth_client_id"
PORT=3000
NODE_ENV=development
CORS_ORIGINS="http://localhost:5173,https://string.sg"
```

### 3. Database Setup
```bash
# Generate Prisma client
npm run prisma:generate

# Run database migrations
npm run prisma:migrate

# (Optional) Open Prisma Studio
npm run prisma:studio
```

### 4. Development
```bash
# Start development server with hot reload
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

## API Endpoints

### Authentication
- `POST /api/auth/login` - Login with Google token
- `GET /api/auth/me` - Get current user info
- `POST /api/auth/logout` - Logout (analytics)

### Games
- `GET /api/games` - Get user's games (auth required)
- `POST /api/games` - Create new game (auth required)
- `GET /api/games/:id` - Get specific game (public or owner)
- `PUT /api/games/:id` - Update game (owner only)
- `DELETE /api/games/:id` - Delete game (owner only)
- `GET /api/games/:id/play` - Get game for playing (public)

### Sessions and Results
- `POST /api/games/:id/join` - Join a custom game with a session ID and player name
- `POST /api/games/:id/progress` - Update custom game progress
- `GET /api/games/:id/results` - Get custom game results (authenticated owner only)
- `POST /api/default-sessions` - Create an anonymous default session, or return the existing session
- `POST /api/default-sessions/:sessionId/progress` - Replace default session progress and completion state
- `GET /api/default-sessions/:sessionId` - Get a default session

Default session endpoints do not require login. The session ID is used to retrieve
or update a session; these endpoints are not an administrative analytics API.

### Health Check
- `GET /health` - Server health status

## Database Schema

### Users
- `id` - Unique identifier
- `email` - User email (unique)
- `name` - Display name
- `picture` - Profile picture URL
- `googleId` - Google OAuth user ID
- `createdAt`, `updatedAt` - Timestamps

### Games
- `id` - Unique identifier
- `name` - Game name
- `creatorEmail` - Creator's email
- `challengesJson` - Array of 25 challenges (JSON)
- `isPublic` - Whether game is publicly accessible
- `createdAt`, `updatedAt` - Timestamps

### GameSessions
- `id` - Unique identifier
- `gameId` - Reference to game
- `sessionId` - Client-generated identifier (unique together with `gameId`)
- `playerName` - Player-provided display name
- `progressJson` - Game progress data (JSON)
- `isCompleted` - Whether the player has achieved bingo
- `completedAt` - Completion timestamp
- `createdAt`, `updatedAt` - Timestamps

### DefaultGameSessions
- `id` - Unique identifier
- `sessionId` - Unique client-generated identifier
- `challengeSet` - Challenge CSV filename
- `gridSize` - Reported grid side length
- `progressJson` - Latest completed cell indices (JSON)
- `isCompleted` - Whether the player has achieved bingo
- `completedAt` - Completion timestamp
- `createdAt`, `updatedAt` - Timestamps

## Analytics Review and Proposed Scope (#56)

This is a source-code review, not confirmation of production database migrations
or successful Google Analytics delivery. The original "Better analytics" issue
remains relevant for measurement quality and reporting, but Phase 1 should not be
implemented again.

### Existing coverage

| Original requirement | Current implementation |
| --- | --- |
| Phase 1: default session storage, API, frontend integration | Present: `DefaultGameSession`, the `20251208012308_add_default_game_sessions` migration, the endpoints above, and `BingoApp.initializeDefaultGameSession()`. A UUID is persisted under `defaultBingoSession` in localStorage and reused on reload. |
| Phase 2: game metrics | Partial: the database holds the latest cell indices and bingo state; GA event calls exist for `default_game_start`, `default_cell_completed`, and `default_game_completed`. No challenge-level aggregates exist. |
| Phase 2: session and performance metrics | Creation/update/completion timestamps exist, but no explicit active-duration, interaction-history, drop-off, load-time, or error-rate instrumentation exists. Console/server logs are not an error-rate metric. |
| Phase 3: analytics dashboard | Not implemented for default games. The existing dashboard shows custom-game results to their creator, not site-wide default-game statistics or trends. |

GA is optional: development reads `VITE_GOOGLE_ANALYTICS_ID`; runtime configuration
uses `googleAnalyticsId`. Production configuration supplies it; staging currently
does not. Database tracking operates independently of GA.

### Measurement limitations to resolve first

- **Missing starts:** `initializeDefaultGameSession()` attempts `default_game_start`
  before `setupAnalytics()` defines `gtag`, so the event is skipped on a normal
  fresh page load.
- **Sessions are not plays or people:** the default UUID has no expiry and reset
  does not rotate it. Changing the challenge set also reuses it; the create
  endpoint returns existing metadata unchanged. Row counts cannot be described
  as distinct plays or unique users.
- **Inaccurate completion state:** photo confirmation sends progress, but photo
  removal and reset do not. Reload restores images, not the grid's completed-cell
  set. Subsequent updates can therefore overwrite progress with incomplete state.
- **Repeated completions:** after bingo, subsequent successful progress updates
  emit `default_game_completed` again and replace `completedAt`. Completion means
  a bingo line, not every cell completed, and the timestamp is not reliably the
  first bingo.
- **Unreliable challenge denominators:** tracking reports `CONFIG.GRID.size`
  (currently 3), while default `setupGrid()` renders its default size of 5.
  Progress and GA labels use cell indices, not the CSV challenge IDs; filenames
  alone do not version challenge content.
- **No active-time or drop-off evidence:** `updatedAt - createdAt` measures elapsed
  time between database writes, including idle time and resumed visits. A latest
  progress snapshot does not reveal interaction order or when a player left.

### Privacy boundary

The default-session schema has no name, email, user relation, or photo field, and
the frontend sends only session metadata and cell indices. Custom sessions are
different: they include player-provided names. Persistent UUIDs still link visits,
and `progressJson` currently accepts arbitrary JSON rather than enforcing the
frontend's minimal payload.

Do not equate this schema with end-to-end anonymity: GA is a separate third-party
collector, and Morgan's `combined` access logs include IP address, request URL,
referrer, and user agent. No analytics consent/opt-out flow or session retention
cleanup is implemented. Decide disclosure, consent where applicable, retention,
and log redaction before expanding collection. Do not send names, emails, photos,
free text, or raw session IDs to GA or aggregate reports.

### Recommended follow-up work, in order

1. **Tracking correctness and privacy contract (next implementation issue).**
   Define a play as a new/reset/expired board, with a documented inactivity
   timeout; reload within that window resumes it. Rotate on challenge-set/version
   changes. Initialize permitted GA tracking before emitting events; count start
   and first bingo once per play. Synchronize removal/reset/reload state, preserve
   first-bingo time, and report actual board dimensions. Validate bounded session
   metadata and unique in-range cell indices on the backend. Specify and implement
   retention/cleanup and tracking controls.
   **Acceptance:** focused tests cover new/resumed/reset/expired sessions, changed
   challenge sets, first/repeated bingo, removal, reload, malformed payloads, and
   GA-disabled/API-failure paths; analytics failure must not prevent play.
2. **Minimal aggregate reporting, after reliable collection.**
   Record stable challenge IDs, content version, and which challenges were shown.
   Define bingo rate as plays reaching first bingo / eligible started plays,
   challenge completion rate as plays completing a challenge / plays shown it,
   and time-to-first-bingo separately from active duration. Specify reporting
   windows, treatment of unfinished plays, and exclusion of legacy unreliable
   records. Add an authenticated, explicitly authorized site-admin aggregate API
   and a small dashboard for starts, bingo rate, and challenge performance over
   time. Being a custom-game creator must not grant site-wide access.
   **Acceptance:** known-data tests verify denominators, empty windows, legacy
   exclusions, and authorization; reports expose no raw sessions or player data.
3. **Separate discovery for engagement and performance.**
   Define foreground-active time, a drop-off timeout/funnel, load-time boundaries,
   and an error-rate denominator before choosing instrumentation. Evaluate GA's
   existing reports before adding event storage. Any added telemetry should use
   bounded categories, sampling, retention, and no identifying URLs/error payloads.
   **Acceptance:** idle/background tabs do not inflate active time, absent unload
   events do not imply completion, and tests verify redaction and failure isolation.

Treat Phase 1 as implemented but needing verification, and split Phases 2–3 into
these follow-ups rather than closing them as delivered. This review changes
documentation only; it does not fix the measurement gaps or add telemetry.

## Security Features

- Google OAuth JWT validation
- CORS protection
- Helmet security headers
- Request rate limiting
- Input validation
- SQL injection protection (Prisma)

## Deployment

This backend can be deployed to:
- Railway
- Render
- Vercel (serverless functions)
- Heroku
- Any Node.js hosting platform

Make sure to:
1. Set all environment variables
2. Run database migrations
3. Configure CORS origins for your frontend domain