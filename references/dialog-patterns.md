# Dialog Patterns — macOS Native UI Elements

Verified element hierarchies for common macOS native dialogs, accessed via
`tell application "System Events" → tell process "<ProcessName>"`.

> **Locale**: All labels below assume **zh-TW** (Traditional Chinese) macOS.
> English equivalents are noted in parentheses where applicable.

---

## Table of Contents

- [Save Dialog (Sheet)](#save-dialog-sheet)
- [Open / File Picker Dialog](#open--file-picker-dialog)
- [Alert Dialog](#alert-dialog)
- [Print Dialog](#print-dialog)
- [UI Exploration Technique](#ui-exploration-technique)
- [Process Name Reference](#process-name-reference)
- [Chrome AppleScript Integration](#chrome-applescript-integration)

---

## Save Dialog (Sheet)

Appears as a **sheet** attached to the app window (not a standalone window).

### Element Path

```
process "<AppName>"
  └─ window 1
       └─ sheet 1
            └─ splitter group 1
                 ├─ text field "儲存為：" (en: "Save As:")     — filename input
                 ├─ pop up button "位置：" (en: "Where:")      — location dropdown
                 ├─ button "儲存" (en: "Save")                 — confirm
                 └─ button "取消" (en: "Cancel")               — cancel
```

### Set Filename

```applescript
tell splitter group 1 of sheet 1 of window 1
  set value of text field "儲存為：" to "my-file.jpg"
end tell
```

### Select Location (Preset)

Common menu items in the location popup:
- `桌面` (Desktop)
- `下載項目` (Downloads)
- `文件` (Documents)
- `應用程式` (Applications)
- Any Finder sidebar favorites

```applescript
tell splitter group 1 of sheet 1 of window 1
  click pop up button "位置："
  delay 0.3
  click menu item "桌面" of menu 1 of pop up button "位置："
  delay 0.3
end tell
```

### Select Location (Custom Path)

Use Cmd+Shift+G to open "Go to Folder" when the path isn't in the popup:

```applescript
tell splitter group 1 of sheet 1 of window 1
  keystroke "g" using {command down, shift down}
  delay 0.5
  keystroke "/Users/joneshong/custom/path/"
  keystroke return
  delay 0.5
end tell
```

### Click Save

```applescript
tell splitter group 1 of sheet 1 of window 1
  click button "儲存"
end tell
```

### Wait for Dialog to Appear

```applescript
repeat 20 times
  try
    if exists sheet 1 of window 1 then exit repeat
  end try
  delay 0.5
end repeat
```

---

## Open / File Picker Dialog

Similar structure to Save, but may appear as a **sheet** or **standalone window**.

### Element Path (Sheet Variant)

```
process "<AppName>"
  └─ window 1
       └─ sheet 1
            ├─ splitter group 1
            │    ├─ text field (filename/search)
            │    ├─ pop up button (file type filter)
            │    └─ browser/outline (file list)
            ├─ button "打開" (en: "Open")
            └─ button "取消" (en: "Cancel")
```

### Element Path (Standalone Window Variant)

```
process "<AppName>"
  └─ window "打開" (en: "Open")
       ├─ splitter group 1
       │    └─ ... (same inner structure)
       ├─ button "打開"
       └─ button "取消"
```

### Navigate to Path

```applescript
keystroke "g" using {command down, shift down}
delay 0.5
keystroke "/path/to/target/file.pdf"
keystroke return
delay 0.5
click button "打開"
```

---

## Alert Dialog

Simple confirmation/warning dialogs.

### Element Path

```
process "<AppName>"
  └─ window 1
       └─ sheet 1   (or standalone dialog)
            ├─ static text "..."  — message text
            ├─ button "好" (en: "OK")
            ├─ button "取消" (en: "Cancel")
            └─ button "不儲存" (en: "Don't Save")  — if applicable
```

### Example: Dismiss Alert

```applescript
tell process "<AppName>"
  if exists sheet 1 of window 1 then
    click button "好" of sheet 1 of window 1
  end if
end tell
```

---

## Print Dialog

### Element Path

```
process "<AppName>"
  └─ window 1
       └─ sheet 1
            ├─ pop up button "印表機：" (en: "Printer:")
            ├─ text field (copies)
            ├─ checkbox "全部" (en: "All") / radio buttons for page range
            ├─ button "列印" (en: "Print")
            ├─ button "取消" (en: "Cancel")
            └─ menu button "PDF"  — save as PDF option
```

### Save as PDF via Print Dialog

```applescript
tell process "<AppName>"
  tell sheet 1 of window 1
    click menu button "PDF"
    delay 0.3
    click menu item "儲存為 PDF⋯" of menu 1 of menu button "PDF"
    -- This opens a Save dialog (see Save Dialog pattern above)
  end tell
end tell
```

---

## UI Exploration Technique

When encountering an unfamiliar dialog, use `entire contents` to discover all elements:

### Step 1: Confirm Dialog Exists

```applescript
tell application "System Events"
  tell process "<AppName>"
    exists sheet 1 of window 1
  end tell
end tell
```

### Step 2: List All Elements

```applescript
tell application "System Events"
  tell process "<AppName>"
    entire contents of sheet 1 of window 1
  end tell
end tell
```

This returns a flat list of every UI element. Parse it to find:
- `text field` — input fields (look for their `description` or title)
- `button` — clickable buttons (titles like "儲存", "取消")
- `pop up button` — dropdown menus
- `checkbox` — toggle options
- `static text` — labels and messages

### Step 3: Read Specific Properties

```applescript
tell application "System Events"
  tell process "<AppName>"
    properties of button 1 of sheet 1 of window 1
  end tell
end tell
```

### Step 4: Try Interactions

```applescript
tell application "System Events"
  tell process "<AppName>"
    click button "儲存" of splitter group 1 of sheet 1 of window 1
  end tell
end tell
```

> **Tip**: If `entire contents` output is too large, narrow the scope:
> `entire contents of splitter group 1 of sheet 1 of window 1`

---

## Process Name Reference

The `process` name in System Events must match the app's process name exactly.

| Application | Process Name | Notes |
|-------------|-------------|-------|
| Google Chrome | `Google Chrome` | |
| Safari | `Safari` | |
| Firefox | `Firefox` | |
| Arc | `Arc` | |
| Finder | `Finder` | |
| Preview | `Preview` / `預覽程式` | Varies by locale |
| TextEdit | `TextEdit` | |
| VS Code | `Code` | Not "Visual Studio Code" |
| Cursor | `Cursor` | |
| Terminal | `Terminal` | |
| iTerm2 | `iTerm2` | |
| Slack | `Slack` | |
| Discord | `Discord` | |

### Discover Process Name

```applescript
tell application "System Events"
  name of every process where frontmost is true
end tell
```

Or for a specific app:

```applescript
tell application "System Events"
  name of process "Google Chrome"
end tell
```

---

## Chrome AppleScript Integration

Execute JavaScript in Chrome's active tab to trigger browser-side actions:

### Basic Pattern

```applescript
tell application "Google Chrome"
  tell active tab of window 1
    execute javascript "document.title"
  end tell
end tell
```

### Click a Button by Text Content

```applescript
tell application "Google Chrome"
  tell active tab of window 1
    execute javascript "
      const btns = document.querySelectorAll('button');
      const btn = Array.from(btns).find(b =>
        b.textContent.includes('目標文字'));
      if (btn) { btn.click(); 'clicked'; }
      else { 'not found'; }
    "
  end tell
end tell
```

### Reveal Hover-Only Buttons

Some buttons are hidden until hover. Dispatch `mouseenter` on the parent first:

```applescript
execute javascript "
  const btn = /* find the button */;
  btn.closest('[class]').dispatchEvent(
    new MouseEvent('mouseenter', {bubbles: true}));
  setTimeout(() => btn.click(), 500);
"
```

### Combined: Browser Action → Native Dialog

This is the core pattern for downloading files:

1. **Chrome AppleScript** triggers the download button (JS click)
2. **System Events AppleScript** handles the native save dialog

These are two separate `osascript` calls — the second should wait for the
dialog to appear after the first completes.
