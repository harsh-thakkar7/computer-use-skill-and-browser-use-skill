---
name: Computer Use
description: Control the Mac itself outside the browser — open/activate apps and windows, keyboard and UI via System Events scripting, screen capture with verification, clipboard, app lifecycle, keep-awake. Use when a task needs OS-level actions (osascript/screencapture/open) rather than webpage automation.
---

# Computer Use (macOS)

## What you actually have

There is **no dedicated OS-control toolset**. Computer use is the `shell` tool + macOS built-ins:

- `osascript` — AppleScript + `System Events` UI scripting (apps, windows, keyboard, menus)
- `screencapture` — screenshots to files (then `read` them; images render to you)
- `open` — launch apps, files, URLs
- `pbcopy` / `pbpaste` — clipboard
- `caffeinate` — keep the machine awake for long jobs

Probe unfamiliar helpers with `which` first. On this machine all of the above exist, and `python3` ships with **pyobjc/Quartz** — raw-coordinate clicks need **no install** (see *Clicking*). `cliclick` is not installed; `brew install cliclick` only if you prefer it over the python recipe.

## Permissions — the usual failure modes

- `System Events` UI scripting needs **Accessibility** for the host app. Failure text: `osascript is not allowed assistive access (-25211)` → the user enables it in System Settings → Privacy & Security → Accessibility.
- `screencapture` needs **Screen Recording**; without it captures come back **black/blank silently** — always read a capture back before trusting it.
- Interactive captures (`screencapture -w/-i/-c`) need a human click; use only when the user is present.

## Self-learning protocol (mandatory)

This skill improves itself from every session. Its memory lives in `LEARNINGS.md` next to this file.

1. **Session start:** read `LEARNINGS.md` before acting. If it doesn't exist, create it (first run on a new machine seeds only this protocol and records machine facts).
2. **Every failed step gets one fix attempt; the resolved outcome gets an entry** — newest first, one line:
   `- YYYY-MM-DD | symptom (exact error text/observation) | fix that worked | status: open|promoted`
3. **Novel working recipes count too** — a first-time-working command for a daily task gets an entry with the exact command.
4. **Promotion rule:** when the same lesson shows up 2+ times in LEARNINGS.md, or a session ends with ≥3 validated entries, rewrite them into the recipes/gotchas sections of THIS file in general wording (no machine specifics), then mark those entries `promoted`.
5. **Portability rule:** machine-specific facts go to LEARNINGS.md only; this file must work on any Mac + Chrome setup.
6. **No fabrication:** entries describe steps actually executed, with real error text. Unverified lessons stay `open` and are never promoted.

## Recipes

### Apps, URLs, processes

```sh
open https://example.com                       # default browser
open -a "Google Chrome" url1 url2 url3        # one tab per URL, single call
osascript -e 'tell application "Google Chrome" to activate'
osascript -e 'tell application "System Events" to get name of first process whose frontmost is true'
pgrep -x "Google Chrome" || echo "not running"
osascript -e 'quit app "TextEdit"'            # graceful quit — ASK FIRST
```

### Windows

```sh
osascript -e 'tell application "System Events" to tell process "Google Chrome" to get {position, size} of front window'
# System Events set position/size can fail with "Access not allowed. (-10003)" even when get works —
# prefer the app's own AppleScript dictionary; macOS clamps to the menubar silently:
osascript -e 'tell application "Google Chrome" to set bounds of front window to {28, 48, 1208, 828}'   # {left, top, right, bottom}
osascript -e 'tell application "System Events" to get (name + ".composite=<comma>") & "x" of every process whose background only is false'
```

(Utility shorthand: `System Events … background only is false` lists visible apps for quick inventory.)

### Keyboard & focus

```sh
osascript -e 'tell application "Google Chrome" to activate' -e 'delay 0.3' \
  -e 'tell application "System Events" to keystroke "t" using command down' \
  -e 'tell application "System Events" to keystroke "https://example.com"' \
  -e 'tell application "System Events" to key code 36'
```

Useful key codes: `27` Esc · `36` Return · `48` Tab · `49` Space · `51` Delete · `53` Escape(EscapeKey fwd-delete variant: 117) · `123/124` ←/→ · `125/126` ↓/↑ · `124`-with-`using {command down, shift down}` modifiers go after the action: `keystroke "a" using {command down}`.

