# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and the AI generates them using Claude with a virtual file system. Components are displayed in a live preview with hot reload.

## Setup and Development Commands

### Initial Setup
```bash
npm run setup
```
Runs `npm install`, generates Prisma client, and runs database migrations.

### Development
```bash
npm run dev        # Start dev server with Turbopack
npm run dev:daemon # Start dev server in background (logs to logs.txt)
```
Development server runs at http://localhost:3000

### Testing and Build
```bash
npm test          # Run Vitest tests
npm run build     # Build production bundle
npm run lint      # Run ESLint
```

### Database Management
```bash
npx prisma generate           # Generate Prisma client
npx prisma migrate dev        # Run migrations
npm run db:reset              # Reset database (destructive)
```

## Architecture

### Virtual File System
The core architecture uses an in-memory virtual file system (`src/lib/file-system.ts`) that manages all generated component files without writing to disk. The `VirtualFileSystem` class provides:
- File and directory CRUD operations
- Path normalization and parent directory auto-creation
- Serialization/deserialization for database persistence
- Text editor operations (view, replace, insert)

### AI-Powered Component Generation
The chat API (`src/app/api/chat/route.ts`) orchestrates component generation:
- Uses Vercel AI SDK with Claude (Anthropic API)
- Provides two AI tools to modify the virtual file system:
  - `str_replace_editor`: Edit file contents via string replacement
  - `file_manager`: Rename/delete/move files and directories
- System prompt (`src/lib/prompts/generation.tsx`) instructs AI to:
  - Create React components styled with Tailwind CSS
  - Use `/App.jsx` as the entry point (required for all projects)
  - Use `@/` import alias for local file imports
  - Operate on virtual FS root (`/`)
- Supports up to 40 agentic steps for complex component generation
- Falls back to mock provider when `ANTHROPIC_API_KEY` is not set (limits to 4 steps)

### Authentication & Authorization
- JWT-based session management (`src/lib/auth.ts`)
- Sessions stored in HTTP-only cookies (7-day expiration)
- Middleware (`src/middleware.ts`) protects `/api/projects` and `/api/filesystem` routes
- Anonymous users can use the app but cannot persist projects

### Data Persistence
- Prisma with SQLite (`prisma/schema.prisma`)
- Database stores:
  - **User**: Email/password authentication
  - **Project**: Component data and chat history (serialized JSON)
- Project data includes:
  - `messages`: Full chat conversation history
  - `data`: Serialized virtual file system state
- Generated Prisma client located at `src/generated/prisma`

### Component Preview System
The preview frame (`src/components/preview/PreviewFrame.tsx`) executes generated components:
- Transpiles JSX using Babel Standalone in the browser
- Transforms `@/` imports to access files from the virtual file system
- Renders components in an isolated iframe for safety
- Hot reloads on file system changes

### State Management
Two primary React contexts:
- `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`): Manages virtual file system state
- `ChatContext` (`src/lib/contexts/chat-context.tsx`): Manages chat messages and AI interactions

## Tech Stack

- **Framework**: Next.js 15 (App Router) with React 19
- **Styling**: Tailwind CSS v4
- **Database**: Prisma with SQLite
- **AI**: Anthropic Claude via Vercel AI SDK
- **Auth**: JWT with jose library
- **Testing**: Vitest with React Testing Library
- **UI Components**: Radix UI primitives

## Environment Variables

Required in `.env`:
```
ANTHROPIC_API_KEY=""  # Optional - app works without it (uses mock responses)
```

## Key File Locations

- Entry point: `src/app/page.tsx` (home/landing page)
- Project view: `src/app/[projectId]/page.tsx` (main editor interface)
- Chat API: `src/app/api/chat/route.ts`
- Virtual FS: `src/lib/file-system.ts`
- AI Tools: `src/lib/tools/` (str-replace.ts, file-manager.ts)
- Database schema: `prisma/schema.prisma`
- System prompt: `src/lib/prompts/generation.tsx`
