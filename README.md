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
- **apple-design** — Apple Human Interface Guidelines-inspired design skill (from [emilkowalski/skills](https://github.com/emilkowalski/skills))

## Usage

To make these available in a Claude Code session, symlink or copy the
`skills/*` directories into `~/.claude/skills/`, e.g.:

```bash
git clone https://github.com/princehabib17/prompt-mastery /tmp/prompt-mastery
cp -r /tmp/prompt-mastery/skills/* ~/.claude/skills/
```

Each skill is discoverable individually (top-level folders under `skills/`)
so they load without extra nesting.
