---
name: website-automation
description: |
  General-purpose browser automation via playwright-cli. This skill should be used when the user
  wants to automate any website operation that requires real browser interaction — logging in
  (QR code scan, credentials, SSO, enterprise WeChat), searching, filling forms, clicking through
  multi-page flows, extracting data from JavaScript-rendered pages, or taking screenshots.
  Triggers include: "帮我自动登录 XX 网站搜索 XX", "帮我在 XX 平台操作", "截图这个网页",
  "帮我填这个表单", "自动化操作 XX 网站", "用浏览器打开 XX", or any request to interact with
  a website beyond simple content fetching.
agent_created: true
---

# Website Automation

## Overview

Use `playwright-cli` for full browser automation on any website. It launches a real Chromium browser supporting JavaScript rendering, login flows (QR scan / credentials / SSO), search, form filling, data extraction, and screenshots. Cookies, localStorage, and login state persist across commands within a session.

## Prerequisites

- Node.js 18+ (use managed binary: `/Users/sophia-xp/.workbuddy/binaries/node/versions/22.12.0/bin/node`)
- npm available; `playwright-cli` installed globally: `npm install -g @playwright/cli`
- System Chrome is used (no separate download needed if Chrome is already installed)

Verify installation and install if needed:

```bash
# Check
PATH="/Users/sophia-xp/.npm-global/bin:$PATH" playwright-cli install-browser chrome

# If not installed
npm install -g @playwright/cli@latest
```

## Core Workflow

Every automation task follows this pattern. Run all commands with `PATH="/Users/sophia-xp/.npm-global/bin:$PATH"` prefixed.

### 1. Open target URL

```bash
playwright-cli open --browser=chrome <url>
# Or: playwright-cli open <url> --browser=chrome
```

This starts the browser daemon. All subsequent commands reuse the same daemon and preserve login state.

### 2. Inspect the page

```bash
playwright-cli snapshot --filename=<name>.yaml
cat <name>.yaml
```

The snapshot lists interactive elements with `[ref=...]` identifiers. Use these refs for clicking, typing, and filling. The ref format may include a session prefix like `f5e39` — always use the full ref as shown in the latest snapshot.

**Common element patterns in snapshots:**
- `textbox "搜索" [ref=xxx]` — search input
- `button "登录" [ref=xxx]` — buttons
- `link "微信" [ref=xxx]` — links
- `checkbox "同意协议" [ref=xxx]` — checkboxes
- `textbox "请输入手机号" [ref=xxx]` — input fields

### 3. Interact with the page

```bash
# Click an element
playwright-cli click <ref>

# Fill a textbox (clears existing content, types new)
playwright-cli fill <ref> "text to enter"

# Press keyboard keys
playwright-cli press Enter

# Type text (appends to existing content, for focused elements only)
playwright-cli type "text"

# Check/uncheck
playwright-cli check <ref>
playwright-cli uncheck <ref>
```

### 4. Read results

```bash
# Take snapshot to read structured content
playwright-cli snapshot --filename=results.yaml
cat results.yaml

# Take screenshot for visual inspection
playwright-cli screenshot --filename=results.png
```

### 5. Close the browser

```bash
playwright-cli close
```

**Always close the browser when the task is complete.** Do not leave the daemon running.

## Login Patterns

### QR Code Login (Enterprise WeChat / WeChat)

1. Open login page and snapshot to find the login method link
2. Click the link (e.g., `playwright-cli click <ref>` for "企业微信")
3. Snapshot to confirm redirect to QR code page
4. Screenshot and present to user: `playwright-cli screenshot --filename=qr.png`
5. Wait for user to scan, then snapshot again to confirm login success (page title/URL will change)

### Phone + Verification Code

1. Fill phone number: `playwright-cli fill <ref> "13800138000"`
2. Click "发送验证码": `playwright-cli click <ref>`
3. Ask user for the verification code
4. Fill code: `playwright-cli fill <ref> "123456"`
5. Check agreement checkbox: `playwright-cli check <ref>`
6. Click login: `playwright-cli click <ref>`

### Username + Password (SSO / Custom)

1. Fill username: `playwright-cli fill <ref> "username"`
2. Fill password: `playwright-cli fill <ref> "password"`
3. Click submit: `playwright-cli click <ref>`

## Search Patterns

1. Locate search input in snapshot: `textbox "搜索" [ref=xxx]` or similar
2. Fill the keyword: `playwright-cli fill <ref> "关键词"`
3. Submit: `playwright-cli press Enter` or click the search button
4. Read results with snapshot

## Navigation

```bash
playwright-cli goto <url>        # Navigate to new URL in current tab
playwright-cli go-back           # Browser back
playwright-cli go-forward        # Browser forward
playwright-cli reload            # Refresh page
```

## Multi-tab

```bash
playwright-cli tab-new <url>     # Open new tab
playwright-cli tab-list          # List all tabs
playwright-cli tab-select <n>    # Switch to tab N
playwright-cli tab-close <n>     # Close tab N
```

## Debugging

```bash
# View console logs
cat .playwright-cli/console-*.log

# Evaluate JavaScript
playwright-cli eval "document.title"

# Full-page screenshot
playwright-cli screenshot --filename=full.png
```

## Important Notes

- **Refs change after navigation.** Always take a fresh snapshot after any page transition before using refs.
- **Snapshot files may be empty.** If `cat` returns empty, use `screenshot` instead or try `playwright-cli eval "document.body.innerText"`.
- **Persistent session.** Login state (cookies, localStorage) persists until `playwright-cli close`. Navigate between pages without re-logging in.
- **Close on finish.** Always run `playwright-cli close` when done — even on error paths. Leaving the daemon leaks resources.
- **Working directory.** Run commands from the project root so snapshot/screenshot files land in an accessible location.
