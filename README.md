# UIGen

AI-powered React component generator with live preview.

## Prerequisites

- Node.js 18+
- npm

## Setup

1. **Optional** Edit `.env` and add your Anthropic API key:

```
ANTHROPIC_API_KEY=your-api-key-here
```

The project will run without an API key. Rather than using a LLM to generate components, static code will be returned instead.

2. Install dependencies and initialize database

```bash
npm run setup
```

This command will:

- Install all dependencies
- Generate Prisma client
- Run database migrations

## Running the Application

### Development

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

## Usage

1. Sign up or continue as anonymous user
2. Describe the React component you want to create in the chat
3. View generated components in real-time preview
4. Switch to Code view to see and edit the generated files
5. Continue iterating with the AI to refine your components

## Features

- AI-powered component generation using Claude
- Live preview with hot reload
- Virtual file system (no files written to disk)
- Syntax highlighting and code editor
- Component persistence for registered users
- Export generated code

## Architecture

UIGen is built on a modular architecture with several key components working together:

### Virtual File System

The core of UIGen uses an in-memory virtual file system (`src/lib/file-system.ts`) that manages all generated component files without writing to disk. The `VirtualFileSystem` class provides:

- File and directory CRUD operations
- Path normalization and parent directory auto-creation
- Serialization/deserialization for database persistence
- Text editor operations (view, replace, insert)

This approach allows for fast component generation and preview without cluttering the user's file system.

### AI-Powered Component Generation

The chat API (`src/app/api/chat/route.ts`) orchestrates component generation using Claude AI:

- Leverages Vercel AI SDK with Anthropic's Claude
- Provides two AI tools to modify the virtual file system:
  - `str_replace_editor`: Edit file contents via string replacement
  - `file_manager`: Rename/delete/move files and directories
- System prompt (`src/lib/prompts/generation.tsx`) instructs the AI to:
  - Create React components styled with Tailwind CSS
  - Use `/App.jsx` as the entry point for all projects
  - Use `@/` import alias for local file imports
  - Operate on virtual file system root (`/`)
- Supports up to 40 agentic steps for complex component generation
- Falls back to mock provider when `ANTHROPIC_API_KEY` is not set

### Component Preview System

The preview frame (`src/components/preview/PreviewFrame.tsx`) executes generated components in real-time:

- Transpiles JSX using Babel Standalone in the browser
- Transforms `@/` imports to access files from the virtual file system
- Renders components in an isolated iframe for safety
- Provides hot reload on file system changes

### Authentication & Authorization

Security and user management:

- JWT-based session management (`src/lib/auth.ts`)
- Sessions stored in HTTP-only cookies with 7-day expiration
- Middleware (`src/middleware.ts`) protects API routes
- Anonymous users can use the app but cannot persist projects

### Data Persistence

Database layer using Prisma with SQLite:

- **User**: Email/password authentication
- **Project**: Component data and chat history (serialized JSON)
- Project data includes:
  - `messages`: Full chat conversation history
  - `data`: Serialized virtual file system state
- Generated Prisma client located at `src/generated/prisma`

### State Management

Two primary React contexts coordinate application state:

- `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`): Manages virtual file system state
- `ChatContext` (`src/lib/contexts/chat-context.tsx`): Manages chat messages and AI interactions

## Tech Stack

- Next.js 15 with App Router
- React 19
- TypeScript
- Tailwind CSS v4
- Prisma with SQLite
- Anthropic Claude AI
- Vercel AI SDK
- Radix UI primitives
- Vitest with React Testing Library
