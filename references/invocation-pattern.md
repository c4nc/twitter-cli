---
name: invocation-pattern
description: Exact terminal invocation patterns for twitter-cli operations
---

# twitter-cli Invocation Patterns

twitter-cli is a **terminal binary** (`twitter`), not a Hermes native tool.
There is no `twitter_*` tool — always invoke through the `terminal` tool.

## Standard Workflow

```bash
# 1. Auth check (MUST run first)
twitter status --yaml >/dev/null && echo AUTH_OK || echo AUTH_NEEDED

# 2. Fetch content
twitter article <parent-tweet-id>           # Full article via the posting tweet
twitter tweet <id>                          # Tweet + replies
twitter search <query> --json               # Search results
twitter user-posts <handle> --max 20 --json # User's tweets

# 3. Write operations
twitter post "text"
twitter reply <id> "text"
```

## Verified Session: YanXbt 15-Levels Article (2026-09)

**Tweet ID:** 2068629714776756339
**URL:** https://x.com/i/status/2068629714776756339
**Author:** YanXbt (@IBuzovskyi)
**Article Title:** "15 LEVELS OF HERMES AGENT. FROM CHATBOT TO 24/7 AUTONOMOUS SYSTEM."

```bash
# Auth check
which twitter && twitter --help 2>&1 | head -30
# -> Binary at ~/.local/bin/twitter, all commands listed

# Article fetch (note: a STATUS URL — i.e. the posting tweet — not /i/article/)
twitter article https://x.com/i/status/2068629714776756339 2>&1
# -> ok: true, schema_version: 1, ~12000 chars, no truncation
# -> metrics: 473 likes, 1191 bookmarks, 341.6K views
```

Ingestion results:
- Vault note: `HermesConfig/SKILLS/hermes-docs-and-best-practices.md`
- Installed skill: `~/.hermes/skills/devops/hermes-docs-and-best-practices/SKILL.md`

## Common Failures

| Mistake | Fix |
|---------|-----|
| Treating `twitter` as a Hermes native tool | Wrap in the `terminal` tool: `terminal("twitter article <id>")` |
| Passing an `/i/article/<id>` URL to `twitter article` | Use the POSTING TWEET's id (article content rides in it) — see SKILL.md "Article" section |
| Missing `--full-text` for article body | `twitter article <id>` returns the full article text by default |
| Not checking auth first | Run `twitter status --yaml >/dev/null` before content fetch |
| search/followers/likes 404 | Stale PyPI 0.8.5 — reinstall from the `c4nc` fork |
| Rate limit hit | Space queries 5–10s apart; back off to 30–60s after a 226 |
