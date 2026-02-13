---
name: macos-ui-automation
description: >-
  This skill should be used when the user asks to "control a macOS dialog",
  "handle a save dialog", "automate macOS UI", "AppleScript 控制視窗",
  "處理儲存對話框", "下載檔案到指定位置", mentions native macOS dialog automation,
  or discusses controlling save/open/alert dialogs, file pickers, or other
  native macOS UI elements via AppleScript and System Events.
version: 0.1.1
tools: Bash
---

# macOS UI Automation

Automate native macOS dialogs (save, open, alert, print) and browser-triggered UI
via AppleScript System Events. Covers dialog detection, element interaction,
UI exploration for unknown dialogs, and Chrome JavaScript integration.

## Config

| Parameter | Value |
|-----------|-------|
| Default save path | `~/Downloads/claude_code_skill/` |

## Prerequisites

- **Accessibility permission**: The app running AppleScript (Terminal, iTerm, VS Code, etc.)
  must be granted access in **System Settings → Privacy & Security → Accessibility**.
  Without this, System Events commands will fail silently or error.
- **Default save directory**: Ensure `~/Downloads/claude_code_skill/` exists before saving.
  Create it if needed: `mkdir -p ~/Downloads/claude_code_skill/`

## Core Pattern

All native macOS UI automation follows this two-layer approach:

```
┌─────────────────────────────┐
│  Layer 1: App AppleScript   │  ← trigger actions (e.g., Chrome execute javascript)
│  tell application "AppName" │
└──────────┬──────────────────┘
           │ triggers native dialog
           ▼
┌─────────────────────────────┐
│  Layer 2: System Events     │  ← interact with native UI elements
│  tell process "AppName"     │
└─────────────────────────────┘
```

Key distinction:
- `tell application "X"` — app-level commands (open URL, execute JS, activate)
- `tell application "System Events" → tell process "X"` — native UI element control

## Workflow

### Step 1 — Identify the Target Process

The process name must match exactly. Common mappings:

| App | Process Name |
|-----|-------------|
| Google Chrome | `Google Chrome` |
| Safari | `Safari` |
| VS Code | `Code` |
| Finder | `Finder` |

Discover dynamically:
```applescript
tell application "System Events"
  name of every process where frontmost is true
end tell
```

### Step 2 — Detect the Dialog

Native dialogs appear as **sheets** (attached to window) or **standalone windows**.

```bash
osascript -e '
tell application "System Events"
  tell process "Google Chrome"
    exists sheet 1 of window 1
  end tell
end tell'
```

Wait for dialog to appear (poll with timeout):
```applescript
repeat 20 times
  try
    if exists sheet 1 of window 1 then exit repeat
  end try
  delay 0.5
end repeat
```

### Step 3 — Interact with Dialog Elements

#### Save Dialog (most common)

Element path: `splitter group 1 of sheet 1 of window 1`

Default save location is `~/Downloads/claude_code_skill/`. Before saving, ensure the
directory exists (`mkdir -p ~/Downloads/claude_code_skill/`). Since this is a custom path,
use Cmd+Shift+G to navigate instead of the location popup:

```bash
osascript -e '
tell application "System Events"
  tell process "Google Chrome"
    tell splitter group 1 of sheet 1 of window 1
      -- Set filename
      set value of text field "儲存為：" to "output.jpg"

      -- Navigate to default save path via Go to Folder
      keystroke "g" using {command down, shift down}
      delay 0.5
      keystroke "/Users/joneshong/Downloads/claude_code_skill/"
      keystroke return
      delay 0.5

      -- Save
      click button "儲存"
    end tell
  end tell
end tell'
```

For preset locations (桌面, 下載項目, etc.), use the popup instead:
```applescript
click pop up button "位置："
delay 0.3
click menu item "桌面" of menu 1 of pop up button "位置："
delay 0.3
```

#### Open Dialog

Similar to save. Key button: `button "打開"` instead of `button "儲存"`.
Navigate via Cmd+Shift+G, then click "打開".

#### Alert Dialog

```applescript
click button "好" of sheet 1 of window 1
```

### Step 4 — Explore Unknown Dialogs

When encountering an unfamiliar dialog, use `entire contents` to discover all elements:

