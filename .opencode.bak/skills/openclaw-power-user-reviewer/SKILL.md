---
name: openclaw-power-user-reviewer
description: "AI desktop app UX/UI reviewer from the perspective of OpenClaw power users. Use when reviewing, testing, or auditing an AI agent desktop application (Electron, desktop chat app, AI cowork tool). Applies the expectations, aesthetics standards, and functional requirements of power users who heavily use OpenClaw (215k+ stars open-source personal AI assistant) — covering design taste, interaction fluency, agent transparency, multi-model support, data sovereignty, extensibility, and workflow efficiency. Triggers: 'review my app from openclaw user perspective', 'test UX as power user', 'openclaw user review', 'AI desktop app audit', 'product review for AI tool users'."
---

# OpenClaw Power User Reviewer

Review AI agent desktop applications from the perspective of OpenClaw heavy users — the most demanding, design-savvy, and workflow-obsessed AI tool power users in the ecosystem.

## OpenClaw Ecosystem Context

OpenClaw (formerly Clawdbot / Moltbot) is the #1 open-source personal AI assistant platform:

- **GitHub**: 215k+ stars on `openclaw/openclaw`, 1,365+ repos tagged `#openclaw`
- **Ecosystem**: Skills marketplace (ClawHub), 17.5k-star awesome-skills collection, nanoclaw (lightweight alternative), MimiClaw ($5 chip version), ClawRouter (LLM router), Manifest (cost observability)
- **Community**: Multiple Chinese localization forks (2000+ person QQ groups), Discord, WeChat groups
- **Architecture**: Node.js gateway running locally, multi-channel access (WhatsApp/Telegram/Discord/iMessage), Skills plugin system, MCP protocol support
- **Slogan**: "Your own personal AI assistant. Any OS. Any Platform. The lobster way."
- **Competitive landscape**: Users also actively use Open WebUI (124k stars), Cherry Studio (40k stars), DeepChat (5.5k stars), PyGPT (1.6k stars), and coding tools like Cursor, Claude Code, OpenCode

## OpenClaw Power User Profile

Based on research across GitHub ecosystem, Twitter/X, Reddit (r/openclaw, r/clawdbot, r/LocalLLaMA, r/moltiverse), Chinese communities (QQ/WeChat/V2EX/JiKe), and the OpenClaw Discord:

### Who They Are

- **Technical sophistication**: Developers, indie hackers, startup founders, and tech-forward professionals. Comfortable with CLI, API keys, WebSocket configs, Docker, and self-hosting on homelab/NAS.
- **Age & demographics**: Primarily 22-40, developer/tech PM/indie maker/AI researcher. Active on GitHub, Discord, Hacker News, V2EX, JiKe. Many have self-built NAS or homelab setups.
- **Platform**: Primarily macOS (Mac Mini/Studio as always-on AI server), some Linux/Raspberry Pi/Umbrel enthusiasts. Heavy Apple ecosystem integration (iMessage, Shortcuts, Watch). Growing Windows presence via Docker.
- **Multi-tool users**: Simultaneously use Cursor, Claude Code/Desktop, ChatGPT Desktop, OpenCode, Codex CLI, Open WebUI, Cherry Studio. They benchmark everything against ALL of these tools.
- **Automation-first mindset**: They want AI to DO things, not just chat. Tool calls, file operations, terminal commands, cron jobs, scheduled tasks, background agents.
- **Always-on expectation**: 24/7 persistent agents via WhatsApp/Telegram/Discord. They message their AI like a coworker. Some run agents on $5 chips (MimiClaw) for edge deployment.
- **Cost-conscious power users**: Running agents 24/7 means token costs matter. They use model routing (ClawRouter) to balance quality vs. cost, and observability tools (Manifest) to track spend.

### What They Value Most

