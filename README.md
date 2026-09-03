# Capson AI Studio

An AI workspace that grounds every answer and every draft in **your own knowledge base** instead of generic training data.

Capson ships with a built-in local generation engine (`capson-engine`) that needs **no API keys and no network access**, so the app is fully functional out of the box. Add an optional provider key and all chat replies and studio generations are automatically routed through that hosted model instead — with an automatic fallback to the local engine if the call fails.

![Stack](https://img.shields.io/badge/Next.js-16-black) ![Stack](https://img.shields.io/badge/Drizzle_ORM-PostgreSQL-green) ![Stack](https://img.shields.io/badge/Tailwind_CSS-4-38bdf8)

## Features

**Assistant** — persistent conversations, five switchable personas (Generalist, Strategist, Marketer, Engineer, Coach), markdown rendering with tables, per-message engine badge, and citation footers showing exactly which knowledge entries were used.

**Studio** — ten grounded generation tools:

| Tool | What it produces |
| --- | --- |
| Tagline Forge | Tagline options with the strategic reasoning behind each |
| Value Proposition | Positioning statement, message hierarchy, objection handling |
| Product Copy | Headline variants, features rewritten as benefits, CTA |
| Email Composer | Subject lines to test, body copy, follow-up cadence |
| Content Architect | SEO-aware article outline, section briefs, meta description |
| Strategy Sprint | 90-day plan with phases, milestones table, risks |
| SWOT Analyst | Structured SWOT plus a recommendation and assumptions |
| Social Pack | A week of channel-native posts plus channel notes |
| FAQ Builder | Customer questions grouped by theme with shippable answers |
| Role Brief | Job post, interview loop, and a hiring scorecard |

**Knowledge base** — full CRUD on the facts the engine retrieves, with categories, tags, and search. Everything you add immediately shapes every generation across the app.

**Library** — every studio output is versioned: filter by tool, favorite the keepers, copy the markdown, delete the misses.

## Getting started

```bash
# 1. install dependencies
npm install

# 2. create your env file
cp .env.example .env
#    then edit DATABASE_URL to point at your Postgres instance

# 3. create the tables
npx drizzle-kit push

# 4. run it
npm run dev        # http://localhost:3000
```

Production:

```bash
npm run build
npm run start
```

Five starter knowledge entries are seeded automatically on first use, so the workspace is never empty. Delete them from the Knowledge page and add your own facts.

## Environment variables

| Variable | Required | Purpose |
| --- | --- | --- |
| `DATABASE_URL` | yes | PostgreSQL connection string used by Drizzle ORM |
| `OPENAI_API_KEY` | no | Route generation through OpenAI |
| `OPENAI_MODEL` | no | Defaults to `gpt-4o-mini` |
| `ANTHROPIC_API_KEY` | no | Route generation through Anthropic |
| `ANTHROPIC_MODEL` | no | Defaults to `claude-3-5-sonnet-latest` |
| `GEMINI_API_KEY` | no | Route generation through Google Gemini |

Only `NEXT_PUBLIC_*` variables are exposed to the browser. Provider keys are read exclusively inside server-side route handlers via `process.env`, never shipped to the client.

## How the engine works

`src/lib/ai/engine.ts` contains the local generation engine:

1. **Retrieval** — every request is tokenised (with stop-word filtering) and scored against your knowledge base, weighting title, tag, and content matches. The top matches become grounding facts.
2. **Intent detection** — chat messages are classified (greeting, capability question, "make it shorter" revision, tool-routed request, general question) and handled by a dedicated composer.
3. **Composition** — ten tone-aware templates build structured markdown using the retrieved facts, so output is specific rather than plausible-sounding.
4. **Citation** — the grounding entries are appended to the response so you can verify every claim.

`src/lib/ai/provider.ts` wraps the optional hosted models with a 25-second timeout and returns `null` on any failure, which is what makes the local fallback seamless.

## Project structure

```
src/
├── app/
│   ├── page.tsx                    # Overview dashboard
│   ├── chat/page.tsx               # Assistant
│   ├── studio/page.tsx             # Generation tools
│   ├── knowledge/page.tsx          # Knowledge base CRUD
│   ├── library/page.tsx            # Saved outputs
│   └── api/                        # REST route handlers
│       ├── chat/                   # POST a message, persist the thread
│       ├── conversations/          # List / create / rename / delete threads
│       ├── knowledge/              # Knowledge CRUD
│       ├── generate/               # Run a studio tool
│       ├── generations/            # List / favorite / delete outputs
│       ├── stats/                  # Workspace metrics
│       └── health/                 # Liveness check
├── components/                     # Chat, Studio, Knowledge, Library clients
│   └── Markdown.tsx                # Zero-dependency markdown renderer
├── db/
│   ├── index.ts                    # Drizzle client (pooled)
│   └── schema.ts                   # conversations, messages, knowledge_items, generations
└── lib/
    ├── ai/engine.ts                # Local generation engine
    ├── ai/provider.ts              # Optional hosted-model layer
    ├── ai/tools.ts                 # Tool definitions + tone registry
    └── data.ts                     # Queries, seeding, stats
```

## Notes on deployment

- Set `DATABASE_URL` in your hosting provider's environment variables; do not commit a real production connection string.
- Run `npx drizzle-kit push` (or generate migrations) against the production database once before the first boot.
- `drizzle.config.json` points at a local default (`postgres:postgres@127.0.0.1:5432/app_db`). Change it if your local database differs, or replace it with a `drizzle.config.ts` that reads `process.env.DATABASE_URL`.
- All data pages and route handlers are `force-dynamic`, so no build-time database access is required.

## Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the dev server |
| `npm run build` | Production build |
| `npm run start` | Start the production server |
| `npm run lint` | ESLint |
| `npm run typecheck` | TypeScript, no emit |
