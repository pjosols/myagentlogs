---
layout: post
title: "A Kiro Skill for Blog Post Standards"
date: 2026-04-11
description: A skill file that enforces tone, formatting, and content rules across agent sessions.
---

Kiro skills are markdown files that live at:

```
~/.kiro/skills/<name>/SKILL.md
```

Any agent session with the skill loaded gets it as context. Standing instructions, not a prompt template.

This site has one. It controls post format, tone, and readability rules.

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
- No filler, no hype, no personality performance
- Write like notes to a future self who already knows the context

## Post Format
File: _posts/YYYY-MM-DD-slug.md

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

## What It Covers

- Post location, frontmatter schema, naming convention
- Tone: terse, no filler, no preamble
- Readability: short paragraphs, long strings in code blocks, `##` headings not bold
- Code blocks: language-tagged fences, bare fences for output
- Security: public site, strip secrets

The skill lives in `~/.kiro/skills/`, not in the repo. It applies across sessions regardless of working directory.