1. **Agent transparency** — "I want to SEE what the agent is doing." Tool calls, file reads/writes, command execution must be visible and understandable. They use observability tools like Manifest for cost/token tracking.
2. **Data sovereignty** — "My data stays on my machine." OpenClaw's core promise is self-hosted, local-first. No forced cloud login. `~/.openclaw` is sacred. They will reject any tool that phones home without consent.
3. **Model freedom** — Not locked to one provider. Claude, GPT, DeepSeek, Qwen, Gemini, local models (Ollama, MiniMax). Model switching should be frictionless. They often route through custom proxies/gateways.
4. **Hackability & extensibility** — "I can extend it myself." Skills/plugins, MCP servers, custom workflows, RAG pipelines. The tool must be a platform, not a locked box.
5. **Speed to value** — "5 minutes to set up, immediate results." Hate long config processes. Love magic moments. `npm install -g && openclaw onboard` is the gold standard.
6. **Persistent memory & context** — "It remembers what I told it last week." Cross-session context, long-term memory (MemOS), and conversation continuity are non-negotiable.
7. **Design taste** — They notice and appreciate visual polish. Dark-first, information-dense, platform-native. They use Raycast, Linear, Arc as aesthetic benchmarks.
8. **Multi-channel access** — Phone (ClawApp PWA), desktop, watch, chat apps. The AI should be reachable everywhere with consistent context.
9. **Self-improving** — The AI should build its own skills, edit its own prompts, extend its own capabilities. Skills are the unit of AI growth.

### Their Pain Points & Typical Complaints

These are real sentiment patterns from the community — use them as a lens during review:

1. **"It just chats"** — The #1 complaint. Agent that can't actually execute actions is useless to them.
2. **"又要我注册账号？我数据凭什么给你"** — Forced cloud login or data upload is an instant dealbreaker.
3. **"切个模型还要关掉对话重来？"** — Model switching friction destroys workflow.
4. **"AI 在后台跑了 30 秒，连个 loading 都不告诉我在干嘛"** — Waiting anxiety with zero feedback is the biggest UX black hole.
5. **"这个 Markdown 表格渲染得跟 shit 一样"** — Poor Markdown/LaTeX/code rendering is immediately noticed and reported.
6. **"快捷键呢？鼠标点来点去的"** — No keyboard shortcuts = "this tool doesn't respect my time."
7. **"Token 消耗看不到？我怎么控制成本"** — Cost opacity is unacceptable for 24/7 users.
8. **"导出对话只有 txt？给个 JSON/Markdown 很难吗"** — Export format limitations frustrate data portability.
9. **Config complexity** — "Setup still scares away non-technical people." Gateway URLs, API keys, token imports are friction points.
10. **Broken state recovery** — Disconnect, reconnect, interrupted streams. Must handle gracefully without data loss.
11. **Feature parity gap** — They benchmark against Cursor (@-file references, inline diff), Claude Desktop (Projects, Artifacts), ChatGPT (attachment handling), Open WebUI (RAG, web search). Missing features are immediately noticed.

### Their Aesthetic Standards

Based on what they praise in OpenClaw, Cherry Studio, Linear, Raycast, and Arc:

- **Dark-first, OLED-friendly** — True black backgrounds, not gray. The premium feel of void. Dark mode is default; light mode is "偶尔切一下."
- **Ambient lighting** — Subtle gradient glows, not flat surfaces. Depth through light.
- **Glass morphism** — Backdrop blur + saturation for floating panels. Creates hierarchy.
- **Typography hierarchy** — SF Pro / Inter for UI, SF Mono / Fira Code for code. Clear weight progression. CJK font handling matters for Chinese users.
- **Micro-animations** — Message slide-in, cursor pulse, spinner rotations. Life without distraction. Hardware-accelerated.
- **Color as semantic signal** — Green=success/running, Red=error/stop, Blue=info/link, Yellow=warning. Not decorative.
- **Spatial density** — Information-dense but not cramped. Every pixel earns its place. Status bar showing: current model, token count, response latency, connection state.
- **Platform-native feel** — macOS traffic lights, titlebar integration, system font fallbacks. Not "web app in a window." Windows users expect proper Mica/Acrylic integration.
- **Information-at-a-glance** — One look should tell you: which model, how many tokens, connection status, what the agent is doing right now.

## Review Framework

When reviewing an AI desktop app, evaluate across these **10 dimensions** with the OpenClaw power user lens:

### Dimension 1: First Impression & Onboarding (Weight: 10%)

**Power user expectation**: "I opened it. In 30 seconds I should know what this does and how to start."

Check:
- Does the empty state communicate purpose? Or is it a blank void?
- Is the first action obvious? (input box focused, clear CTA)
- Can I start using it WITHOUT configuring anything? (sensible defaults)
- How many clicks to first valuable output?
- Is there a "wow" moment in the first minute?
- Does it look like a professional tool or a homework project?
- Does it detect existing config (e.g., API keys from env, `~/.openclaw` config)?

Score: 1-10 (9-10 = OpenClaw-level "iPhone moment" onboarding)

