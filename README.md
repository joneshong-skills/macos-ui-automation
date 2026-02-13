# macOS UI Automation

Automate native macOS dialogs (save, open, alert, print) and browser-triggered UI via AppleScript System Events. Control save/open/alert dialogs, file pickers, and other native macOS UI elements.

## Quick Start

**Prerequisites:**
- Accessibility permission: Grant Terminal/app access in System Settings > Privacy & Security > Accessibility
- Default save directory: `mkdir -p ~/Downloads/claude_code_skill/`

## Core Pattern

All macOS UI automation uses two layers:

```
Layer 1: App AppleScript    → tell application "AppName"
         triggers actions
         ↓
Layer 2: System Events      → tell process "AppName"
         interacts with UI
```

## Common Workflows

### Save Dialog

```bash
osascript -e '
tell application "System Events"
  tell process "Google Chrome"
    tell splitter group 1 of sheet 1 of window 1
      set value of text field "儲存為：" to "output.jpg"
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

### Browser Integration

Trigger download via Chrome JavaScript, then handle save dialog:
```bash
osascript -e '
tell application "Google Chrome"
  tell active tab of window 1
    execute javascript "document.querySelector(\"button\").click()"
  end tell
end tell'
```

### Dialog Detection

Poll for native dialogs:
```bash
osascript -e '
tell application "System Events"
  tell process "Google Chrome"
    exists sheet 1 of window 1
  end tell
end tell'
```

## Key Features

- **Native Dialog Control**: Handle save, open, alert, and print dialogs
- **Browser Integration**: Trigger actions via Chrome AppleScript, control native UI
- **Element Discovery**: Use `entire contents` to explore unknown dialogs
- **Accessibility**: Works with Terminal, VS Code, and other apps with permission
- **Error Handling**: Graceful fallbacks and proper delay handling

## Dialog Element Reference

| Element | zh-TW | English |
|---------|-------|---------|
| Filename field | `text field "儲存為："` | `text field "Save As:"` |
| Location popup | `pop up button "位置："` | `pop up button "Where:"` |
| Save button | `button "儲存"` | `button "Save"` |
| Cancel button | `button "取消"` | `button "Cancel"` |

## Use Cases

- Automate file downloads to specific locations
- Handle native dialogs programmatically
- Control macOS UI without user interaction
- Combine with browser automation for complete workflows
- Integrate with CI/CD pipelines

## Common Locations

| zh-TW | English | Path |
|-------|---------|------|
| 桌面 | Desktop | `~/Desktop` |
| 下載項目 | Downloads | `~/Downloads` |
| 文件 | Documents | `~/Documents` |

## License

Included in Claude Code as part of the Skills Collection.
