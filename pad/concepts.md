# 🖥️ Power Automate Desktop (PAD)

Desktop automation — for when there's no API.

---

## What is PAD?

PAD automates what a human does manually on a Windows computer:
- Click buttons
- Read/write Excel
- Fill forms in legacy apps
- Scrape data from websites
- Move files and folders

**Key difference from Cloud Flows:** PAD works on the **screen** — no API needed.

---

## When to Use PAD vs Cloud Flows

| Situation | Use |
|---|---|
| Modern app with REST API | ✅ Cloud Flow |
| Legacy desktop app (SAP, Oracle, old ERP) | ✅ PAD |
| Excel manipulation | ✅ PAD |
| Web scraping (no API) | ✅ PAD |
| Email/Teams/SharePoint | ✅ Cloud Flow |
| Scheduled report generation | PAD + Cloud Flow |
| Cross-system data sync (APIs available) | ✅ Cloud Flow |

**Rule:** Always prefer API-first (Cloud Flows) when possible.  
PAD is a workaround for systems that have no API — not a first choice.

---

## Core Concepts

### Actions
PAD has 400+ built-in actions organized in categories:
- **Browser automation** — launch Chrome/Edge, click, extract data
- **Excel** — open, read, write, close
- **File/Folder** — create, move, delete, rename
- **UI Automation** — interact with Windows desktop apps
- **Text** — manipulate, parse, extract
- **Variables** — store and pass data between actions

### Variables
All variables are wrapped in `%`:
```
%ExcelData%     → stores a table
%CurrentRow%    → current row in a loop
%WebPageText%   → extracted text from browser
%FilePath%      → path to a file
```

### Control Flow
- `If / Else` — conditional branching
- `Loop` — repeat N times
- `For each` — iterate over list or table rows
- `Loop condition` — loop while condition is true

### Subflows
Like functions — reusable blocks of actions.  
Keep main flow clean, put complex logic in subflows.

---

## Attended vs Unattended

| | Attended | Unattended |
|---|---|---|
| Requires human at PC | ✅ Yes | ❌ No |
| Runs on schedule | ❌ | ✅ |
| License | Lower cost | Higher cost |
| Use case | Assist human in real-time | Fully automated background tasks |

---

## Error Handling in PAD

### On Block Error
Wraps a group of actions — like try/catch:
```
On block error
    → [actions that might fail]
End
→ Handle error here
```

### Retry
Configure any action to retry on failure:
- Max retries
- Wait between retries

---

## PAD vs n8n vs Cloud Flows — Full Comparison

| Feature | PAD | Cloud Flows | n8n |
|---|---|---|---|
| UI/screen automation | ✅ | ❌ | ❌ |
| API integration | ⚠️ Limited | ✅ | ✅ |
| AI integration | ⚠️ Via Cloud Flow | ✅ AI Builder | ✅ Any API |
| Self-hostable | ❌ | ❌ | ✅ |
| Enterprise governance | ✅ | ✅ | Manual |
| Best for | Legacy systems | Microsoft ecosystem | Flexible pipelines |

---

## Interview Answer — "How do you decide between PAD and Cloud Flows?"

> "My default is always Cloud Flows — they're API-first, scalable, and maintainable. PAD is the right tool when you're dealing with legacy systems that have no API — like old SAP instances or desktop applications from the 90s. In modern Microsoft environments, most integrations have connectors or REST APIs, so PAD is rarely the first choice. That said, PAD is powerful for Excel-heavy processes and screen automation when there's genuinely no other option."