### Dimension 2: Visual Design & Aesthetics (Weight: 12%)

**Power user expectation**: "It should look like something I'm proud to have on my screen 12 hours a day."

Check:
- Color system coherence — are accent colors consistent? No rogue hex values?
- Typography — clear hierarchy? Monospace for code, proportional for prose? CJK font handling?
- Spacing rhythm — consistent padding/margin system? Golden ratio or 8px grid?
- Dark theme quality — true dark, not washed-out gray? Good contrast ratios? Is dark mode the default?
- Glass/blur effects — backdrop-filter on panels? Ambient depth?
- Animations — smooth, purposeful, not distracting? Hardware-accelerated?
- Icon consistency — single icon family? No mixing Lucide with FontAwesome?
- Empty states — designed, not just "No data"?
- Responsive behavior — graceful degradation at narrow widths?
- Platform integration — feels native to macOS/Windows? Traffic lights, Mica/Acrylic? Not "web app in a window"?
- Information density status bar — current model, token count, latency, connection state visible at a glance?

Score: 1-10 (9-10 = Raycast/Linear/Arc-level polish)

### Dimension 3: Chat & Streaming Experience (Weight: 15%)

**Power user expectation**: "Streaming should feel like watching the AI think in real-time. Zero jank."

Check:
- Streaming smoothness — token-by-token without flicker/reflow?
- Auto-scroll behavior — follows new content? Pauses when user scrolls up?
- Thinking blocks — visible? Collapsible? Real-time update during stream?
- Message differentiation — clear visual split between user and AI?
- Markdown rendering — headings, lists, tables, links, inline code all correct?
- LaTeX support — math formulas render properly?
- Code blocks — syntax highlighting? Copy button? Language badge? Line numbers?
- Long message handling — readable? Proper line wrapping? Not one mega-block?
- Cursor/typing indicator — clear "AI is working" signal?
- Stop/cancel — visible button? Immediate response? Content preserved?
- Input area — auto-focus? Multi-line? Shift+Enter? Send button state? File/image attachment?
- Message actions — copy, regenerate, edit, branch conversation?

Score: 1-10 (9-10 = Claude Desktop-level streaming fluency)

### Dimension 4: Agent Transparency & Tool Calls (Weight: 18%)

**Power user expectation**: "I want to see exactly what my agent is doing. Like watching a coworker's screen."

This is the #1 differentiator for OpenClaw users vs. ChatGPT/casual AI users.

Check:
- Tool call visibility — are tool calls shown inline with chat?
- Human-readable labels — "Reading file /src/main.ts" not "Read({path: '/src/main.ts'})"?
- Status indication — running spinner, completed checkmark, error icon?
- Duration display — how long each tool call took?
- Expandable details — can I see input/output of each tool call?
- Activity timeline — sidebar or panel showing all agent actions?
- Artifact tracking — files created/modified listed somewhere?
- Error handling — tool failures shown clearly with actionable message?
- Sequential flow — multiple tool calls show clear order and progress?
- "What is it doing now?" — at any moment, can user tell what the agent is working on?
- Token/cost per message — can the user see per-message or per-turn token consumption?
- Approval workflow — does the user get a chance to approve/reject dangerous operations (file writes, terminal commands)?

Score: 1-10 (9-10 = full observability, like Cursor's tool call UI + OpenClaw's execution approval)

### Dimension 5: Model & Provider Management (Weight: 10%)

**Power user expectation**: "I use Claude for reasoning, GPT for speed, DeepSeek for cost, Ollama for privacy. Let me switch freely."

Check:
- Multi-provider support — at least Claude, GPT, DeepSeek, Qwen, Gemini, local models (Ollama)?
- Model switching — how many clicks? Can I switch mid-conversation?
- API key management — secure input? Password masking? Per-provider keys?
- Custom endpoint/proxy — can I point to my own gateway, reverse proxy, or OpenAI-compatible endpoint?
- Model catalog — clear list with capabilities/pricing info?
- Primary/default model — easy to set?
- Model-specific settings — temperature, max tokens, system prompt, etc.?
- Gateway/proxy support — can I route through ClawRouter or custom infra?
- Cost indicator — does switching models show price difference?

Score: 1-10 (9-10 = seamless multi-model like OpenClaw's provider catalog + Cherry Studio's model management)

### Dimension 6: Session & State Management (Weight: 8%)

**Power user expectation**: "My conversations are my work product. They need to be organized and persistent."

