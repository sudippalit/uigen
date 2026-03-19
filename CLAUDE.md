# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run setup        # First-time setup: install deps, generate Prisma client, run migrations
npm run dev          # Start dev server with Turbopack (http://localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
npm test             # Run all tests with Vitest
npx vitest run src/components/chat/__tests__/  # Run a specific test directory
npx prisma migrate dev   # Apply new migrations
npm run db:reset         # Reset database (destructive)
```

Set `ANTHROPIC_API_KEY` in `.env` to use real Claude AI. Without it, the app falls back to a mock provider that returns static components.

## Architecture

UIGen is a Next.js 15 App Router application where users chat with Claude to generate React components. The AI writes code into an in-memory virtual file system, which is rendered live in an iframe preview and editable in a Monaco code editor.

### Core Data Flow

1. User sends a message via `ChatInterface` → `ChatContext` (`useChat` from Vercel AI SDK)
2. `POST /api/chat` streams a response from Claude with the current virtual FS as context
3. Claude calls tools (`str_replace_editor`, `file_manager`) to create/edit virtual files
4. File changes update `FileSystemContext` → Monaco editor re-renders, iframe preview reloads
5. For authenticated users, the chat history and file system are persisted to SQLite via Prisma server actions

### Virtual File System (`src/lib/file-system.ts`)

All "files" are in-memory JavaScript objects — nothing is written to disk. `VirtualFileSystem` supports create/read/update/delete/rename and serializes to JSON for database storage. The `str_replace_editor` tool (modeled after Claude's text editor tool) lets the AI make targeted edits via line-based `str_replace` and `insert` commands.

### AI Integration (`src/app/api/chat/route.ts`, `src/lib/provider.ts`)

- Model: `claude-haiku-4-5` via `@ai-sdk/anthropic` + Vercel AI SDK streaming
- Two tools: `str_replace_editor` (create/view/edit files) and `file_manager` (rename/delete)
- System prompt at `src/lib/prompts/generation.tsx` instructs Claude to generate React + Tailwind components
- Max 40 tool-use steps per request; prompt caching enabled
- `src/lib/provider.ts` exports a single `getModel()` that returns either the real Anthropic model or a `MockLanguageModel`

### Authentication (`src/lib/auth.ts`, `src/middleware.ts`)

JWT sessions via `jose`, stored in httpOnly cookies (7-day expiry). Bcrypt for password hashing. Server actions in `src/actions/` handle sign-up/sign-in/sign-out/getUser. Middleware protects `/api/projects` and `/api/filesystem` routes.

### State Management

- `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`) — owns the `VirtualFileSystem` instance and exposes file operations
- `ChatContext` (`src/lib/contexts/chat-context.tsx`) — wraps Vercel AI SDK's `useChat`, passes serialized FS state to the API on each message
- Anonymous sessions: work tracked in `sessionStorage` via `src/lib/anon-work-tracker.ts`
- Authenticated sessions: projects persisted in Prisma (`Project.messages` = JSON chat history, `Project.data` = JSON file system)

### UI Layout (`src/app/main-content.tsx`)

Resizable panels via `react-resizable-panels`: chat (35% left) | preview+code (65% right). The right panel has Preview/Code tabs; Code tab splits into file tree (30%) and Monaco editor (70%). Preview uses an iframe with Babel standalone to transpile and render JSX client-side.

### Path Alias

`@/*` maps to `src/*` — use this for all imports.

### Database

SQLite via Prisma. Schema: `User` (email, bcrypt password) → has many `Project` (name, messages JSON, data JSON). Prisma client is generated into `src/generated/prisma`.
