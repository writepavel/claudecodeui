# AGENTS.md - Claude Code UI Development Guide

This document helps agents work effectively in the Claude Code UI repository. It contains commands, patterns, conventions, and gotchas needed for productive development.

## Project Overview

Claude Code UI is a web-based interface for Claude Code CLI, Cursor CLI, and OpenAI Codex. It provides a responsive desktop and mobile UI for managing projects, sessions, and conversations with AI assistants.

**Tech Stack:**
- Frontend: React 18 + Vite + Tailwind CSS
- Backend: Node.js + Express + WebSocket
- Database: SQLite (better-sqlite3)
- Code Editor: CodeMirror 6
- Terminal: xterm.js
- Authentication: JWT + bcrypt

## Essential Commands

### Development
```bash
# Install dependencies
npm install

# Start development server (both frontend and backend)
npm run dev

# Start only backend server
npm run server

# Start only frontend dev server
npm run client

# Build for production
npm run build

# Preview production build
npm run preview

# Start production server
npm start
```

### Release Process
```bash
# Create a new release (builds, tags, publishes to npm, creates GitHub release)
npm run release

# Manual release steps
git checkout main && git pull && npm install
```

### CLI Commands (after global install)
```bash
# Start server
claude-code-ui
# or
cloudcli

# Start with custom port
cloudcli -p 8080

# Check configuration and data locations
cloudcli status

# Update to latest version
cloudcli update

# Show version
cloudcli version
```

## Code Organization & Structure

### Frontend Structure (`src/`)
```
src/
├── components/          # React components
│   ├── ui/             # Reusable UI primitives (button, input, etc.)
│   └── settings/       # Settings panel components
├── contexts/           # React contexts for state management
├── hooks/              # Custom React hooks
├── lib/                # Utility functions
├── utils/              # API and utility modules
└── main.jsx           # Application entry point
```

### Backend Structure (`server/`)
```
server/
├── routes/             # Express route handlers
├── middleware/         # Express middleware
├── database/          # Database setup and initialization
├── utils/             # Server utilities
├── cli.js             # CLI entry point
└── index.js           # Server entry point
```

## Key Patterns & Conventions

### Component Patterns
- Use functional components with hooks
- Import UI components from `components/ui/`
- Follow the established component structure with proper prop destructuring
- Use `className={cn(...)}` for conditional Tailwind classes (from `lib/utils.js`)

### State Management
- Use React Context for global state (see `contexts/`)
- Local state with `useState` for component-specific data
- Server state via WebSocket communication (`WebSocketContext`)

### API Patterns
- Use `authenticatedFetch()` from `utils/api.js` for protected endpoints
- Regular `fetch()` for public endpoints
- Follow the existing endpoint structure in `server/routes/`

### Styling
- Use Tailwind CSS classes
- Dark mode support with `dark:` prefixes
- Responsive design with mobile-first approach
- Use semantic HTML elements

### Error Handling
- Wrap async operations in try-catch blocks
- Use proper HTTP status codes in API responses
- Display user-friendly error messages in the UI
- Check for `.ok` status on fetch responses

## Important Gotchas

### Session Protection System
The app has a sophisticated session protection system that prevents project updates during active conversations. When working with WebSocket updates:

- Sessions are marked as "active" when users send messages
- Project updates are paused for active sessions to prevent UI disruption
- New sessions start with temporary IDs ("new-session-*") before getting real IDs

### WebSocket Communication
- Real-time updates flow through WebSocket messages
- Message types include `projects_updated`, `session_created`, etc.
- Handle WebSocket messages in `useEffect` hooks with proper dependency arrays

### Multi-Provider Support
The app supports three AI providers:
- Claude Code (Anthropic)
- Cursor CLI
- OpenAI Codex

Each provider has different session management and API patterns. Check the provider context when implementing features.

### File System Operations
- File operations go through the backend API for security
- Use the existing file tree and editor components
- Respect project boundaries and permissions

### Database
- Single-user system with SQLite
- User authentication with JWT tokens
- API keys and settings stored in database
- Use transactions for database operations

### Development Environment
- Requires Node.js v20 or higher (see `.nvmrc`)
- Environment variables in `.env` (copy from `.env.example`)
- Default ports: Backend 3001, Frontend 5173

### Mobile vs Desktop
- Responsive design with mobile navigation
- Touch-friendly interfaces
- PWA support with standalone mode detection
- Sidebar behaves differently on mobile (overlay vs fixed)

## Testing

Currently no formal test suite is implemented. Testing is done manually:
1. Start development server with `npm run dev`
2. Test features in browser at `http://localhost:3001`
3. Test mobile responsiveness with browser dev tools
4. Test different AI providers if available

## Security Considerations

- Tools are disabled by default and must be manually enabled
- File access is restricted to project directories
- Authentication required for most operations
- API keys stored securely in database
- CSP headers and proper input validation

## Common Development Tasks

### Adding New Components
1. Create component in appropriate directory (`src/components/`)
2. Use existing UI primitives from `src/components/ui/`
3. Follow naming conventions (PascalCase for files)
4. Import and use in parent components

### Adding New API Endpoints
1. Create route handler in `server/routes/`
2. Add authentication middleware if needed
3. Follow existing error handling patterns
4. Update API client in `src/utils/api.js`

### Adding New Settings
1. Add to relevant settings component in `src/components/settings/`
2. Use localStorage for client-side preferences
3. Use database for persistent settings
4. Add to existing settings panels

### Working with WebSocket
1. Add message types to existing patterns
2. Handle messages in components with `useWebSocketContext`
3. Send messages using `sendMessage` from context
4. Follow existing message structure

## Model Constants

All AI model definitions are centralized in `shared/modelConstants.js`. This file serves as the single source of truth for:
- Supported Claude models (SDK and API formats)
- Cursor model options
- OpenAI Codex models
- Default model selections

Always reference this file when working with model-related features rather than hardcoding model names.

## Deployment

### Production Build
```bash
npm run build    # Builds frontend to dist/
npm start        # Starts production server
```

### Environment Setup
- Copy `.env.example` to `.env` and configure
- Set appropriate `PORT` and `VITE_PORT`
- Configure database path if needed
- Ensure proper file permissions

### Background Service (Optional)
```bash
# Install PM2
npm install -g pm2

# Start as background service
pm2 start claude-code-ui --name "claude-code-ui"

# Enable auto-start
pm2 startup
pm2 save
```

## Release Process

The project uses `release-it` for automated releases:
1. Run `npm run release`
2. Automatically builds project
3. Creates git tag and changelog
4. Publishes to npm
5. Creates GitHub release

Ensure `GITHUB_TOKEN` is set in environment for GitHub integration.