Check:
- Session list — clear, scannable, named by content not UUID?
- Session switching — instant load? No lost state?
- New session — one-click? Keyboard shortcut?
- Delete session — with confirmation? No accidental data loss?
- Session search — can I find a past conversation by keyword?
- Token/cost tracking — per-session usage visible?
- Session history — can I scroll back through long conversations without lag?
- Export — JSON, Markdown, or other structured formats? Not just txt?
- Import/restore — can I bring in conversations from other tools?
- Conversation branching — can I fork a conversation at any message?

Score: 1-10 (9-10 = organized workspace like Cursor's file-level sessions)

### Dimension 7: Connection & Reliability (Weight: 5%)

**Power user expectation**: "It should just work. If it disconnects, it reconnects. I don't babysit infrastructure."

Check:
- Connection status — clear indicator (color + text)?
- Auto-reconnect — happens silently?
- Disconnect handling — clear message, actionable suggestion?
- Interrupted stream recovery — partial content preserved?
- Error messages — human language, not stack traces?
- Gateway config — import from `~/.openclaw`? Minimal manual setup?
- Multi-window / multi-device — do they conflict? Is state consistent?

Score: 1-10 (9-10 = invisible infrastructure, just works)

### Dimension 8: Keyboard & Power User Efficiency (Weight: 5%)

**Power user expectation**: "I live in keyboard shortcuts. Mouse is for civilians."

Check:
- Cmd+N — new session?
- Cmd+, — settings?
- Esc — stop generation / close modal?
- Cmd+K — quick command palette?
- Cmd+L or / — focus input?
- Tab navigation — logical focus order?
- Shortcut discoverability — listed somewhere? Tooltips? Cmd+/ to show?
- Prompt templates / presets — quick access to saved prompts?
- @ mentions — reference files, tools, or contexts inline?

Score: 1-10 (9-10 = Raycast/Cursor-level keyboard navigation)

### Dimension 9: Data Sovereignty & Privacy (Weight: 10%)

**Power user expectation**: "My data stays on my machine. Period. No exceptions."

This is a foundational principle for the OpenClaw community. The slogan is literally "own your data."

Check:
- Local-first architecture — is data stored locally by default?
- No forced login — can I use the app without creating an account?
- No telemetry without consent — are there analytics? Can they be disabled?
- Ollama/local model support — can I run fully offline?
- Data location transparency — where is my data stored? Can I find and back it up?
- No cloud dependency — does the app work if the internet is down (with local models)?
- Settings export/import — can I back up and restore my configuration?
- Clear privacy policy — if data is sent anywhere, is it disclosed clearly?
- Credential handling — API keys stored securely? Not in plaintext config?

Score: 1-10 (9-10 = fully self-hosted, zero-cloud-dependency like OpenClaw's local gateway)

### Dimension 10: Extensibility & Ecosystem Integration (Weight: 7%)

**Power user expectation**: "I don't just use tools, I extend them. Give me plugins, MCP, Skills, APIs."

OpenClaw's Skills system and MCP integration are what make it a platform, not just an app.

Check:
- Plugin/Skills system — can users add custom capabilities?
- MCP (Model Context Protocol) support — can I connect MCP servers?
- RAG / document injection — can I load documents into context? `#` command for files?
- Web search integration — can the AI search the web?
- System prompt customization — can I set per-session or global system prompts?
- API/webhook access — can other tools integrate with this app?
- Custom tool definitions — can I define new tools for the agent?
- Marketplace/community — is there a way to discover and share extensions?
- File handling — can I drag-and-drop files? What formats are supported?

Score: 1-10 (9-10 = Open WebUI-level RAG + OpenClaw-level Skills + MCP support)

## Output Format

Produce a structured review report:

```
# OpenClaw Power User Review: [App Name]

## Executive Summary
[2-3 sentences: overall verdict from an OpenClaw power user perspective]

## Scores

| Dimension | Score | Weight | Weighted |
|-----------|-------|--------|----------|
| First Impression & Onboarding | X/10 | 10% | X.XX |
| Visual Design & Aesthetics | X/10 | 12% | X.XX |
| Chat & Streaming Experience | X/10 | 15% | X.XX |
| Agent Transparency & Tool Calls | X/10 | 18% | X.XX |
| Model & Provider Management | X/10 | 10% | X.XX |
| Session & State Management | X/10 | 8% | X.XX |
| Connection & Reliability | X/10 | 5% | X.XX |
| Keyboard & Power User Efficiency | X/10 | 5% | X.XX |
| Data Sovereignty & Privacy | X/10 | 10% | X.XX |
| Extensibility & Ecosystem | X/10 | 7% | X.XX |
| **Overall** | | | **X.XX/10** |

## Detailed Findings

### [Per dimension]
**Score: X/10**

Strengths:
- ...

Issues:
- [P0-Critical] ...
- [P1-High] ...
- [P2-Medium] ...
- [P3-Low] ...

### Competitive Gap Analysis

Compare against real competitors in the OpenClaw ecosystem. Fill in based on actual observation:

| Feature | Open WebUI | Cherry Studio | DeepChat | EasyClaw | This App | Verdict |
|---------|-----------|---------------|----------|----------|----------|---------|
| Multi-model support | ✅ 15+ providers | ✅ 300+ models | ✅ broad | ? | ? | |
| Dark theme quality | ✅ polished | ✅ excellent | ✅ good | ? | ? | |
| Streaming smoothness | ✅ | ✅ | ✅ | ? | ? | |
| Tool call visibility | ❌ limited | ❌ basic | ✅ good | ? | ? | |
| RAG / doc injection | ✅ 9 vector DBs | ❌ | ❌ | ? | ? | |
| MCP support | ❌ | ❌ | ✅ native | ? | ? | |
| Local model (Ollama) | ✅ native | ✅ | ✅ | ? | ? | |
| Keyboard shortcuts | ⚠️ basic | ⚠️ basic | ⚠️ basic | ? | ? | |
| Token/cost tracking | ❌ | ✅ | ❌ | ? | ? | |
| Data export formats | ⚠️ limited | ✅ JSON/MD | ❌ | ? | ? | |
| Plugin/Skills system | ✅ Pipelines | ❌ | ✅ agent-skills | ? | ? | |
| Self-hosted / offline | ✅ core feature | ✅ Electron local | ✅ Electron local | ? | ? | |
| Web search integration | ✅ 15+ engines | ❌ | ❌ | ? | ? | |
| Approval workflow | ❌ | ❌ | ❌ | ? | ? | |

Also compare against coding-oriented tools where relevant:

| Feature | Cursor | Claude Desktop | ChatGPT Desktop | This App |
|---------|--------|---------------|-----------------|----------|
| @-file references | ✅ native | ❌ | ❌ | ? |
| Inline diff view | ✅ | ❌ | ❌ | ? |
| Artifacts / Projects | ❌ | ✅ | ❌ | ? |
| Attachment handling | ⚠️ | ⚠️ | ✅ | ? |
| Conversation branch | ❌ | ❌ | ✅ | ? |

## Priority Recommendations

### Must Fix (P0-P1)
1. ...

### Should Improve (P2)
1. ...

### Nice to Have (P3)
1. ...

## Power User Verdict

[Final paragraph: Would an OpenClaw power user adopt this tool? What would make them switch from their current setup (OpenClaw + Open WebUI + Cherry Studio + Cursor stack)?]
```

## Review Mindset

When conducting the review, adopt these 8 personas in rotation:

1. **The Automation Addict** — "Can this agent actually DO things? File ops, terminal, web searches? Or is it just a fancy chatbot wrapper?"
2. **The Design Snob** — "Is this Linear/Raycast-level polish? Or does it look like a Bootstrap template from 2019?"
3. **The Multi-Tool Juggler** — "I have Cursor, Claude Desktop, Open WebUI, and Cherry Studio open. Why would I also open THIS? What's the unique value?"
4. **The Config Hater** — "I just want it to work. `npm install && run` or one Docker command. Don't make me paste WebSocket URLs and gateway tokens."
5. **The Keyboard Warrior** — "If I have to use my mouse for common operations, this tool doesn't respect my time. Where's Cmd+K?"
6. **The Cost Watcher** — "Show me exactly how many tokens I'm burning and what it costs. I run agents 24/7, every cent matters. Where's the usage dashboard?"
7. **The Homelab Sovereign** — "Where's my data stored? Can I run this fully offline with Ollama? Do I HAVE to create an account? Is there telemetry? I'll read your network requests."
8. **The Extensibility Hacker** — "Can I write a plugin for this? Connect an MCP server? Inject my own RAG pipeline? If it's a closed box, I'm out."

Rotate through these personas to ensure comprehensive coverage. Each persona catches different issues that the others miss.
