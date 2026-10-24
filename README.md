# Computer Use + Browser Use — OpenCode skills

Two skills that let an AI agent drive a Mac and its Chrome browser like a person with hands: opening apps, clicking real pixels, typing, reading menus, managing tabs and forms — screenshots as proof, and a memory that compounds with every session.

## What's in here

| File | Skill |
|---|---|
| `computer-use-skill.md` | Full macOS control via one file: apps and windows, AppleScript dictionaries, raw Quartz mouse events, synthetic keyboard, clipboard, native dialogs, menu reading, screenshots with verification, keep-awake |
| `browser-use-skill.md` | Driving a connected Chrome from the harness: tabs, accessibility snapshots with refs, form filling, JS evaluation, console/network debugging, multi-tab lifecycle patterns |

## Install

Each skill is a single file. Drop them into your OpenCode skills directory:

```sh
mkdir -p ~/.config/opencode/skills/computer-use ~/.config/opencode/skills/browser-use
cp computer-use-skill.md ~/.config/opencode/skills/computer-use/SKILL.md
cp browser-use-skill.md ~/.config/opencode/skills/browser-use/SKILL.md
```

That's the whole setup. On first run, each skill seeds its own `LEARNINGS.md` next to the skill file — no extra config, and your machine's specifics stay on your machine.

## The self-learning loop

Both skills get better every time someone uses them:

1. **Session start** — read `LEARNINGS.md`; if missing, create it and record machine facts (screen geometry, installed tools, permission state).
2. **Every failed step** gets one fix attempt, then a dated one-line entry with the exact error text and the fix that worked.
3. **Novel wins** count too — a first-time-working recipe is logged with the exact commands.
4. **Promotion** — a lesson seen twice, or 3+ validated entries in one session, gets rewritten into the skill file in general wording and marked promoted.
5. **No fabrication** — unverified lessons stay marked `open`, never promoted. The skill file only carries what actually worked.

## Requirements

- macOS with Accessibility and Screen Recording permission granted to your terminal
- Chrome running for browser-use (no extensions needed)
- python3 with pyobjc for raw mouse events (no cliclick required)
- Any agent harness with shell access plus browser tooling works — the files are not tied to one setup

## Field notes

Both skills were hardened across 27 live tests before publishing: clipboard-to-app paste workflows, native save-dialog traps, Finder file ops, menu reading, volume and system-state toggling, Spotlight and System Settings deep links, multi-tab lifecycles, form flows (including a portal page that refused every submit path and got routed around), and the fullscreen-Space screenshot gate. Every failure and the fix that resolved it is documented inside the files, so you hit the fix on attempt one instead of discovering it the hard way.
