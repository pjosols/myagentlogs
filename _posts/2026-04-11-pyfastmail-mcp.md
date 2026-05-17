---
layout: registry
title: "pyfastmail-mcp"
date: 2026-04-11
description: MCP server for Fastmail — email, contacts, calendars, file storage via JMAP, CalDAV, and WebDAV.
repo: https://github.com/pjosols/pyfastmail-mcp
install: pip install pyfastmail-mcp
tools: [mail, contacts, calendar, files]
tags: [mcp]
mcp_config:
  command: uvx
  args: [pyfastmail-mcp]
  env:
    - FASTMAIL_API_TOKEN
    - FASTMAIL_EMAIL
    - FASTMAIL_APP_PASSWORD
---

Full Fastmail access over MCP. Mail and contacts use JMAP. Calendar and file storage use CalDAV/WebDAV (require an app password).

## Mail

Send, reply, forward, search, read, pin, archive, manage keywords, masked email, attachments, threads, import/export, identities.

## Contacts

List address books, CRUD contacts, query with filters and sorting.

## Calendar

List calendars, CRUD events. Requires a Fastmail app password with DAV scope.

## Files

List, upload, download, move, delete, create folders. Requires a Fastmail app password with DAV scope.

## Configuration

```json
{
  "mcpServers": {
    "fastmail": {
      "command": "uvx",
      "args": ["pyfastmail-mcp"],
      "env": {
        "FASTMAIL_API_TOKEN": "fmu1-...",
        "FASTMAIL_EMAIL": "you@fastmail.com"
      }
    }
  }
}
```

Add `FASTMAIL_APP_PASSWORD` to enable calendar and file tools.
