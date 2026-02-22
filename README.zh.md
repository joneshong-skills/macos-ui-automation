[English](README.md) | [繁體中文](README.zh.md)

# macos-ui-automation

使用 AppleScript System Events 自動化原生 macOS 對話框（儲存、開啟、警告、列印）及瀏覽器觸發的 UI。

## 快速開始

**前置需求：**
- 無障礙權限：在「系統設定 > 隱私權與安全性 > 無障礙功能」中授予終端機/應用程式存取權
- 預設儲存目錄：`mkdir -p ~/Downloads/claude_code_skill/`

## 核心模式

所有 macOS UI 自動化使用兩層架構：

```
第 1 層：App AppleScript    → tell application "AppName"
         觸發動作
         ↓
第 2 層：System Events      → tell process "AppName"
         與 UI 互動
```

## 常見工作流程

### 儲存對話框

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

## 功能特色

- **原生對話框控制**：處理儲存、開啟、警告和列印對話框
- **瀏覽器整合**：透過 Chrome AppleScript 觸發動作，控制原生 UI
- **元素探索**：使用 `entire contents` 探索未知對話框
- **無障礙功能**：適用於終端機、VS Code 和其他具有權限的應用程式

## 對話框元素參考

| 元素 | 繁體中文 | English |
|---------|-------|---------|
| 檔案名稱欄位 | `text field "儲存為："` | `text field "Save As:"` |
| 位置彈出按鈕 | `pop up button "位置："` | `pop up button "Where:"` |
| 儲存按鈕 | `button "儲存"` | `button "Save"` |
| 取消按鈕 | `button "取消"` | `button "Cancel"` |

## 授權

作為 Skills Collection 的一部分包含在 Claude Code 中。
