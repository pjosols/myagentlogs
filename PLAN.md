# my-agent-blog — Project Plan

A website about AI agents, MCP servers, skills, and instructions.
Readable by humans (HTML+CSS) and AI agents (markdown/JSON) from the same content source.

## Goals

- Author content in markdown with YAML frontmatter
- Serve styled HTML to browsers, raw markdown or JSON to agents
- Track analytics server-side (no client-side JS required)
- Clean, fast, minimal dependencies

## Tech Stack

- **Framework**: Litestar (ASGI, native content negotiation)
- **Templating**: Jinja2
- **Markdown**: python-markdown + frontmatter parsing
- **Analytics storage**: SQLite
- **Deployment**: TBD (Docker, fly.io, VPS, etc.)

## Content Structure

```
content/
├── index.md                    # homepage
├── agents/
│   ├── index.md                # agents overview
│   └── mcp-servers.md
├── skills/
│   └── index.md
└── instructions/
    └── index.md
```

Each markdown file has YAML frontmatter:

```yaml
---
title: MCP Servers
description: How to build and configure MCP servers
tags: [mcp, servers, agents]
updated: 2026-03-22
---
```

## Content Negotiation

Three output formats from every route:

| Accept header / suffix | Response |
|---|---|
| `text/html` (default) | Rendered HTML page with CSS |
| `text/markdown` or `.md` suffix | Raw markdown with frontmatter |
| `application/json` or `.json` suffix | Structured JSON (see below) |

Resolution order:
1. URL suffix (`.md`, `.json`)
2. `Accept` header
3. Default to HTML

## JSON Schema for Agents

```json
{
  "title": "MCP Servers",
  "description": "How to build and configure MCP servers",
  "tags": ["mcp", "servers", "agents"],
  "updated": "2026-03-22",
  "content": "# MCP Servers\n\nRaw markdown here...",
  "links": ["/agents", "/skills/mcp-integration"]
}
```

## Machine-Readable Discovery

- `/llms.txt` — plain text site summary following the llms.txt convention
- `/sitemap.json` — full page index with metadata for agent crawling
- `/sitemap.xml` — standard sitemap for search engines

## App Structure

```
my-agent-blog/
├── content/                # markdown source files
├── templates/
│   ├── base.html           # layout, nav, CSS
│   └── page.html           # single page template
├── static/                 # CSS, favicon, etc.
├── app.py                  # Litestar app, routes, middleware
├── content_loader.py       # markdown parsing, frontmatter extraction
├── analytics.py            # request logging to SQLite
├── requirements.txt
├── Dockerfile              # optional
└── PLAN.md
```

## Server-Side Analytics

Middleware logs every request to SQLite:

- Timestamp
- Path
- User-Agent (raw + classified: browser/bot/agent)
- IP (hashed)
- Accept header
- Response format served (html/md/json)

No client-side JS. No cookies. No third-party services.

## Implementation Phases

### Phase 1 — Core
- [ ] Litestar app with catch-all route
- [ ] Markdown loader with frontmatter parsing
- [ ] Content negotiation (HTML / markdown / JSON)
- [ ] Base HTML template with minimal CSS
- [ ] A few sample content pages

### Phase 2 — Discovery & Analytics
- [ ] Request logging middleware + SQLite
- [ ] `/llms.txt` endpoint
- [ ] `/sitemap.json` endpoint
- [ ] `/sitemap.xml` endpoint

### Phase 3 — Polish
- [ ] Navigation (auto-generated from content tree)
- [ ] Tag pages
- [ ] RSS feed
- [ ] Analytics dashboard (simple HTML page, server-rendered)
- [ ] Deployment setup

## Expanded Vision: Living Pattern Library

The site could be more than documentation — it could be a self-improving pattern registry:

- **Patterns** — reusable guides (e.g. "how to build an MCP server") that agents consume as skills
- **Experiments** — structured records of agents applying patterns to real projects, with metrics
- **Proposals** — agent-submitted pattern improvements based on experiment results, merged after human review
- **Feedback loop** — patterns evolve based on empirical evidence from agent builds

### Git-Native Approach

The web app may be unnecessary for the core loop. Git gives us versioning, branching, PRs for review, auth, and CI for free. The repo *is* the platform:

- Agents clone the repo to read patterns
- Experiments and proposals come in as PRs
- GitHub Actions validates frontmatter/structure
- The Litestar app becomes an optional read-only presentation layer (HTML for humans, JSON for agents who prefer HTTP over git)
- GitHub Pages could handle the human-readable side with zero infrastructure

This significantly reduces what we need to build. The MVP might just be content structure + seed patterns + a submission script.

## Open Questions

- **Git-native vs app-first?** — could start git-native and add the web layer later if needed
- Cache strategy? (markdown files rarely change — could cache rendered HTML in memory)
- Search? (full-text search over content, maybe SQLite FTS5)
- Do agents need pagination for listing endpoints?
- Custom CSS theme or use something like Simple.css / Pico CSS?
- Pattern versioning — do experiments reference the pattern version they used?
- Auto-merge thresholds — experiments auto-merge, pattern changes require human review?
- Cross-pattern insights — aggregate learnings across patterns?
