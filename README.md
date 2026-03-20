# UIGen

AI-powered React component generator with live preview. Describe a component in plain English, watch Claude generate it in real time, and iterate via chat.

![Next.js](https://img.shields.io/badge/Next.js-15-black) ![React](https://img.shields.io/badge/React-19-61DAFB) ![TypeScript](https://img.shields.io/badge/TypeScript-5-blue) ![Tailwind CSS](https://img.shields.io/badge/Tailwind-v4-38BDF8)

## Features

- **AI code generation** — Claude writes React + Tailwind components based on your description
- **Live preview** — Components render instantly in an iframe using Babel standalone
- **Virtual file system** — All files exist in memory; nothing written to disk
- **Code editor** — Monaco editor with file tree for viewing and editing generated code
- **Project persistence** — Authenticated users can save and resume projects
- **Works without an API key** — Falls back to a mock provider returning static examples

## Prerequisites

- Node.js 18+
- npm
- Anthropic API key (optional)

## Setup

```bash
# 1. Clone the repo
git clone https://github.com/sudippalit/uigen.git
cd uigen

# 2. (Optional) Add your Anthropic API key
echo 'ANTHROPIC_API_KEY=your-api-key-here' > .env

# 3. Install dependencies, generate Prisma client, run migrations
npm run setup

# 4. Start the dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## Usage

1. Sign up for an account or continue as an anonymous user
2. Type a description of the React component you want (e.g. "a dark mode pricing table with three tiers")
3. Watch the component generate in the live preview panel
4. Switch to **Code** view to inspect or edit the generated files in Monaco
5. Keep chatting to refine and iterate

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15 (App Router) |
| UI | React 19, Tailwind CSS v4, shadcn/ui |
| AI | Anthropic Claude (`claude-haiku-4-5`), Vercel AI SDK |
| Editor | Monaco Editor |
| Database | Prisma + SQLite |
| Auth | JWT (jose) + bcrypt |
| Testing | Vitest + Testing Library |

## Scripts

```bash
npm run dev       # Start dev server with Turbopack
npm run build     # Production build
npm test          # Run tests
npm run db:reset  # Reset the database (destructive)
```
