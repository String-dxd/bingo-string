# Development Guide

Welcome to the String Bingo development guide! This document provides comprehensive information for developers working on the String-sg/bingo-string project.

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Initial Setup](#initial-setup)
- [Frontend Development](#frontend-development)
- [Backend Development](#backend-development)
- [Database Management](#database-management)
- [Environment Configuration](#environment-configuration)
- [Development Workflow](#development-workflow)
- [Building and Deployment](#building-and-deployment)
- [Testing](#testing)
- [Contributing Guidelines](#contributing-guidelines)
- [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)
- [Project Structure](#project-structure)

## Project Overview

String Bingo is an interactive web-based bingo game developed for the String community. Users can play pre-configured bingo games or create their own custom bingo boards with personalized challenges.

**Key Features:**
- Interactive 5x5 bingo grid with customizable challenges
- Google OAuth authentication for game creation and saving
- Mobile-responsive design with touch support
- Camera integration for photo challenges
- Image download functionality
- Public and private game modes
- Session tracking and progress saving

**Tech Stack:**
- **Frontend**: Vanilla JavaScript, Vite, HTML5, CSS3
- **Backend**: Express.js, TypeScript, Prisma ORM
- **Database**: PostgreSQL (NeonDB)
- **Authentication**: Google OAuth 2.0
- **Deployment**: Vercel (Frontend + Serverless Functions), GitHub Pages (Static Assets)

## Architecture

The project follows a client-server architecture:

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────┐
│   Frontend      │────────▶│  Backend API     │────────▶│  Database   │
│   (Vite)        │◀────────│  (Express + TS)  │◀────────│ (PostgreSQL)│
└─────────────────┘         └──────────────────┘         └─────────────┘
      │                              │
      │                              │
      ▼                              ▼
┌─────────────────┐         ┌──────────────────┐
│  Static Assets  │         │  Serverless API  │
│  (GitHub Pages) │         │    (Vercel)      │
└─────────────────┘         └──────────────────┘
```

### Frontend Components

- **app.js**: Main application controller
- **bingoGrid.js**: Bingo grid rendering and state management
- **authManager.js**: Google OAuth authentication handling
- **challengeLoader.js**: Challenge data loading and management
- **cameraManager.js**: Camera integration for photo challenges
- **modalManager.js**: Modal dialog management
- **touchManager.js**: Mobile touch and zoom interactions
- **game-creation.js**: Custom game creation interface

### Backend Components

- **API Routes**: RESTful endpoints for games, authentication, and sessions
- **Middleware**: Authentication, CORS, security headers, error handling
- **Prisma ORM**: Database schema and query management
- **Google OAuth**: Token validation and user authentication

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js**: v18.x or higher
- **npm**: v9.x or higher
- **Git**: Latest version
- **PostgreSQL**: v15.x or higher (or NeonDB account)
- **Google Cloud Console Access**: For OAuth credentials

## Initial Setup

### 1. Clone the Repository

```bash
git clone https://github.com/String-sg/bingo-string.git
cd bingo-string
```

### 2. Install Dependencies

```bash
# Install root dependencies
npm install

# Install backend dependencies
cd backend
npm install
cd ..
```

### 3. Set Up Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google Identity Services API
4. Create OAuth 2.0 credentials:
   - Application type: Web application
   - Authorized JavaScript origins: `http://localhost:5173`, `https://bingo.string.sg`
   - Authorized redirect URIs: `http://localhost:5173`, `https://bingo.string.sg`
5. Copy the Client ID for environment configuration

### 4. Configure Environment Variables

Create `.env` files based on the examples:

**Root `.env`** (Frontend):
```bash
cp .env.example .env
```

Edit `.env`:
```env
VITE_GOOGLE_CLIENT_ID=your_google_client_id_here
VITE_API_BASE_URL=http://localhost:3000/api
```

**Backend `.env`**:
```bash
cd backend
cp .env.example .env
```

Edit `backend/.env`:
```env
DATABASE_URL="postgresql://username:password@localhost:5432/bingo_db?sslmode=require"
GOOGLE_CLIENT_ID=your_google_client_id_here
PORT=3000
NODE_ENV=development
CORS_ORIGINS="http://localhost:5173,https://bingo.string.sg"
```

### 5. Set Up Database

```bash
cd backend

# Generate Prisma client
npm run prisma:generate

# Run database migrations
npm run prisma:migrate

# (Optional) Open Prisma Studio to view your database
npm run prisma:studio
```

## Frontend Development

### Running the Development Server

```bash
# From the project root
npm run dev
```

This starts the Vite development server at `http://localhost:5173` with:
- Hot Module Replacement (HMR)
- API proxy to backend at `/api/*`
- Fast builds and updates

### Frontend Structure

```
├── index.html          # Main game page
├── new.html            # Game creation page
├── js/                 # JavaScript modules
│   ├── app.js         # Main app controller
│   ├── bingoGrid.js   # Grid management
│   ├── authManager.js # Authentication
│   └── ...
├── styles/             # CSS stylesheets
└── public/             # Static assets
    ├── challenges.csv # Default challenges
    └── config-*.json  # Environment configs
```

### Key Development Tasks

#### Adding New Features

1. Create or modify JavaScript modules in `js/`
2. Update HTML templates if needed
3. Add CSS styles in `styles/`
4. Test in browser at `http://localhost:5173`

#### Working with the Bingo Grid

The bingo grid is managed by `bingoGrid.js`:
- Grid state is stored in memory and synced to backend
- Cell clicks toggle completion state
- Win detection checks for bingo patterns

#### Mobile Considerations

Based on issue #58, the app includes:
- Touch support for mobile devices
- Pinch-to-zoom functionality
- Scroll and pan capabilities
- **Future Enhancement**: Re-center button to return to original layout after zooming/panning

### Building for Production

```bash
npm run build
```

This creates an optimized production build in the `dist/` directory.

## Backend Development

### Running the Backend Server

```bash
cd backend
npm run dev
```

This starts the Express server at `http://localhost:3000` with:
- Auto-reload on file changes (via tsx watch)
- CORS enabled for frontend origin
- Morgan logging for requests

### Backend Structure

```
backend/
├── src/
│   ├── index.ts          # Main server entry point
│   ├── routes/           # API route handlers
│   │   ├── auth.ts       # Authentication endpoints
│   │   ├── games.ts      # Game CRUD operations
│   │   └── sessions.ts   # Game session management
│   ├── middleware/       # Express middleware
│   │   ├── auth.ts       # JWT validation
│   │   ├── cors.ts       # CORS configuration
│   │   └── errorHandler.ts
│   └── lib/              # Utility functions
│       ├── prisma.ts     # Prisma client
│       └── google.ts     # Google OAuth validation
├── prisma/
│   ├── schema.prisma     # Database schema
│   └── migrations/       # Migration files
└── package.json
```

### API Endpoints

#### Authentication
- `POST /api/auth/login` - Login with Google ID token
- `GET /api/auth/me` - Get current user info
- `POST /api/auth/logout` - Logout (for analytics)

#### Games
- `GET /api/games` - List user's games (authenticated)
- `POST /api/games` - Create new game (authenticated)
- `GET /api/games/:id` - Get specific game
- `PUT /api/games/:id` - Update game (owner only)
- `DELETE /api/games/:id` - Delete game (owner only)
- `GET /api/games/:id/play` - Get game for playing (public)

#### Sessions
- `GET /api/sessions/:gameId` - Get user's session for a game
- `POST /api/sessions` - Create/update game session
- `PUT /api/sessions/:id` - Update session progress

#### Health Check
- `GET /health` - Server health status

### Adding New API Endpoints

1. Create or modify route files in `src/routes/`
2. Add middleware if needed in `src/middleware/`
3. Update type definitions in TypeScript files
4. Test endpoints using curl or Postman
5. Update API documentation in `backend/README.md`

### Building Backend

```bash
cd backend
npm run build
```

This compiles TypeScript to JavaScript in `dist/`.

## Database Management

### Schema Overview

The database includes three main tables:

#### Users
- Stores user information from Google OAuth
- Unique constraint on email and googleId

#### Games
- Stores custom bingo games created by users
- Contains challenges as JSON array (25 items)
- Has public/private visibility flag

#### GameSessions
- Tracks individual game play sessions
- Stores progress as JSON
- Links to game and user

### Common Database Tasks

#### View Database
```bash
cd backend
npm run prisma:studio
```

#### Create a Migration
```bash
cd backend
# Make changes to prisma/schema.prisma first
npm run prisma:migrate
```

#### Reset Database (Development Only)
```bash
cd backend
npx prisma migrate reset
```

#### Seed Database (if seeding is configured)
```bash
cd backend
npx prisma db seed
```

## Environment Configuration

### Environment Files

- **Root `.env`**: Frontend environment variables (prefixed with `VITE_`)
- **`backend/.env`**: Backend environment variables
- **`.env.example`**: Template with all required variables

### Configuration Files

- **`public/config-production.json`**: Production frontend config
- **`public/config-staging.json`**: Staging frontend config
- **`vite.config.js`**: Vite build configuration
- **`vercel.json`**: Vercel deployment configuration

### Switching Environments

For local development:
```env
VITE_API_BASE_URL=http://localhost:3000/api
```

For staging:
```env
VITE_API_BASE_URL=https://staging-api.bingo.string.sg/api
```

For production:
```env
VITE_API_BASE_URL=/api
```

## Development Workflow

### Typical Development Cycle

1. **Start Services**
   ```bash
   # Terminal 1: Frontend
   npm run dev
   
   # Terminal 2: Backend
   cd backend && npm run dev
   ```

2. **Make Changes**
   - Edit code in your preferred editor
   - Changes auto-reload in browser (frontend)
   - Server auto-restarts (backend)

3. **Test Changes**
   - Test in browser at `http://localhost:5173`
   - Verify API calls work correctly
   - Check mobile responsiveness

4. **Commit Changes**
   ```bash
   git add .
   git commit -m "Description of changes"
   git push origin your-branch-name
   ```

### Git Workflow

1. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. Make commits with clear messages:
   ```bash
   git commit -m "Add re-center button for mobile zoom (fixes #58)"
   ```

3. Push and create a pull request:
   ```bash
   git push origin feature/your-feature-name
   ```

### Code Style Guidelines

- **JavaScript**: Use ES6+ features, consistent indentation (2 spaces)
- **TypeScript**: Use strict mode, define types for parameters and return values
- **CSS**: Use BEM naming convention where applicable
- **Commits**: Use conventional commit messages (feat:, fix:, docs:, etc.)

## Building and Deployment

### Frontend Deployment (GitHub Pages)

The frontend is automatically deployed to GitHub Pages when pushing to `main` or `staging` branches.

**Workflow**: `.github/workflows/deploy.yml`
- Builds the project with Vite
- Deploys to GitHub Pages
- Sets up custom domain (bingo.string.sg or staging.bingo.string.sg)

**Manual Deployment**:
```bash
npm run build
npm run deploy
```

### Backend Deployment (Vercel)

The backend API is deployed as Vercel serverless functions.

**Requirements**:
- Set environment variables in Vercel dashboard
- Ensure DATABASE_URL points to production database
- Configure CORS_ORIGINS for production domains

**Files**:
- `vercel.json`: Vercel configuration
- `api/index.ts`: Serverless function entry point

### Environment-Specific Deployments

**Staging**:
- Branch: `staging`
- URL: `https://staging.bingo.string.sg`
- Database: Staging database

**Production**:
- Branch: `main`
- URL: `https://bingo.string.sg`
- Database: Production database

## Testing

### Manual Testing

Since the project doesn't have automated tests currently, focus on manual testing:

1. **Authentication Flow**
   - Test Google login
   - Verify user session persistence
   - Test logout functionality

2. **Game Creation**
   - Create a new custom game
   - Verify challenges are saved correctly
   - Test public/private visibility

3. **Game Playing**
   - Load default game
   - Load custom game
   - Click cells to mark completion
   - Verify bingo detection
   - Test image upload for challenges

4. **Mobile Testing**
   - Test on various device sizes
   - Verify touch interactions
   - Test pinch-to-zoom (issue #58)
   - Verify button layouts (issue #59)

5. **Cross-Browser Testing**
   - Chrome/Edge
   - Firefox
   - Safari
   - Mobile browsers

### Future Testing Improvements

Consider adding:
- Unit tests with Vitest (recommended for Vite projects) or Jest
- Integration tests for API endpoints
- E2E tests with Playwright (already in dependencies)
- Automated visual regression tests

## Contributing Guidelines

### Before Contributing

1. Check existing issues for similar features/bugs
2. Create an issue to discuss significant changes
3. Fork the repository and create a feature branch
4. Follow the code style guidelines

### Pull Request Process

1. **Ensure your code works**:
   - Test locally with both frontend and backend
   - Verify mobile responsiveness
   - Check browser compatibility

2. **Update documentation**:
   - Update this file if adding new features
   - Update README.md if changing setup process
   - Add JSDoc comments to new functions

3. **Create the PR**:
   - Write a clear title and description
   - Reference related issues (e.g., "Fixes #58")
   - Add screenshots for UI changes
   - Request review from maintainers

4. **Address feedback**:
   - Respond to review comments
   - Make requested changes
   - Push updates to the same branch

### Issue References

When working on issues, use these formats:
- `Fixes #58` - Closes the issue when PR is merged
- `Relates to #59` - References without closing
- `Part of #56` - Indicates partial implementation

Example from current issues:
- **Issue #58**: Add re-center button for mobile zoom functionality
- **Issue #59**: Improve button layout (all buttons in same row)
- **Issue #56**: Enhanced analytics and tracking
- **Issue #53**: Better error monitoring and logging

## Common Issues and Troubleshooting

### Frontend Issues

**Problem**: Vite dev server won't start
```bash
# Solution: Clear cache and reinstall
rm -rf node_modules .vite
npm install
npm run dev
```

**Problem**: Google OAuth not working
- Verify `VITE_GOOGLE_CLIENT_ID` is set correctly
- Check authorized origins in Google Cloud Console
- Ensure you're using HTTPS in production

**Problem**: API calls failing
- Verify backend is running on port 3000
- Check `VITE_API_BASE_URL` in `.env`
- Inspect network tab for error details

### Backend Issues

**Problem**: Backend won't start
```bash
# Solution: Check for port conflicts and env vars
lsof -i :3000  # Check if port 3000 is in use
cd backend && cat .env  # Verify env vars are set
```

**Problem**: Database connection failed
- Verify `DATABASE_URL` is correct
- Check database server is running
- Test connection with `npx prisma studio`

**Problem**: Prisma errors
```bash
# Solution: Regenerate Prisma client
cd backend
npm run prisma:generate
```

**Problem**: CORS errors
- Check `CORS_ORIGINS` in backend `.env`
- Verify frontend URL matches allowed origin
- Clear browser cache

### Database Issues

**Problem**: Migrations out of sync
```bash
cd backend
npx prisma migrate reset  # WARNING: Deletes all data
npm run prisma:migrate
```

**Problem**: Prisma Client outdated
```bash
cd backend
npm run prisma:generate
```

### Build Issues

**Problem**: Build fails with TypeScript errors
```bash
cd backend
npm run build  # Check for type errors
# Fix type errors in source files
```

**Problem**: Vite build fails
```bash
# Check for missing dependencies or import errors
npm run build
# Review error output and fix issues
```

### Deployment Issues

**Problem**: Vercel deployment fails
- Check build logs in Vercel dashboard
- Verify all environment variables are set
- Ensure `vercel.json` is properly configured

**Problem**: GitHub Pages shows 404
- Verify GitHub Pages is enabled for the branch
- Check CNAME file is present in `dist/`
- Ensure base path in `vite.config.js` is correct

## Project Structure

```
bingo-string/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions deployment
├── api/
│   └── index.ts                # Vercel serverless API entry
├── backend/
│   ├── src/
│   │   ├── index.ts           # Backend server entry
│   │   ├── routes/            # API routes
│   │   ├── middleware/        # Express middleware
│   │   └── lib/               # Utilities
│   ├── prisma/
│   │   ├── schema.prisma      # Database schema
│   │   └── migrations/        # Database migrations
│   ├── package.json
│   └── tsconfig.json
├── js/
│   ├── app.js                 # Main app logic
│   ├── bingoGrid.js           # Grid management
│   ├── authManager.js         # Authentication
│   ├── challengeLoader.js     # Challenge loading
│   ├── cameraManager.js       # Camera integration
│   ├── modalManager.js        # Modal dialogs
│   ├── touchManager.js        # Touch handling
│   └── game-creation.js       # Game creation UI
├── styles/                     # CSS stylesheets
├── public/                     # Static assets
│   ├── challenges.csv         # Default challenges
│   ├── config-*.json          # Config files
│   └── *.csv                  # Sample challenge files
├── index.html                 # Main game page
├── new.html                   # Game creation page
├── package.json               # Root package config
├── vite.config.js             # Vite configuration
├── vercel.json                # Vercel configuration
├── .env.example               # Environment template
├── .gitignore
├── README.md                  # Project overview
└── development.md             # This file
```

## Additional Resources

### Documentation
- [Vite Documentation](https://vitejs.dev/)
- [Express.js Guide](https://expressjs.com/)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [Google OAuth 2.0](https://developers.google.com/identity/protocols/oauth2)

### Community
- Report bugs and request features via [GitHub Issues](https://github.com/String-sg/bingo-string/issues)
- Contact maintainers: @KOKHC, @physicstjc

### Related Issues
- [Issue #58](https://github.com/String-sg/bingo-string/issues/58): Re-center button for mobile zoom
- [Issue #59](https://github.com/String-sg/bingo-string/issues/59): Button layout improvements
- [Issue #56](https://github.com/String-sg/bingo-string/issues/56): Enhanced analytics
- [Issue #53](https://github.com/String-sg/bingo-string/issues/53): Better monitoring

---

**Happy coding! 🎉**

If you have questions or need help, don't hesitate to open an issue or reach out to the maintainers.
