# 🔧 n8n — Workflow Patterns & Best Practices

n8n is an open-source workflow automation tool.  
Self-hostable, API-first, and extremely powerful when combined with AI.

---

## Why n8n over other tools

| Feature | n8n | Power Automate |
|---|---|---|
| Self-hostable | ✅ | ❌ |
| Code nodes | ✅ Full JS/Python | ❌ Limited expressions |
| HTTP calls | ✅ Free, native | ⚠️ Premium connector |
| AI integration | ✅ Any API | ⚠️ Azure OpenAI / AI Builder |
| Cost | Free (self-hosted) | Per-user license |
| Enterprise governance | ❌ Manual | ✅ Built-in CoE/DLP |

**Rule of thumb:**
- Personal projects, startups, flexible pipelines → **n8n**
- Enterprise Microsoft environment → **Power Automate**

---

## Self-Hosting Setup

### Recommended: Hetzner VPS + Docker

```bash
# Minimum spec: CX21 (2 vCPU, 4GB RAM) — ~€5/month
# Install Docker, then:

docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### With persistent storage + auto-restart:
```bash
docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=yourpassword \
  n8nio/n8n
```

---

## Core Patterns

### Pattern 1: Webhook → Transform → Action

Most common pattern. Anything triggers it via HTTP POST.

```
Webhook trigger
    → Code node (transform/validate data)
        → HTTP Request (call external API)
            → IF node (route based on response)
                → Success: Notify Teams
                → Failure: Log to DB + alert
```

### Pattern 2: Scheduled Pipeline

```
Cron trigger (daily 8am)
    → HTTP Request (fetch data from source)
        → Code node (parse + clean data)
            → Loop over items
                → HTTP Request (process each item)
                    → Postgres/Supabase (save results)
```

### Pattern 3: AI-Powered Extraction

```
Trigger (webhook or schedule)
    → Download file (PDF/HTML)
        → Extract text
            → HTTP Request to Claude/OpenAI API
                → Code node (parse JSON response)
                    → Save structured data to DB
                        → Notify stakeholders
```

---

## Error Handling in n8n

### Basic — Error Trigger node
Add an "Error Trigger" workflow — runs automatically when any other workflow fails.

```
Error Trigger
    → Send Teams/Slack message
        → "Workflow {{$workflow.name}} failed: {{$json.error.message}}"
```

### Per-node — Continue on Fail
Node settings → **"Continue on Fail"** ON  
Flow continues even if this node errors — you handle it downstream.

### Retry on Fail
Node settings → **"Retry on Fail"** → set max tries + wait between retries  
Good for: API rate limits, temporary network issues.

---

## OAuth 2.0 in n8n

n8n handles OAuth 2.0 natively via Credentials.

```
Credentials → New → OAuth2 API
    → Authorization URL
    → Token URL  
    → Client ID + Secret
    → Scope
```

n8n automatically refreshes tokens — you don't manage it manually.

---

## Code Node — Real Examples

### Parse and filter JSON array
```javascript
const items = $input.all();
return items.filter(item => 
  item.json.impact_level === 'high' || 
  item.json.impact_level === 'medium'
);
```

### Call Claude API
```javascript
const response = await $http.request({
  method: 'POST',
  url: 'https://api.anthropic.com/v1/messages',
  headers: {
    'x-api-key': $env.ANTHROPIC_API_KEY,
    'anthropic-version': '2023-06-01',
    'content-type': 'application/json'
  },
  body: {
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1000,
    messages: [{ role: 'user', content: $json.text }]
  }
});

return [{ json: response }];
```

---

## Production Checklist

- [ ] Error trigger workflow set up
- [ ] Credentials stored in n8n (not hardcoded)
- [ ] Sensitive env vars in `.env` file
- [ ] Retry on fail enabled for HTTP calls
- [ ] Webhook URLs secured (basic auth or token)
- [ ] n8n behind reverse proxy (nginx) with HTTPS
- [ ] Regular backups of `~/.n8n` folder
