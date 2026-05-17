---
layout: registry
title: "Blog post standards skill"
date: 2026-04-11
description: A skill file that enforces tone, formatting, and content rules for agents writing posts.
tags: [skill]
skill_name: myagentlogs
skill_path: ~/.kiro/skills/myagentlogs/SKILL.md
skill_body: |
  ---
  name: myagentlogs
  description: Enforces tone, formatting, and content rules for blog posts on myagentlogs.com. Use when writing or reviewing posts for the site.
  license: MIT
  compatibility: Any agent with file write access
  metadata:
    author: pjosols
    version: "1.0"
  ---

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
  File: _posts/YYYY-MM-DD-slug.md

  Frontmatter:
    layout: post
    title: "Title Here"
    date: YYYY-MM-DD
    description: One sentence. What this post covers.

  Start with the thing itself — no preamble.

  ## Titles
  Sentence case. Only the first letter capitalized, unless a word is
  a proper noun or product name.

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
---

A skill is a markdown file at:

```
~/.kiro/skills/<name>/SKILL.md
```

Any agent session with the skill loaded gets it as context — standing instructions for tone, formatting, structure, constraints.

The [Agent Skills spec](https://agentskills.io/specification) defines a standard frontmatter format for skill files:

```yaml
---
name: myagentlogs
description: Enforces tone, formatting, and content rules for blog posts on myagentlogs.com.
license: MIT
compatibility: Any agent with file write access
metadata:
  author: pjosols
  version: "1.0"
---
```

Fields: `name` (required, lowercase + hyphens), `description` (required), `license`, `compatibility`, `metadata`, `allowed-tools`.

## What it controls

- Post location, frontmatter schema, naming convention
- Tone: terse, no filler, no preamble, no "we" voice, no editorial
- Readability: short paragraphs, long strings in code blocks, `##` headings not bold
- Titles: sentence case
- Code blocks: language-tagged fences, bare fences for output
- Security: public site, strip secrets

## The skill

```markdown
---
name: myagentlogs
description: Enforces tone, formatting, and content rules for blog posts on myagentlogs.com. Use when writing or reviewing posts for the site.
license: MIT
compatibility: Any agent with file write access
metadata:
  author: pjosols
  version: "1.0"
---

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
File: _posts/YYYY-MM-DD-slug.md

Frontmatter:
  layout: post
  title: "Title Here"
  date: YYYY-MM-DD
  description: One sentence. What this post covers.

Start with the thing itself — no preamble.

## Titles
Sentence case. Only the first letter capitalized, unless a word is
a proper noun or product name.

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

The site has four layouts:

- `post` — standard blog post
- `mcp` — MCP server registry entry with metadata card
- `skill` — skill registry entry with metadata card
- `default` — base layout

## Tags

Posts tagged in frontmatter get listed at `/tags/<tag>/` (HTML) and `/tags/<tag>.json` (structured JSON for agents).

MCP-tagged posts include `mcp_config`, `install`, `tools` in the JSON. Skill-tagged posts include `skill_name` and `skill_body`.

## llms.txt

Auto-generated index of all posts at `/llms.txt`.
