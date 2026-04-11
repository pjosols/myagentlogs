---
layout: post
title: "A Skill File for Blog Post Standards"
date: 2026-04-11
description: A skill file that enforces tone, formatting, and content rules for agents writing posts.
tags: [skill]
---

A skill is a markdown file at:

```
~/.kiro/skills/<name>/SKILL.md
```

Any agent session with the skill loaded gets it as context. It's standing instructions — tone, formatting, structure, constraints.

This site has one at:

```
~/.kiro/skills/myagentlogs/SKILL.md
```

## What It Controls

- **Post location**: absolute path to the repo's `_posts/` directory, so agents don't write files to the wrong place
- **Frontmatter schema**: `layout`, `title`, `date`, `description`
- **Tone**: terse, no hedging, no intros, no sign-offs, no "we" voice, no editorial
- **Readability**: short paragraphs, long strings in code blocks not inline, `##` headings not bold
- **Code blocks**: language-tagged fences for code, bare fences for output/errors
- **Security**: public site, strip secrets

## The Skill

```markdown
# myagentlogs — Post Standards

## Site
Personal technical notebook at myagentlogs.com. Public.
Strip anything secret, private, or identifying before publishing.

## Content
No defined scope — write about whatever's worth noting.
Lean toward agents, MCP, skills, tooling.

## Tone
- Terse. Direct. No hedging.
- No intro sentences ("In this post, we'll explore...")
- No sign-off sentences ("That's the point.", "And that's it.")
- No filler, no hype, no personality performance
- No "we" voice. No narrative framing. No opinion sections.
- State what something doesn't do. Don't editorialize about tradeoffs.
- Write like notes to a future self who already knows the context

## Post Format
File: ~/Projects/myagentlogs/_posts/YYYY-MM-DD-slug.md

Frontmatter:
  layout: post
  title: "Title Here"
  date: YYYY-MM-DD
  description: One sentence. What this post covers.

Start with the thing itself — no preamble.

## Readability
- Break up dense paragraphs. Long identifiers, URLs, MIME types
  go in their own code block.
- Never inline long strings. If it wraps, it belongs in a block.
- Use ## headings for sections, not **bold** text.
- Short paragraphs. One idea per paragraph.

## Code Blocks
Always fenced with a language tag. Use bare fences (no language tag)
for output, error messages, or non-code strings.
Keep snippets minimal.

## Length
Whatever it takes, but aim for concise.
If it can be said in 3 sentences, use 3 sentences.
```

## Layouts

The site has three layouts:

- `post` — standard blog post. Date, title, content.
- `mcp` — MCP server registry entry. Adds a metadata card with repo, install command, and tool list. Use with `tags: [mcp]`.
- `default` — base layout. Nav, main, footer.

## Tags

Posts can be tagged in frontmatter:

```yaml
tags: [mcp, skill]
```

Each tag gets a static HTML page and a JSON endpoint:

```
/tags/mcp/       → HTML listing
/tags/mcp.json   → structured JSON for agents
```

The JSON includes all frontmatter fields — title, description, repo, install, tools — so agents can discover and evaluate content without parsing HTML.

## llms.txt

```
/llms.txt
```

Auto-generated index of all posts with title, URL, and description. Agents read this to discover what's on the site.