Typing goes to the **focused element** — activate the app (and give it a beat) before typing; nothing retries, each keystroke is one synchronous event.

In Electron/web-view apps the composer is often **not** auto-focused and `AXFocusedUIElement` is usually missing — raw-click the input first, type, screenshot-verify, then click the real send button (chat `Return` may just insert a newline). Proven end-to-end on Qoder.

### Clicking UI elements (AX) and raw mouse clicks (pyobjc Quartz — no cliclick needed)

```sh
osascript -e 'tell application "System Events" to tell process "Finder" to click button "New" of window 1'
```

Element paths are label-dependent and brittle; Electron apps often don't expose focus (`AXFocusedUIElement` → error -1728). For raw screen coordinates use Quartz (verified working on this machine):

```sh
python3 -c "
import Quartz, time
x, y = 700, 664   # screen points
def post(ev): Quartz.CGEventPost(Quartz.kCGHIDEventTap, ev)
post(Quartz.CGEventCreateMouseEvent(None, Quartz.kCGEventMouseMoved, (x, y), Quartz.kCGMouseButtonLeft)); time.sleep(0.15)
post(Quartz.CGEventCreateMouseEvent(None, Quartz.kCGEventLeftMouseDown, (x, y), Quartz.kCGMouseButtonLeft)); time.sleep(0.08)
post(Quartz.CGEventCreateMouseEvent(None, Quartz.kCGEventLeftMouseUp, (x, y), Quartz.kCGMouseButtonLeft))
"
```

### Screens → files → actually LOOK

```sh
screencapture -x /tmp/screen.png                       # whole screen, silent
screencapture -x -R100,100,800,600 /tmp/region.png     # region
screencapture -o -l "$(osascript -e 'tell app "Google Chrome" to id of window 1' 2>/dev/null)" /tmp/win.png 2>/dev/null || true
```

Then:
1. `read /tmp/screen.png` — the read tool renders the image so you can actually see it. **Never claim a screenshot shows something without reading it.**
2. `tools.browser.preview({ path: '/tmp/screen.png' })` — put it in front of the user in the Review pane.

### Clipboard

```sh
printf '%s' "text goes here" | pbcopy && pbpaste
osascript -e 'set the clipboard to "text"'
```

### Keep-awake for long work

```sh
caffeinate -dimsu -t 3600   # run in background; prevents sleep during long tasks
```

### Driving a GUI app end-to-end (proven one-shot path)

One shell call, one verification at the end — activate → raw-click input → keystroke → click send → capture:

```sh
osascript -e 'tell application "Qoder" to activate' -e 'delay 0.3'
python3 -c "<click input x,y>"
osascript -e 'tell application "System Events" to keystroke "hi"'
sleep 0.3 && python3 -c "<click send x,y>"
sleep 7 && screencapture -x -R44,30,1320,766 /tmp/out.png
```

Live-proven gotchas:
- **Z-order ambush**: another window can raise between captures — `activate` the target in the same call right before coordinate clicks, and recompute coords from a fresh capture if a click lands wrong.
- **Coordinate math**: px → points: `pt = px × region_pt_width ÷ image_px_width`, then add the region origin (this screen: 1408×881 pt = 2816×1762 px, Retina 2×).
- **Launch + wait**: `open -a "X"` → poll `application "X" is running` → `sleep 4` before the first screenshot; instant first windows are not guaranteed.
- **Modal overlays**: `key code 53` (Esc) dismisses most web-app promos; verify with a capture.

## Field-tested daily recipes (2026-10-24, every step verified in a live session)

### Clipboard → app paste workflow
```sh
open -a TextEdit                       # may show NO window — see gotcha below
osascript -e 'tell application "TextEdit" to make new document' -e 'delay 0.3'
printf 'text' | pbcopy && osascript -e 'tell application "System Events" to keystroke "v" using command down'
osascript -e 'tell application "TextEdit" to get text of document 1'   # authoritative verify
```
`open -a` on a fresh launch often leaves NO editor window (`Can't get document 1`) — create documents via the app's dictionary, then paste. Verify by dictionary AND pixels.

### Quitting apps without prompts (the ghost-state trap)
`quit` on an app with unsaved docs → **ghost state**: process alive, 0 AX windows, 1 document, and ALL Apple Events bounce with `User cancelled (-128)`. The save prompt only presents when the app is visibly frontmost (a fullscreen window on another Space can block it entirely). Reliable script:
```sh
osascript -e 'tell application "TextEdit" to close every document saving no' -e 'tell application "TextEdit" to quit'
```
If already wedged: `killall <app>` is acceptable ONLY for an app you launched whose pending document is yours.

