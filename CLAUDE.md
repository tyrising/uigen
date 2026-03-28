# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server with Turbopack
npm run build     # Production build
npm run test      # Run Vitest unit tests
npm run lint      # ESLint
npm run setup     # First-time setup: install + Prisma generate + migrate
npm run db:reset  # Reset SQLite database (destructive)
```

Single test: `npx vitest run <test-file-pattern>`

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in natural language; Claude generates them and renders them live in an iframe.

### Data Flow

1. User types in `ChatInterface` → sends message + current file context to `/api/chat`
2. `/api/chat` streams Claude AI responses with tool calls (`str_replace_editor`, `file_manager`)
3. Tool calls update the **in-memory** `VirtualFileSystem` (no disk writes)
4. `FileSystemContext` propagates file changes to the editor and preview
5. Preview panel uses `@babel/standalone` to compile JSX client-side and render in iframe
6. Authenticated users persist projects to SQLite via Prisma server actions

### Key Contexts

- **`FileSystemContext`** (`src/lib/contexts/file-system-context.tsx`) — owns the virtual file system state; all file operations go through here
- **`ChatContext`** (`src/lib/contexts/chat-context.tsx`) — wraps Vercel AI SDK's `useChat`, handles tool call results and routing

### AI Integration

- Model: Claude Haiku 4.5 (default), configured in `src/lib/provider.ts`
- Uses Vercel AI SDK (`ai` + `@ai-sdk/anthropic`) for streaming and tool use
- Tool definitions live in `src/lib/tools/`
- Prompt caching enabled via Anthropic ephemeral cache headers
- Falls back to a mock provider if no `ANTHROPIC_API_KEY` is set in `.env`

### Backend

- Next.js App Router with Server Actions in `src/actions/` for auth and project CRUD
- JWT sessions stored in HTTP-only cookies (`jose` library)
- Prisma + SQLite (`prisma/dev.db`); schema has `User` and `Project` models
- Prisma client generated to `src/generated/prisma/`

### Tech Stack

- React 19 + Next.js 15 (App Router, Server Components)
- Tailwind CSS v4 + shadcn/ui (Radix UI primitives)
- Monaco Editor for code editing
- Vitest + Testing Library for tests (jsdom environment)
- `node-compat.cjs` shim required for Turbopack compatibility (already wired into all npm scripts)
