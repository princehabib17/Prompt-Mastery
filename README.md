# prompt-mastery

Personal collection of Claude Code skills, kept in a dedicated repo so they
persist across sessions/environments instead of living only in one
container's `~/.claude/skills/`.

## Skills included

- **higgsfield** — Higgsfield AI prompt-writing skill (video/image prompts, model choice, camera controls, Soul ID, etc.)
- **scroll-film-studio** — builds cinematic "scroll-film" websites (GSAP/Lenis pure-code lane, or AI-video lane via Higgsfield/Kie.ai)
- **img2threejs** — turns a reference image into a procedural, animation-ready Three.js model
- **lanshu-awesome-ai-video-kit** — AI video prompt-engineering toolkit (543 tested prompts, 15 models), including 7 sub-skills:
  - `kling-prompter`, `seedance-prompter`, `seedance-storyboard`, `seedance-debugger`,
    `happyhorse-prompter`, `model-selector`, `prompt-translator`
- **scrollcraft** — builds premium scroll-driven interactive landing pages (scroll-scrubbed video, pinned sections, signature page grammar per brand)
- **emil-design-eng** — Emil Kowalski's design-engineering philosophy: UI polish, component design, animation decisions, the invisible details that make software feel right ([source](https://github.com/emilkowalski/skills))
- **impeccable** — Paul Bakaus's design-fluency skill: 24 commands (`polish`, `audit`, `critique`, `animate`, `bolder`, `quieter`, etc.) plus curated anti-pattern detection ([source](https://github.com/pbakaus/impeccable))
- **taste-skill** — anti-slop frontend skill for landing pages/portfolios/redesigns: reads the brief, infers the right design direction, avoids templated-looking output ([source](https://github.com/leonxlnx/taste-skill))

## Usage

To make these available in a Claude Code session, symlink or copy the
`skills/*` directories into `~/.claude/skills/`, e.g.:

```bash
git clone https://github.com/princehabib17/prompt-mastery /tmp/prompt-mastery
cp -r /tmp/prompt-mastery/skills/* ~/.claude/skills/
```

Each skill is discoverable individually (top-level folders under `skills/`)
so they load without extra nesting.

## External integrations (plugins & MCP servers)

This repo ships a project-scoped [`.mcp.json`](.mcp.json) that wires up
Playwright, Firecrawl, and Perplexity as MCP servers, plus Composio (via its
hosted Rube endpoint), for any Claude Code session opened in this directory.
Claude Code will prompt you to trust/load it the first time. Set the two
required env vars below before starting a session, or leave them unset if
you only need the tools that don't require a key.

| Tool | How it's wired up | Auth needed |
| --- | --- | --- |
| **Playwright** | `.mcp.json` → `npx @playwright/mcp@latest` (stdio) | none |
| **Firecrawl** | `.mcp.json` → `npx firecrawl-mcp` (stdio) | `FIRECRAWL_API_KEY` (scrape/search work keyless, rate-limited) |
| **Perplexity** | `.mcp.json` → `npx @perplexity-ai/mcp-server` (stdio) | `PERPLEXITY_API_KEY` (required) |
| **Composio** | `.mcp.json` → `https://rube.app/mcp` (http) | browser OAuth on first use, no key needed |
| **Figma** | `.mcp.json` → `http://127.0.0.1:3845/mcp` (http, local) | Figma desktop app open with Dev Mode MCP Server enabled |
| **Codex** | not wired up here — see note below | — |

### Playwright
Official Anthropic-maintained plugin (bundles the same MCP server as the
`.mcp.json` entry above), if you'd rather install it as a plugin instead:

```bash
claude plugin install playwright@claude-plugins-official
```

Useful flags for the underlying server: `--headless`, `--browser
<chrome|firefox|webkit|msedge>`, `--isolated`, `--caps <vision,pdf,...>`.

### Firecrawl
Official plugin (adds skills that drive the separate `firecrawl-cli`, not
the bare MCP server):

```bash
claude plugin install firecrawl@claude-plugins-official
npm install -g firecrawl-cli
firecrawl login --api-key "fc-YOUR-API-KEY"   # or: firecrawl login --browser
```

Get a key at https://firecrawl.dev/app/api-keys. The `.mcp.json` entry above
is the lighter-weight, MCP-only alternative if you don't want the CLI/plugin.

### Perplexity
No official Claude Code plugin exists yet, so it's MCP-only. Tools exposed:
`perplexity_search`, `perplexity_ask`, `perplexity_research`,
`perplexity_reason`. Get a key at
https://www.perplexity.ai/settings/api.

### Composio
No official Claude Code plugin exists yet. The `.mcp.json` entry points at
[Rube](https://rube.app), Composio's hosted MCP server, which fronts 500+
app integrations (Gmail, Slack, GitHub, Notion, etc.). Authentication is an
interactive OAuth flow per app the first time you use it — no API key to
set up front.

### Figma
The `.mcp.json` entry points at Figma's local Dev Mode MCP server, which
only responds while the Figma desktop app is open with **Preferences →
Enable Dev Mode MCP Server** turned on — no separate install. If you'd
rather use Figma's officially maintained plugin (adds design-to-code,
code-to-design, and diagramming skills on top of the same server):

```bash
claude plugin install figma@claude-plugins-official
```

### Codex
OpenAI's Codex CLI removed its built-in `codex mcp-server` command
(deprecated in v0.149, removed in v0.154), so it isn't included in
`.mcp.json`. Codex still:

- connects **to** external MCP servers (`codex mcp add/list/...`), and
- runs standalone as a coding agent (`codex`, `codex exec "..."`, `codex
  review`).

Install with `npm install -g @openai/codex`, then either `codex login`
(ChatGPT account) or set `OPENAI_API_KEY`. There's no first-party way to
expose Codex itself as an MCP tool right now; third-party wrappers (e.g.
`codex-mcp-server` on npm) exist but are unverified and unofficial —
review them before trusting them with your OpenAI credentials.
