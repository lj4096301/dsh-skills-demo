---
name: feishu-card-build
description: >-
  生成一份能通过飞书校验的 interactive 消息卡片 JSON（官方 schema：header.template + markdown 元素 + action 按钮，禁止自创字段）。
  当要把子代理派发 / 完成台账、agent 归属视图推到飞书群，或任何飞书机器人卡片时使用。
---

# 飞书卡片生成

飞书卡片字段错一个就被拒或渲染错乱。本技能给一份可抄的骨架和推送方式。

## 何时用
- 每次派发批次结束，要推"主卡（收敛结论 + 台账）+ 至多 1 张补充卡"。
- 任何往飞书自定义机器人 POST 的卡片。

## 卡片骨架（照抄，只换文字）
```json
{
  "header": { "template": "blue", "title": { "tag": "plain_text", "content": "标题" } },
  "elements": [
    { "tag": "markdown", "content": "**字段**：值\n- 列表项" },
    { "tag": "action", "actions": [ { "tag": "button", "text": { "tag": "plain_text", "content": "打开" }, "type": "primary", "url": "https://..." } ] }
  ]
}
```

## 硬规则
- `header` 用 `template` + `title.tag=plain_text`；**不要**写 `theme` 或字符串 title。
- markdown 用 `{ "tag": "markdown", "content": "..." }`；**不要**嵌套 `text.content`。
- 按钮用 `action` / `actions` / `button.tag=button` / `text` / `type` / `url`；**不要**写 `button` / `label`。
- 卡片 v1 的 markdown **不保证渲染管道表格**，表格内容改写成编号列表。

## 推送
Python 必须用绝对路径（本机 `python` 不在 PATH）：
```
C:\Users\bumah\.workbuddy\binaries\python\versions\3.13.12\python.exe C:\Users\bumah\WorkBuddy\疑问\feishu_webhook_push.py <卡片.json>
```
- **必须**同时设 `PYTHONIOENCODING=utf-8`（stdout 默认 GBK，卡片含 emoji 会 UnicodeEncodeError）。
- webhook 走 `FEISHU_WEBHOOK` 环境变量；不存在时从 `C:\Users\bumah\.dsh\.env` 读 `FEISHU_WEBHOOK=` 一行再设入环境。
- 开签名校验时再加 `FEISHU_SECRET`。

## 数量规范
一次交付：1 张主卡（结论 + 台账）+ 至多 1 张补充卡（机制 / 细节）。不要为每个片段单独推卡刷屏；长细节写进文档，卡片只放结论与台账。