```bash
osascript -e '
tell application "System Events"
  tell process "AppName"
    entire contents of sheet 1 of window 1
  end tell
end tell'
```

Parse the output to find `text field`, `button`, `pop up button`, `checkbox`, etc.
Narrow scope if output is too large: `entire contents of splitter group 1 of sheet 1 of window 1`.

### Step 5 — Verify Result

After dialog interaction, verify the expected outcome:
```bash
ls -lh ~/Downloads/claude_code_skill/output.jpg
```

## Browser Integration: Chrome AppleScript

Trigger browser-side actions, then handle the resulting native dialog:

**Step A** — Click a download button via JavaScript:
```bash
osascript -e '
tell application "Google Chrome"
  tell active tab of window 1
    execute javascript "
      const btns = document.querySelectorAll(\"button\");
      const dlBtn = Array.from(btns).find(b =>
        b.textContent.includes(\"下載\") ||
        b.getAttribute(\"aria-label\")?.includes(\"download\"));
      if (dlBtn) {
        dlBtn.closest(\"[class]\").dispatchEvent(
          new MouseEvent(\"mouseenter\", {bubbles: true}));
        setTimeout(() => dlBtn.click(), 500);
        \"clicked\";
      } else { \"not found\"; }
    "
  end tell
end tell'
```

**Step B** — Handle the save dialog (separate `osascript` call):
```bash
mkdir -p ~/Downloads/claude_code_skill/
osascript -e '
tell application "System Events"
  tell process "Google Chrome"
    repeat 20 times
      try
        if exists sheet 1 of window 1 then exit repeat
      end try
      delay 0.5
    end repeat
    tell splitter group 1 of sheet 1 of window 1
      set value of text field "儲存為：" to "downloaded-file.jpg"
      keystroke "g" using {command down, shift down}
      delay 0.5
      keystroke "/Users/joneshong/Downloads/claude_code_skill/"
      keystroke return
      delay 0.5
      click button "儲存"
    end tell
  end tell
end tell'
```

The `mouseenter` dispatch ensures hover-reveal buttons become visible before clicking.

## Quick Reference

### Dialog Element Labels (zh-TW)

| Element | zh-TW | English |
|---------|-------|---------|
| Filename field | `text field "儲存為："` | `text field "Save As:"` |
| Location popup | `pop up button "位置："` | `pop up button "Where:"` |
| Save button | `button "儲存"` | `button "Save"` |
| Cancel button | `button "取消"` | `button "Cancel"` |
| Open button | `button "打開"` | `button "Open"` |
| OK button | `button "好"` | `button "OK"` |
| Don't Save | `button "不儲存"` | `button "Don't Save"` |

### Location Popup Options

| zh-TW | English | Path |
|-------|---------|------|
| 桌面 | Desktop | `~/Desktop` |
| 下載項目 | Downloads | `~/Downloads` |
| 文件 | Documents | `~/Documents` |
| 應用程式 | Applications | `/Applications` |

### Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| "not allowed assistive access" | Missing permission | System Settings → Accessibility → add Terminal/app |
| Dialog not found | Dialog not yet rendered | Increase poll count or delay |
| Element label mismatch | Different macOS locale | Use `entire contents` to discover actual labels |
| Chrome JS returns error | Tab navigated away | Re-check active tab matches expected page |
| `set value` doesn't work | Field is read-only or custom | Try `keystroke` instead of `set value` |

## Continuous Improvement

This skill evolves with each use. After every invocation:

1. **Reflect** — Identify what worked, what caused friction, and any unexpected issues
2. **Record** — Append a concise lesson to `lessons.md` in this skill's directory
3. **Refine** — When a pattern recurs (2+ times), update SKILL.md directly

### lessons.md Entry Format

```
### YYYY-MM-DD — Brief title
- **Friction**: What went wrong or was suboptimal
- **Fix**: How it was resolved
- **Rule**: Generalizable takeaway for future invocations
```

Accumulated lessons signal when to run `/skill-optimizer` for a deeper structural review.

## Additional Resources

### Reference Files
- **`references/dialog-patterns.md`** — Complete element hierarchies for save, open, alert,
  and print dialogs; UI exploration technique; process name reference table; Chrome AppleScript
  integration patterns. Read when encountering an unfamiliar dialog type or needing detailed
  element paths.
