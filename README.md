# Salapi — Personal Finance Automation

A set of n8n workflows that automate personal finance tracking via Telegram and Google Sheets.

## Workflows

### 1. Finance Logger

Logs expenses and income by sending a simple text message to a Telegram bot.

**How it works:**

```
You text Telegram: "gastos 350 Grab" or "gainz 21000 DOST"
  → n8n Telegram Trigger receives the message
  → Parse & Categorize extracts type, amount, title, and assigns a category
  → Send Confirmation replies: "Logged: P350 - Grab (Transport)"
  → Route Type splits:
      ├── "gastos" → Append Expenses (Google Sheets > Expenses tab)
      └── "gainz"  → Append Income (Google Sheets > GAINZ tab)
```

**Supported formats:**
- `gastos <amount> <description>` — log an expense (e.g., `gastos 350 Grab`)
- `gains <amount> <source>` — log income (e.g., `gains 21000 DOST`)

**Auto-categorization** for expenses based on keywords:
| Category | Keywords |
|---|---|
| Transport | grab, taxi, fare, gas, jeep, mrt, lrt, tricycle |
| Food | eat, lunch, dinner, coffee, coke, burger, rice |
| Clothes | shirt, pants, shoes, bershka, jeans, jacket |
| Subscriptions | spotify, netflix, canva, premium, youtube |
| Bills & Utilities | bill, electric, water, internet, rent, dost |
| Entertainment | movie, game, concert, cinema |
| Health & Fitness | medicine, doctor, hospital, gym |
| Other | everything else |

**Nodes** (6 total):
```
Telegram Trigger → Parse & Categorize (Code) → Send Confirmation (Telegram)
                                              → Keep Parsed Data (NoOp) → Route Type (IF)
                                                                           ├── Append Expenses (Google Sheets)
                                                                           └── Append Income (Google Sheets)
```

---

### 2. Summary Report (Monthly)

Sends a beautifully formatted HTML email with your monthly financial summary.

**How it works:**

```
Schedule Trigger (1st of month, 9AM)
  → Get Rows in Expenses Sheet (reads all rows)
  → Get Rows in Income Sheet (reads all rows)
  → Merge Data (combines both into one set)
  → Email Body Creation (filters to previous month, calculates totals/categories, generates HTML)
  → Email Monthly Finance Report (Gmail to yoroleo10@gmail.com)
```

**Email includes:**
- Total income, total expenses, balance
- Breakdown by category with amounts
- Top spending category badge
- Transaction count

**Nodes** (6 total):
```
Schedule Trigger (monthly)
  ├── Get Rows in Expenses Sheet (Google Sheets)
  ├── Get Rows in Income Sheet (Google Sheets)
  └── Merge Data → Email Body Creation (Code) → Email Monthly Finance Report (Gmail)
```

---

## Setup Guide

### Prerequisites
- [n8n](https://n8n.io) running locally (Docker)
- Telegram account
- Google account (for Sheets + Gmail)
- ngrok (for Telegram webhooks)

### Step 1: Import Workflows

1. Open your n8n editor (e.g., `http://localhost:5678`)
2. Go to **Workflows** → **+ Add workflow**
3. Click the **three dots (...)** → **Import from File**
4. Select `Finance Logger.json` and `Summary Report.json`

### Step 2: Set Up Credentials

**Telegram Bot:**
1. Open Telegram → search `@BotFather` → `/newbot` → name it (e.g., "Salapi Finance Bot")
2. Copy the API token
3. In n8n: **Settings → Credentials → + Add credential → Telegram API**
4. Paste token → **Save**

**Google Sheets & Gmail (OAuth2):**
1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a project → Enable **Google Sheets API** + **Google Drive API** + **Gmail API**
3. **Credentials → + Create Credentials → OAuth Client ID → Web Application**
4. Add Authorized Redirect URI: `https://your-ngrok-url.ngrok-free.dev/rest/oauth2-credential/callback`
5. Copy **Client ID** and **Client Secret**
6. In n8n: Create **Google Sheets OAuth2 API** and **Gmail OAuth2 API** credentials using those values
7. Sign in with Google when prompted

### Step 3: Activate Workflows

1. Open each workflow
2. Click **Active** toggle at the top
3. For Finance Logger: send a test message to your Telegram bot
4. For Summary Report: test with **Execute Workflow** button first, then let it run on schedule

### Step 4: Google Sheet Structure

The workflows expect a Google Sheet with at least two tabs:

**Expenses tab** (columns): DATE, TITLE, AMOUNT, CATEGORY, TOTAL, GAINZ

**GAINZ tab** (columns): DATE, AMOUNT, SOURCE, TOTAL

The Sheet ID is hardcoded in the nodes — update it if you use a different sheet.
