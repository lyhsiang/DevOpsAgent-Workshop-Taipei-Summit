# DevOps Agent Workshop — Taipei Summit

Hands-on 教材節錄，取自 AWS workshop **"Getting Started with AWS DevOps Agent: A Hands-On Walkthrough"**，供 Summit 當天參加 workshop 的同事使用。

> 原始 workshop：<https://catalog.us-east-1.prod.workshops.aws/workshops/d58f8530-da8b-42c9-8223-44a0c59acc44/en-US>

## 模組 Modules

| # | 模組 | 內容 |
|---|------|------|
| 1 | [Chat & Artifacts](modules/01-chat-and-artifacts.md) | 用自然語言查詢基礎設施、產生 Artifact 報告、Chat vs Investigation 比較 |
| 2 | [Skills](modules/02-skills.md) | 上傳 Skill 教 agent 專業知識、SKILL.md 結構、有/無 skill 的調查結果比較 |
| 3 | [Webhook Trigger](modules/03-webhook-trigger.md) | 產生 webhook 憑證、用 curl 觸發調查、production 架構 |
| 4 | [Incident Triage](modules/04-incident-triage.md) | 多事件關聯（linked / proceed）、triage 運作原理、解除關聯 |

## 資料夾結構

```
DevOpsAgent-Workshop-Taipei-Summit/
├── README.md
├── modules/
│   ├── 01-chat-and-artifacts.md
│   ├── 02-skills.md
│   ├── 03-webhook-trigger.md
│   └── 04-incident-triage.md
└── images/
    ├── chatalarm.png
    ├── artifact.png
    ├── skills.png
    └── triagesave.png
```

## 待補 / 需人工確認

擷取過程中有兩處被系統的隱私遮罩改掉，已在 `03-webhook-trigger.md` 標上明顯佔位符，請對照原頁面補上：

1. **`SECRET=` 那行**（Step 2）— 格式應與上一行 `WEBHOOK_URL="..."` 相同。
2. **第二種驗證方法名稱**（How this works in production 段）— HMAC 之外、供 Splunk/Datadog/New Relic/ServiceNow/Slack 用的那一種。

## 備註

- 程式碼區塊已移除網站的行號，可直接複製貼上。
- 內容為節錄（Chat & Artifacts / Skills / Webhook Trigger / Incident Triage 四個模組），非完整 workshop。
