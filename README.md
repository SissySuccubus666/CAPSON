# 🜏 The Digital Grimoire

A modular, **local-first, Linux-first** platform for exploring the history, literature, symbolism, mythology,
philosophy and practice of occult and esoteric traditions — and for building your own grimoires on top of that
archive.

The AI component is a **research guide and librarian, never an unquestionable authority**. Every claim in the system
carries a source-integrity label.

---

## What is implemented here

This deployment is the full Digital Grimoire stack as a single self-contained web application:

| Spec module | Implementation |
| --- | --- |
| **The Archive (v0.1)** | `src/db/seed-corpus.ts` — 24 public-domain documents with attribution, date, language, region, translator, edition, rights and integrity label, each with retrievable passages |
| **The Librarian (v0.2)** | `/library`, `/library/[slug]`, `/search`, notes, faceted filters, `/api/documents`, `/api/search`, `/api/notes` |
| **The Oracle (v0.3)** | `/oracle` + `src/lib/retrieval.ts` (BM25-style lexical RAG) + `src/lib/ai.ts` (Ollama, with an honest retrieval-only fallback), `ai_sessions` audit log |
| **The Web (v0.4)** | `/graph` force-directed knowledge graph, `/entities`, `/symbols` correspondence tables, `/timeline` |
| **The Workshop (v0.5)** | `/workshop` custom grimoire builder, `.grim` export/import/validate (`src/lib/grim.ts`), user notes |
| **v1.0** | all three — Archive + AI + Graph — feeding the user's own codex |

Also included: a working terminal client, `bin/grimoire.mjs`.

## Source integrity

```
PRIMARY SOURCE · SECONDARY SOURCE · SCHOLARLY INTERPRETATION
MODERN OCCULT INTERPRETATION · USER CREATED · FICTIONAL · UNCERTAIN
```

Each document, passage, entity, symbol, correspondence, graph edge, note and grimoire entry carries one of these
labels, and each entity keeps **three separate interpretation fields** — historical, later and modern — plus separate
primary- and secondary-source lists and an explicit confidence rating (`HIGH` / `MEDIUM` / `LOW`).

Low confidence is never hidden. Where an attribution exists mainly in later occult literature, the record says so.

## Local AI guide

```
question → database retrieval → relevant passages → context → local model → answer → source references
```

* Point at your own machine with `OLLAMA_URL` (default `http://127.0.0.1:11434`) and `OLLAMA_MODEL`.
* The prompt forbids invented manuscripts, dates, shelfmarks and quotations, requires inline `[n]` citations, and
  requires a final `CONFIDENCE:` line.
* If no model is reachable, the guide degrades into a **retrieval report**: the passages found, their labels, and an
  explicit statement that nothing was synthesised. It never silently fabricates.

Personas: **The Archivist**, **The Philologist**, **The Comparative Scholar**, **The Symbolist**, **The Librarian**.

## The `.grim` format

YAML with three top-level keys:

```yaml
grimoire: { name: …, author: …, tradition: …, description: … }
entries:  [ { type: entity|symbol|correspondence|text|note|illustration|reference, name: …, body: …, label: …, meta: … } ]
sources:  [ { title: …, author: …, date: …, label: … } ]
```

`POST /api/grim` validates or imports; `GET /api/grimoires/:slug/export` downloads; `GET /api/grim` returns a blank
template. Validation warns when user-authored material claims to be `PRIMARY SOURCE`.

## HTTP API

```
GET  /api/health
GET  /api/search?q=Hermes&tradition=hermeticism&label=PRIMARY SOURCE
GET  /api/documents[?slug=picatrix]
GET  /api/entities[?slug=paimon]
GET  /api/correspondences[?domain=planet&key=Saturn]
GET  /api/graph[?focus=entity:lilith]
GET  /api/timeline
GET  /api/ask                      → Ollama reachability
POST /api/ask                      → { question, persona, tradition? }
GET|POST /api/notes
GET|POST /api/grimoires
GET|PATCH|DELETE /api/grimoires/:slug
POST|DELETE /api/grimoires/:slug/entries
GET  /api/grimoires/:slug/export   → .grim download
POST /api/grim                     → validate | import
GET  /api/grim                     → template
```

## Terminal client

```bash
./bin/grimoire.mjs status
./bin/grimoire.mjs search hermeticism
./bin/grimoire.mjs entity Hermes
./bin/grimoire.mjs source "Corpus Hermeticum"
./bin/grimoire.mjs correspondences planet Saturn
./bin/grimoire.mjs timeline
./bin/grimoire.mjs graph Hermeticism
./bin/grimoire.mjs ask "What is the historical relationship between Hermeticism and alchemy?" --persona archivist
./bin/grimoire.mjs grim validate the-black-archive.grim
```

## Data model

`traditions · authors · sources · documents · passages · entities · symbols · correspondences · relations ·
timeline_events · notes · custom_grimoires · grimoire_entries · personas · ai_sessions · meta`

Defined in `src/db/schema.ts` (Drizzle ORM). The corpus seeds itself lazily and idempotently on first request, guarded
by a Postgres advisory lock.

## Linux-first notes

The native build targets `~/.local/share/digital-grimoire/`, `~/.config/digital-grimoire/` and
`~/.cache/digital-grimoire/` on Mint, Ubuntu, Debian, Fedora and Arch, with SQLite and Ollama. This hosted build runs
the identical schema and API against PostgreSQL; nothing in the interface depends on a network service, and the
retrieval engine, graph layout and `.grim` handling all run locally.

## Design principles

1. Linux-first · 2. Open-source friendly · 3. Local-first · 4. Modular · 5. Source-aware · 6. Historically
responsible · 7. AI-assisted rather than AI-authoritative · 8. Extensible · 9. User-controlled · 10. Offline-capable
