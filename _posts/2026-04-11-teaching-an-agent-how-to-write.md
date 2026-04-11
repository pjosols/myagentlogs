---
layout: post
title: "Teaching an Agent How to Write for Your Blog"
date: 2026-04-11
description: Using a Kiro skill file to enforce tone, formatting, and content standards across agent sessions.
---

An agent wrote the first post on this site. It was fine — technically correct, readable. But the formatting was off. Bold text instead of headings. Long MIME types jammed inline. Wall-of-text paragraphs.

The fix isn't editing every post. It's giving the agent a skill.

## What's a Skill

A skill in Kiro is a markdown file at:

```
~/.kiro/skills/<name>/SKILL.md
```

Any agent session with that skill loaded gets the file as context. It's not a prompt template — it's standing instructions. Think of it as a style guide the agent actually reads.

## The Skill

Here's the full skill file for this site:

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

## What It Fixed

The first post had three problems the skill now prevents:

1. Section headers were `**bold text**` instead of `##` headings. The CSS styles `h2` as small uppercase with a rule — bold text just looks like a slightly heavier paragraph.

2. Long MIME types were inline, creating unreadable walls of text. The skill now says: if it wraps, it belongs in a block.

3. No guidance on tone. Without the skill, the agent defaults to explanatory tech-blog voice. The skill sets the expectation: terse, no preamble, no filler.

## Why Not a README

A README in the repo would work too. But the skill lives in `~/.kiro/skills/`, which means it applies across sessions regardless of which directory the agent is working in. The agent doesn't need to be in the repo to know the standards.

It also keeps the repo clean. The skill is authoring guidance, not project documentation.
