---
name: agent-reach
description: "Internet access toolkit for AI agents. Read & search Twitter/X, Reddit, YouTube, GitHub, Bilibili, XiaoHongShu, LinkedIn, Boss, RSS, and any web page. Use when the user asks to read a tweet, search Twitter, watch/summarize a YouTube or Bilibili video, search Reddit, read XiaoHongShu posts, browse the web, search the internet, read RSS feeds, or access any online content. Triggers: 'read this tweet', 'search Twitter', 'summarize this video', 'search Reddit', 'read this web page', 'search the internet', 'read XiaoHongShu', 'check this GitHub repo', 'read RSS', 'search YouTube', 'search Bilibili'."
homepage: https://github.com/Panniantong/Agent-Reach
metadata:
  {
    "openclaw":
      {
        "emoji": "🌐",
        "requires": { "anyBins": ["curl", "bird", "yt-dlp", "gh", "mcporter"] },
        "install":
          [
            {
              "id": "uv",
              "kind": "uv",
              "package": "https://github.com/Panniantong/agent-reach/archive/main.zip",
              "bins": ["agent-reach"],
              "label": "Install agent-reach (pip)",
            },
          ],
      },
  }
---

# Agent Reach - Internet Access Toolkit

Give your AI agent access to the entire internet. This skill enables reading and searching across major platforms with zero API fees.

## Supported Platforms & Commands

### Web Pages (always available)

Read any web page using Jina Reader:

```bash
curl -s "https://r.jina.ai/URL"
```

- Works on any public URL
- Returns clean markdown, not raw HTML
- No API key needed

### Twitter / X

Requires: `bird` CLI + Twitter cookies configured.

```bash
# Read a single tweet
bird read "https://x.com/user/status/123" --json

# Search tweets
bird search "keyword" --json

# View user timeline
bird timeline @username --json
```

**Cookie setup:** Use Cookie-Editor Chrome extension on x.com, export as Header String, then:

```bash
agent-reach configure twitter-cookies "PASTED_COOKIE_STRING"
```

### YouTube

Requires: `yt-dlp`

```bash
# Get video metadata + subtitles
yt-dlp --dump-json "https://youtube.com/watch?v=xxx"

# Extract subtitles only
yt-dlp --write-sub --write-auto-sub --sub-lang en --skip-download -o "%(title)s" "URL"

# Search YouTube
yt-dlp "ytsearch10:keyword" --dump-json --flat-playlist
```

### Bilibili

Requires: `yt-dlp` (same tool as YouTube)

```bash
# Get video info + subtitles
yt-dlp --dump-json "https://www.bilibili.com/video/BVxxx"

# Search Bilibili
yt-dlp "bilisearch10:keyword" --dump-json --flat-playlist
```

Note: On servers, may need a proxy configured via `agent-reach configure proxy URL`.

### GitHub

Requires: `gh` CLI

```bash
# View a repository
gh repo view owner/repo

# Search repositories
gh search repos "LLM framework" --limit 10

# Search code
gh search code "function_name" --repo owner/repo

# List issues
gh issue list --repo owner/repo

# Read an issue
gh issue view 123 --repo owner/repo
```

For private repos: `gh auth login`

### Reddit

```bash
# Read a post + comments (JSON API)
curl -s -H "User-Agent: AgentReach/1.0" "https://www.reddit.com/r/subreddit/comments/postid.json"

# Search via Exa (semantic search, better quality)
mcporter call 'exa.web_search_exa(query: "site:reddit.com your search query", numResults: 10)'
```

On servers: may need proxy for direct Reddit access.

### Internet Search (Exa)

Requires: `mcporter` with Exa configured.

```bash
# Semantic web search
mcporter call 'exa.web_search_exa(query: "your search query", numResults: 10)'

# Search with content extraction
mcporter call 'exa.web_search_exa(query: "your query", numResults: 5, textContentsOptions: "text")'
```

### XiaoHongShu (via MCP)

Requires: Docker + xiaohongshu-mcp running.

```bash
# Search posts
mcporter call 'xiaohongshu.search_feeds(keyword: "keyword", page: 1)'

# Read a specific post
mcporter call 'xiaohongshu.get_feed_detail(note_id: "NOTE_ID")'

# Get comments
mcporter call 'xiaohongshu.get_feed_comments(note_id: "NOTE_ID")'
```

Setup:

```bash
docker run -d --name xiaohongshu-mcp -p 18060:18060 xpzouying/xiaohongshu-mcp
mcporter config add xiaohongshu http://localhost:18060/mcp
```

Then open http://localhost:18060 to scan QR code for login.

### LinkedIn (via MCP)

Requires: `linkedin-scraper-mcp`

```bash
# Get person profile
mcporter call 'linkedin.get_person_profile(linkedin_url: "https://linkedin.com/in/username")'

# Search jobs
mcporter call 'linkedin.search_jobs(keywords: "software engineer", location: "San Francisco")'
```

Basic public pages can also be read via Jina Reader: `curl -s "https://r.jina.ai/https://linkedin.com/in/username"`

### Boss (via MCP)

Requires: `mcp-bosszp`

```bash
# Search jobs
mcporter call 'bosszhipin.search_jobs_tool(keyword: "Python", city: "beijing")'

# Send greeting to HR
mcporter call 'bosszhipin.start_chat_tool(job_id: "JOB_ID", greeting: "Hello")'
```

### RSS Feeds

```bash
python3 -c "
import feedparser
d = feedparser.parse('https://example.com/feed.xml')
for e in d.entries[:5]:
    print(f'{e.title}: {e.link}')
"
```

## Diagnostics

Check which channels are available:

```bash
agent-reach doctor
```

## Installation

If `agent-reach` is not installed:

```bash
pip install https://github.com/Panniantong/agent-reach/archive/main.zip
agent-reach install --env=auto
```

Safe mode (no auto system changes):

```bash
agent-reach install --env=auto --safe
```

## Important Notes

- All tools are free and open-source. No paid API keys required.
- Cookie-based platforms (Twitter, XiaoHongShu): use a dedicated/secondary account for safety.
- Proxy (~$1/month) only needed on servers for Reddit/Bilibili access.
- Local machines generally work without proxy.
- When a platform returns errors, run `agent-reach doctor` to diagnose.