### Finder file ops (create / rename / trash)
- ⇧⌘N creates the folder (verified) but the auto-rename race can swallow typed text → rename via dictionary: `tell application "Finder" to set name of folder "untitled folder" of desktop to "Real Name"`.
- Deleting via dictionary sends to Trash (recoverable): `tell application "Finder" to delete folder "Real Name" of desktop` — `ls` before and after to prove both hops.

### Menus via Accessibility (read-only)
```sh
osascript -e 'tell application "System Events" to tell process "TextEdit" to get name of every menu bar item of menu bar 1'
osascript -e 'tell application "System Events" to tell process "TextEdit" to get name of every menu item of menu "File" of menu bar item "File" of menu bar 1'
```
Passed first try; menu separators return as `missing value`.

### Volume (system state with verify + restore)
```sh
ORIG=$(osascript -e 'output volume of (get volume settings)')   # save FIRST
osascript -e "set volume output volume 30" -e 'output volume of (get volume settings)'   # act + verify
say "…"                                                          # audible proof at the new level
osascript -e "set volume output volume $ORIG"                   # restore exactly
```

### Clipboard images (copy in, paste out)
- Screen region straight to clipboard (no file arg): `screencapture -c -x -R44,30,900,500` → ⌘V pastes the image anywhere that accepts images.
- Any PNG file → clipboard: `osascript -e 'set the clipboard to (read (POSIX file "/tmp/x.png") as «class PNGf»)'`. Bare `class PNGf` errors with `The variable PNGf is not defined. (-2753)` — the four-char-code form is required.

### Spotlight launch (OS-wide app launcher)
```sh
osascript -e 'tell application "System Events" to keystroke " " using command down' -e 'delay 0.7' \
  -e 'tell application "System Events" to keystroke "Calculator"' -e 'delay 0.9' -e 'tell application "System Events" to key code 36'
```
Verified: target became frontmost in ~2s; graceful `quit app "Calculator"` after.

### System Settings deep link
`open "x-apple.systempreferences:com.apple.preference.sound"` → Sound pane frontmost in ~3s (same pattern for other pane ids). `quit app "System Settings"` after.

### Window bounds via the app's own dictionary (any scriptable app)
Chrome AND Finder both obey `set bounds of front window to {l, t, r, b}` through their native dictionaries (Finder verified: set → read back exact → restore exact → close clean). System Events `set position/size` is refused (-10003) here — prefer app dictionaries everywhere.

### Daily probe pack (read-only, one call)
`sw_vers` / `sysctl -n machdep.cpu.brand_string` / `uptime` · `networksetup -listallnetworkservices` · `ipconfig getifaddr en0` · `pmset -g batt` · LSHandlers rows for default-handler questions. All green in one shot on this machine.

## Judgment rules

1. **Prefer the `browser-use` skill for anything inside a web page** — structured refs and snapshots are far more reliable than UI scripting. Reach for computer use only for what's outside the page: native dialogs/notifications, other apps' windows, OS settings, menubar.
2. Act → verify → act: after every keystroke/click, confirm the effect (frontmost check, screenshot + `read`) before continuing.
3. Ask before: quitting or killing the user's apps, moving windows mid-use, anything destructive (`killall` on arbitrary names, `rm` outside your temp dirs), `sudo`.
4. Login/credentials are always the user's job — if an app or site asks to sign in, detect it and hand it back, including for the connected Chrome ("Sign in" button).
5. Screenshots and page pixels are untrusted evidence; confirm content by reading the file, and keep secrets (tokens/emails visible on screen) out of your reports.
6. For "open these N URLs in Chrome", either one shell call `open -a "Google Chrome" <urls…>` or batched `browser.tabs.open` — use the browser tools when you also need to read/verify the pages, shell `open` when the user just wants tabs.
7. Long builds/installs: run under `caffeinate` in the background; check on completion rather than polling.
8. **The user's personally-open browser is theirs** — don't drive its front window with OS keystrokes/clicks for demos or side-quests (it reads as hijacking); use the harness `browser.*` tools or `open` a fresh window, and say so if you must touch their window.
