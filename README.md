# twitter-cli

> **Maintained fork** of [public-clis/twitter-cli](https://github.com/public-clis/twitter-cli) (originally [jackwener/twitter-cli](https://github.com/jackwener/twitter-cli)) by [c4nc](https://github.com/c4nc/twitter-cli). Read [About this fork](#about-this-fork).

[![Fork of public-clis/twitter-cli](https://img.shields.io/badge/fork%20of-public--clis%2Ftwitter--cli-blue)](https://github.com/public-clis/twitter-cli)
[![CI](https://github.com/c4nc/twitter-cli/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/c4nc/twitter-cli/actions/workflows/ci.yml)
[![Version](https://img.shields.io/badge/version-0.8.6-brightgreen)](https://github.com/c4nc/twitter-cli/releases)
[![PyPI (upstream, stale)](https://badge.fury.io/py/twitter-cli.svg)](https://pypi.org/project/twitter-cli/)
[![Python](https://img.shields.io/badge/python-%3E%3D3.10-blue.svg)](https://pypi.org/project/twitter-cli/)

A terminal-first CLI for Twitter/X: read timelines, bookmarks, search, and long-form articles — and post, reply, quote, like, retweet, and follow — without API keys. Cookies in, structured data out; built for humans *and* AI agents.

## About this fork

This is a **maintenance fork** of the original `twitter-cli`. It is a **minimal-diff fork, not a rewrite** — the whole feature set (auth, filters, output formats, articles, lists, write operations) is the original project's. This fork exists to keep the tool *working* after X's September 2026 web rebuild broke a core anti-detection header, and to keep shipping fixes while the original repository has been dormant.

### Why this fork exists

X restructured its web frontend in September 2026. The logged-out root page (`https://x.com`) no longer embeds the `ondemand.s` bundle that the `xclienttransaction` library scrapes to generate the `x-client-transaction-id` header. Without that header, every header-gated GraphQL read returns HTTP 404:

- `search`
- `followers`, `following`, `likes`

The original repository has had **no commits since May 2026** (its `main` is at an unreleased 0.8.6; the last *released* version on PyPI is **0.8.5**), and the several community pull requests that fixed this (#74, #79, #84, #86, #89, #90, #91) sat unmerged against an unresponsive maintainer. This fork ships the verified fix so the tool works out of the box.

### What this fork contains (beyond upstream `main`)

1. **ClientTransaction bootstrap fixed** — the bootstrap now fetches `https://x.com/home` *with the session cookie* (the page that still carries the `ondemand.s` bundle), so `x-client-transaction-id` is generated again. This restores `search`, `followers`, `following`, and `likes`. Supersedes the unmerged upstream PRs listed above.
2. **`xclienttransaction >= 1.0.3`** — the pinned 1.0.1 cannot parse the new numeric-module page layout; 1.0.3 (released 2026-06) can. Bumped and locked.
3. **GraphQL bundle scan updated** — matches the new `abs.twimg.com/x-web/x-web/*` asset layout (and its `./assets/*.js` chunks) so the live queryId re-resolution fallback works again.
4. **Five community fixes cherry-picked** (see credits below): author follower/following counts (#57), `possibly_sensitive` + media warning labels (#87), reject incomplete `TweetDetail` responses (#75), retry `Query: Unspecified` via the live queryId resolver (#77), and custom Chromium cookie directories (#76).
5. **`~/.twitter-cli` cache files locked to `0600`** — the ClientTransaction cache stores the full `x.com/home` HTML (which embeds the live `auth_token`/`ct0`); it was written world-readable (`0644`). Now written and read owner-only, with regression tests.
6. **Article handling clarified** — X articles are *not* paywalled, but the API has no article-by-id operation, so an `/i/article/<id>` URL cannot be fetched directly. The command now emits a precise error pointing at the working path: the **tweet that posted the article** carries the full article (`articleTitle` + `articleText`). See [Usage — Article](#article).

### Credits

This fork builds on the work of several people and we credit them explicitly:

- **Original project:** [twitter-cli](https://github.com/public-clis/twitter-cli) by **jackwener** ([jakevingoo@gmail.com](mailto:jakevingoo@gmail.com)), [Apache-2.0](LICENSE). Everything in this fork that is not listed as a change above is the original author's work, unmodified.
- **`xclienttransaction`** ([iSarabjitDhiman/XClientTransaction](https://github.com/iSarabjitDhiman/XClientTransaction)) by **Sarabjit Dhiman**, MIT-licensed — the `x-client-transaction-id` generator this tool depends on.
- **Cherry-picked community fixes:**
  - **Lucius Chen** ([@LuciusChen](https://github.com/LuciusChen)) — #75 (reject incomplete TweetDetail), #77 (retry `Unspecified` with live IDs), #76 (custom Chromium cookie directories)
  - **edwin bernadus** ([@edwinbernadus](https://github.com/edwinbernadus)) — #57 (author follower/following counts)
  - **Li Chenxi** ([@ayanamists](https://github.com/ayanamists)) — #87 (`possibly_sensitive` + media warning labels)
- **This fork (c4nc):** the ClientTransaction bootstrap fix ([#93](https://github.com/public-clis/twitter-cli/pull/93)), the `xclienttransaction >= 1.0.3` bump, the x-web bundle-scan update, the cache-permission fix, and the test coverage for all of the above.

The fix in this fork is also contributed upstream as [public-clis/twitter-cli#93](https://github.com/public-clis/twitter-cli/pull/93).

### Upstream & contributing

We are **not** forking to fork. The priority is a working tool and a clean upstream. If the original repository resumes active maintenance — in particular, if it merges the ClientTransaction fix and ships a release — this fork will be archived (or merged back), and we'll direct people to the original. Until then, **this fork is the actively-maintained version**, and contributions are welcome: open an issue or PR here, or against the original, whichever you prefer. If X changes its page layout again and both repositories break, this is where the next fix lands.

### How this README stays current

**Rule:** every commit that adds, changes, or fixes *user-facing* behavior updates this README in the same commit — the feature list, the usage examples, the "What this fork contains" list, and (when the version bumps) the version badge. The `SKILL.md` shipped in this repository follows the same rule: it is the single source of truth for AI-agent instructions, and it is updated alongside every functionality change (see [Use as AI Agent Skill](#use-as-ai-agent-skill)).

---

## Features

**Read:**
- Timeline: fetch `for-you` and `following` feeds, with cursor pagination
- Bookmarks: list saved tweets from your account
- Search: find tweets by keyword with Top/Latest/Photos/Videos tabs
- Tweet detail: view a tweet and its replies; use `show <N>` to open tweet #N from the last list output
- Article: fetch a Twitter/X long-form article (full text, no truncation) and export it as Markdown — via the tweet that posted it
- List timeline: fetch tweets from a Twitter List
- User lookup: profile, tweets, likes, followers, and following (with author follower/following counts)
- `--full-text`: disable tweet text truncation in rich table output
- Structured output: export any data as YAML or JSON for scripting and AI-agent integration
- Optional scoring filter: rank tweets by engagement weights
- Structured output contract: [SCHEMA.md](./SCHEMA.md)

> **AI Agent Tip:** Prefer `--yaml` for structured output unless a strict JSON parser is required. Non-TTY stdout defaults to YAML automatically. Use `--max` to limit results.

**Write:**
- Post: create new tweets and replies, with optional image attachments (up to 4)
- Quote: quote-tweet with optional images
- Delete: remove your own tweets
- Like / Unlike: manage tweet likes
- Retweet / Unretweet: manage retweets
- Bookmark: bookmark/unbookmark (`favorite`/`unfavorite` kept as compatibility aliases)
- Write commands also support explicit `--json` / `--yaml` output

**Auth & Anti-Detection:**
- Cookie auth: browser cookies (auto-extracted) or environment variables
- Full cookie forwarding: extracts ALL browser cookies for richer browser context
- TLS fingerprint impersonation: `curl_cffi` with dynamic Chrome version matching
- `x-client-transaction-id` header generation (the Sept-2026 fix, see above)
- Request timing jitter to avoid pattern detection
- Write operation delays (1.5–4s random) to mitigate rate limits
- Proxy support via `TWITTER_PROXY` environment variable

## Installation

**The working version is this fork** — install it:

```bash
# Recommended: uv tool (fast, isolated)
uv tool install --from git+https://github.com/c4nc/twitter-cli@main twitter-cli

# Alternative: pipx
pipx install git+https://github.com/c4nc/twitter-cli@main
```

The **original** (upstream `public-clis/twitter-cli`, published on PyPI) is what most people have installed. It is at **0.8.5** on PyPI and *missing* the ClientTransaction fix, so `search` / `followers` / `following` / `likes` will 404 against X's current pages. Do not install it until upstream ships the fix:

```bash
# ⚠️ stale (0.8.5, missing the CT fix):
uv tool install twitter-cli
```

Upgrade to the latest fork (re-installs the newest `main`):

```bash
uv tool upgrade --reinstall twitter-cli
```

> **Tip:** Upgrade regularly to avoid unexpected errors from outdated API handling. If the original repository resumes maintenance and ships the fix, switch back to `uv tool install twitter-cli` (PyPI) and retire this fork — see [About this fork](#about-this-fork).

Install from source:

```bash
git clone https://github.com/c4nc/twitter-cli
cd twitter-cli
uv sync
```

## Quick Start

```bash
# Fetch home timeline (For You)
twitter feed

# Fetch Following timeline
twitter feed -t following

# Enable ranking filter explicitly
twitter feed --filter
```

## Usage

```bash
# Feed
twitter feed --max 50
twitter feed --cursor "<next-cursor-from-previous-response>"
twitter feed --full-text
twitter feed --output tweets.json
twitter feed --input tweets.json
twitter feed --json                    # Structured stdout for scripts/agents

# Bookmarks
twitter bookmarks
twitter bookmarks --full-text
twitter bookmarks --max 30 --yaml

# Search
twitter search "Claude Code"
twitter search "AI agent" -t Latest --max 50
twitter search "AI agent" --full-text
twitter search "topic" -o results.json         # Save to file
twitter search "trending" --filter              # Apply ranking filter

# Tweet detail (view tweet + replies)
twitter tweet 1234567890
twitter tweet 1234567890 --full-text
twitter tweet https://x.com/user/status/1234567890

# Open tweet by index from last list output
twitter show 2                         # Open tweet #2 from last feed/search
twitter show 2 --full-text             # Full text in reply table
twitter show 2 --json                  # Structured output

# List timeline
twitter list 1539453138322673664
twitter list 1539453138322673664 --cursor "<next-cursor-from-previous-response>"
twitter list 1539453138322673664 --full-text

# User
twitter user elonmusk
twitter user-posts elonmusk --max 20
twitter user-posts elonmusk --full-text
twitter user-posts elonmusk -o tweets.json
twitter likes elonmusk --max 30          # ⚠️ own likes only (private since Jun 2024)
twitter likes elonmusk --full-text
twitter likes elonmusk -o likes.json
twitter followers elonmusk --max 50
twitter following elonmusk --max 50

# Write operations
twitter post "Hello from twitter-cli!"
twitter post "Hello!" --image photo.jpg            # Post with image
twitter post "Gallery" -i a.png -i b.jpg -i c.webp  # Up to 4 images
twitter post "reply text" --reply-to 1234567890
twitter reply 1234567890 "Nice!" -i screenshot.png  # Reply with image
twitter quote 1234567890 "Look" -i chart.png        # Quote with image
twitter post "Hello from twitter-cli!" --json
twitter delete 1234567890
twitter like 1234567890
twitter like 1234567890 --yaml
twitter unlike 1234567890
twitter retweet 1234567890
twitter unretweet 1234567890
twitter bookmark 1234567890
twitter unbookmark 1234567890
twitter follow elonmusk --json
```

### Article

X long-form articles are **not paywalled** — but the API has no *article-by-id* operation: the number in an `/i/article/<id>` URL is the article's internal ID, and `twitter article` can't resolve it. The article content ships inside **the tweet that posted it**, so the working path is:

```bash
# Find the tweet that posted the article (the author's timeline, a reply, or a
# search for distinctive words from the title), then:
twitter article 2103542021071978601            # full article text
twitter article https://x.com/author/status/2103542021071978601
twitter article 2103542021071978601 --json     # .data[0].articleText
twitter article 2103542021071978601 --markdown # export as Markdown
twitter article 2103542021071978601 -o article.md
```

Passing an `/i/article/<id>` URL directly returns a precise error explaining this and pointing at the command above.

## Authentication

twitter-cli uses this auth priority:

1. **Environment variables**: `TWITTER_AUTH_TOKEN` + `TWITTER_CT0`
2. **Browser cookies** (recommended): auto-extract from Arc/Chrome/Edge/Firefox/Brave

Browser extraction is recommended — it forwards ALL Twitter cookies (not just `auth_token` + `ct0`) and aligns request headers with your local runtime, which is closer to normal browser traffic than minimal cookie auth.

**Chrome multi-profile**: All Chrome profiles are scanned automatically. To specify a profile:

```bash
TWITTER_CHROME_PROFILE="Profile 2" twitter feed
```

**Browser priority:** If you have multiple browsers, set `TWITTER_BROWSER` to try a specific browser first:

```bash
TWITTER_BROWSER=chrome twitter feed    # Supported: arc, chrome, edge, firefox, brave
```

**Custom Chromium profile:** For ungoogled-chromium or another Chromium installation started with `--user-data-dir`, point twitter-cli at that directory:

```bash
TWITTER_CHROMIUM_USER_DATA_DIR="/path/to/user-data-dir" twitter feed
```

The path may also point directly to a profile directory. Set `TWITTER_CHROME_PROFILE="Profile 2"` as well to select one profile below a User Data root.

After loading cookies, the CLI performs lightweight verification. Commands that require account access fail fast on clear auth errors (`401/403`).

## Proxy Support

Set `TWITTER_PROXY` to route all requests through a proxy:

```bash
# HTTP proxy
export TWITTER_PROXY=http://127.0.0.1:7890

# SOCKS5 proxy
export TWITTER_PROXY=socks5://127.0.0.1:1080
```

Using a proxy can help reduce IP-based rate limiting risks.

## Configuration

Create `config.yaml` in your working directory:

```yaml
fetch:
  count: 50

filter:
  mode: "topN"          # "topN" | "score" | "all"
  topN: 20
  minScore: 50
  lang: []
  excludeRetweets: false
  weights:
    likes: 1.0
    retweets: 3.0
    replies: 2.0
    bookmarks: 5.0
    views_log: 0.5

rateLimit:
  requestDelay: 2.5     # base delay between requests (randomized ×0.7–1.5)
  maxRetries: 3          # retry count on rate limit (429)
  retryBaseDelay: 5.0    # base delay for exponential backoff
  maxCount: 200          # hard cap on fetched items
```

Fetch behavior:

- `fetch.count` is the default item count for read commands when `--max` is omitted
- Rich table output truncates long tweet text by default; use `--full-text` to show full body text in list views

Filter behavior:

- Default behavior: no ranking filter unless `--filter` is passed
- With `--filter`: tweets are scored/sorted using `config.filter`

Scoring formula:

```text
score = likes_w * likes
      + retweets_w * retweets
      + replies_w * replies
      + bookmarks_w * bookmarks
      + views_log_w * log10(max(views, 1))
```

Mode behavior:

- `mode: "topN"` keeps the highest `topN` tweets by score
- `mode: "score"` keeps tweets where `score >= minScore`
- `mode: "all"` returns all tweets after sorting by score

## Best Practices (Avoiding Bans)

- **Use a proxy** — set `TWITTER_PROXY` to avoid direct IP exposure
- **Keep request volumes low** — use `--max 20` instead of `--max 500`
- **Space queries out** — wait 5–10s between read queries; after a 226, back off 30–60s
- **Don't run too frequently** — each startup fetches x.com to initialize anti-detection headers
- **Use browser cookie extraction** — provides full cookie fingerprint
- **Avoid datacenter IPs** — residential proxies are much safer

## Output Modes

- Use the default rich table for interactive reading
- Use `--full-text` when reading long posts in terminal tables
- Use `--yaml` or `--json` for scripts and agent pipelines
- Use `-c` / `--compact` when token efficiency matters more than completeness

## Troubleshooting

- `No Twitter cookies found`
  - Ensure you are logged in to `x.com` in a supported browser (Arc/Chrome/Edge/Firefox/Brave).
  - For a custom Chromium profile, set `TWITTER_CHROMIUM_USER_DATA_DIR` to its `--user-data-dir`.
  - Or set `TWITTER_AUTH_TOKEN` and `TWITTER_CT0` manually.
  - Run with `-v` to see browser extraction diagnostics.

- `Cookie expired or invalid (HTTP 401/403)`
  - Re-login to `x.com` and retry.

- `Unable to get key for cookie decryption` (macOS Keychain)
  - **SSH sessions**: Keychain is locked by default over SSH. Run:
    ```bash
    security unlock-keychain ~/Library/Keychains/login.keychain-db
    ```
  - **Local terminal**: Open **Keychain Access** → search for **"<Browser> Safe Storage"** → **Access Control** → add your Terminal app → **Save Changes**.
  - Or click **"Always Allow"** when the Keychain authorization popup appears.

- `Twitter API error 404` on `search` / `followers` / `following` / `likes`
  - If `feed` / `tweet` work but these 404, you have the **stale PyPI 0.8.5** (missing the ClientTransaction fix). Reinstall from this fork:
    ```bash
    uv tool install --reinstall --from git+https://github.com/c4nc/twitter-cli@main twitter-cli
    ```

- `Twitter API error 404` (other)
  - This can happen when upstream GraphQL query IDs rotate.
  - Retry the command; the client attempts a live queryId fallback.

- `not_found` on `twitter article <url>`
  - The URL is an `/i/article/<id>` (article internal ID). Use the **tweet that posted the article** — see [Article](#article).

- `Invalid tweet JSON file`
  - Regenerate input using `twitter feed --json > tweets.json`.

- **Windows: no output captured by pipe/subprocess** (AI agent integration)
  - This is a **ConPTY** issue, not a twitter-cli bug. Windows Terminal's ConPTY pseudo-terminal can intercept pipe output from commands with network latency.
  - **Fix**: Use **Git Bash** as your terminal shell and set `"windowsEnableConpty": false` in your terminal settings.
  - If disabling ConPTY with PowerShell, emoji output may fail with `UnicodeEncodeError: 'gbk'`. Git Bash handles UTF-8 natively.
  - Standard `subprocess.run(capture_output=True)` and file redirection (`> file 2>&1`) work correctly regardless of ConPTY.

Structured error codes commonly include `not_authenticated`, `not_found`, `invalid_input`, `rate_limited`, and `api_error`.

## Development

```bash
# Install dev dependencies
uv sync --extra dev

# Lint + type-check + tests
uv run ruff check .
uv run mypy twitter_cli
uv run pytest -q
```

CI validates the project on Python 3.10, 3.11, and 3.12.

### Project Structure

```text
twitter_cli/
├── cli.py             # Click CLI entry point
├── client.py          # HTTP client, ClientTransaction bootstrap (the CT fix)
├── graphql.py         # GraphQL query IDs, URL building, JS bundle scanning
├── parser.py          # Tweet, User, Media parsing logic
├── auth.py            # Cookie extraction & auth
├── cache.py           # Tweet cache (0600)
├── config.py
├── constants.py
├── exceptions.py
├── filter.py
├── formatter.py
├── output.py
├── search.py
├── serialization.py
├── timeutil.py
└── models.py
```

## Use as AI Agent Skill

twitter-cli ships with a [`SKILL.md`](./SKILL.md) — the **single source of truth** for AI-agent instructions (install-from-fork, auth workflow, rate-limit policy, article retrieval, error handling). It is updated alongside every functionality change to this repository (see [How this README stays current](#how-this-readme-stays-current)).

### Skills CLI (Recommended)

```bash
# Install the skill from this fork into the supported agent's directory
npx skills add c4nc/twitter-cli -g -a hermes-agent -y
#    (-g global/user-level, -a <agent>, -y non-interactive)

# Later, update it (re-pulls from this fork, not from PyPI):
npx skills update twitter-cli -g
```

> Note: the Skills CLI copies the repository's skill files; for this repo it means `SKILL.md`, `SCHEMA.md`, and `references/`. If you installed via a plain clone, prune anything else that was copied (tests, `twitter_cli/`, etc.).

The original is also available as `npx skills add public-clis/twitter-cli`, but it ships the upstream (pre-fix) skill with PyPI install instructions — do not use it until upstream ships the fix.

### Manual Install

```bash
mkdir -p .agents/skills
git clone https://github.com/c4nc/twitter-cli .agents/skills/twitter-cli
```

## License

[Apache-2.0](LICENSE) — same as the original project.
