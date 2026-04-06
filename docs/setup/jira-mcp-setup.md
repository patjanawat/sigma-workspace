# Jira MCP Setup สำหรับ Claude Code

ให้ Claude Code เข้าถึง Jira tickets ของ manaosoftware ได้โดยตรง

---

## Config ที่ใช้

**File:** `.mcp.json` (ที่ root ของ workspace)

```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@sooperset/mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://manaosoftware.atlassian.net",
        "JIRA_EMAIL": "pa@manaosoftware.com",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    }
  }
}
```

Token อ่านจาก environment variable `JIRA_API_TOKEN` — ไม่เก็บ hardcode ในไฟล์

---

## วิธี Setup ครั้งแรก (หรือเมื่อ Token หมดอายุ)

### 1. สร้าง API Token

ไปที่ https://id.atlassian.com/manage-profile/security/api-tokens → **Create API token**

### 2. Set Environment Variable (Windows)

เปิด PowerShell แล้วรัน:

```powershell
[System.Environment]::SetEnvironmentVariable("JIRA_API_TOKEN", "<token>", "User")
```

### 3. สร้าง `.mcp.json` ไฟล์

สร้างไฟล์ `.mcp.json` ที่ workspace root (หรือ copy config ข้างบน) โดยใส่ config ที่แสดงไว้ในส่วน "Config ที่ใช้"

### 4. Restart VS Code

ต้อง restart เพื่อให้ Claude Code โหลด `.mcp.json` และ env var ใหม่

---

## ทดสอบว่าใช้งานได้

ถาม Claude ว่า: `สรุป SIGMA-XXXX` หรือ `ดึง SIGMA-1011` — ถ้าดึงข้อมูลได้แสดงว่า setup สำเร็จ

---

## หมายเหตุ

- `.mcp.json` อยู่ใน `.gitignore` — ปลอดภัย ไม่ถูก commit
- Token มีอายุตามที่ Atlassian กำหนด ถ้าใช้ไม่ได้ให้สร้าง token ใหม่แล้วทำ Step 1–4 ซ้ำ
- ⚠️ ไม่ควรใส่ MCP servers config ใน `settings.local.json` — ใช้ `.mcp.json` แทน
