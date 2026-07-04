# Website Automation

WorkBuddy 技能 — 通用网站浏览器自动化，基于 playwright-cli。

## 功能

- 🔑 **自动登录**：支持扫码登录（微信/企业微信）、手机验证码、用户名密码、SSO
- 🔍 **自动搜索**：定位搜索框 → 输入关键词 → 执行搜索 → 提取结果
- 📝 **表单填写**：点击、输入、勾选、下拉选择
- 📸 **截图提取**：可视化验证和内容抓取

## 安装

将此技能放入 WorkBuddy 的 `~/.workbuddy/skills/` 目录下即可自动加载。

依赖：`npm install -g @playwright/cli`

## 使用示例

对 WorkBuddy 说：

> "帮我在 XX 网站自动搜索 YY"

或

> "帮我自动登录 XX 平台，然后截图"

WorkBuddy 会自动加载此技能并执行浏览器自动化操作。

## 技能文件

- `SKILL.md` — 完整的技能指令和工作流
