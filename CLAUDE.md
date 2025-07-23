# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OTS (One-Time Secret) is a secure secret-sharing web application with:
- **Backend**: Go web server handling API and secret storage
- **Frontend**: Vue 3 + TypeScript SPA with client-side AES-256 encryption
- **Storage**: Supports memory and Redis backends
- **Key Feature**: Secrets self-destruct after first access

## Essential Commands

### Development
```bash
# Run local development server with hot-reloading
tilt up

# Build everything locally
make build-local

# Build only the frontend
make frontend

# Run frontend linter
make frontend_lint

# Build for production
make publish
```

### Testing & Quality
```bash
# Run security scan
make trivy

# Lint TypeScript/Vue code
make frontend_lint
```

### Other Tasks
```bash
# Update translations
make translate

# Build Docker image
make docker

# Clean build artifacts
make clean
```

## Architecture Overview

### Backend Architecture
The Go backend follows a clean architecture pattern:
- **Entry Point**: `main.go` - HTTP server setup, routing, middleware
- **API Handlers**: Defined in `main.go` using Gorilla Mux router
- **Storage Interface**: `pkg/storage/` defines abstract storage with memory and Redis implementations
- **Metrics**: Prometheus metrics exposed via `pkg/metrics/`
- **CLI Tool**: Separate CLI client in `cmd/ots-cli/`

### Frontend Architecture
Vue 3 SPA with TypeScript:
- **Main Components**: 
  - `src/components/Create.vue` - Secret creation form
  - `src/components/View.vue` - Secret viewing/decryption
- **Crypto**: `src/js/crypto.ts` - Client-side AES-256 encryption/decryption
- **Routing**: Vue Router configured in `src/js/router.ts`
- **i18n**: Multi-language support via `src/langs/`
- **State**: Uses Vue 3 Composition API, no Vuex

### API Design
RESTful API with client-side encryption:
1. **Create Secret**: POST `/api/create` - Returns secret ID, key stays client-side
2. **Retrieve Secret**: GET `/api/get/:id` - Returns encrypted payload, destroyed after access
3. **Secret URL Format**: `/#/:locale/secret/:id/:key` - Key fragment never sent to server

### Security Model
- Secrets encrypted client-side with AES-256-GCM
- Encryption key never leaves the browser (URL fragment)
- Server stores only encrypted payload
- Automatic destruction after first access
- Optional password protection adds second encryption layer

## Development Tips

### Frontend Development
- TypeScript strict mode is enabled
- ESLint configured for Vue 3 + TypeScript
- Uses ESBuild for fast builds
- Component styles use scoped CSS

### Backend Development
- Uses Go modules (go.mod)
- Structured logging throughout
- Prometheus metrics for monitoring
- Supports both memory and Redis storage

### Docker Development
- Multi-stage Dockerfile for minimal image size
- docker-compose.yml available for quick setup
- Exposes port 3000 by default