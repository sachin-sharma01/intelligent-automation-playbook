# ⚡ Expressions in Power Automate

Expressions are the most powerful — and most confusing — part of Power Automate.  
This is a practical reference built from real usage.

---

## Syntax

```
@{expression}     ← inside a string field
@expression       ← standalone field
```

---

## Date & Time

### Get today's date (formatted)
```
formatDateTime(utcNow(), 'yyyy MMM dd')
// Output: 2026 Apr 01
```

### Common format patterns
| Pattern | Output | Notes |
|---|---|---|
| `yyyy` | 2026 | Year |
| `MM` | 04 | Month number ← CAPITAL M |
| `MMM` | Apr | Month short name ← CAPITAL M |
| `MMMM` | April | Month full name ← CAPITAL M |
| `dd` | 01 | Day |
| `HH` | 14 | Hour (24hr) |
| `mm` | 30 | Minutes ← small m |
| `ss` | 45 | Seconds |

### ⚠️ Classic gotcha — MM vs mm
```
// WRONG — this gives you minutes, not month!
formatDateTime(utcNow(), 'yyyy mmm dd')

// CORRECT — capital M = month
formatDateTime(utcNow(), 'yyyy MMM dd')
```
> Rule: **Capital M = Month, small m = Minutes**

### Action timeout — ISO 8601 format
Power Automate uses ISO 8601 for durations (e.g. in action timeout fields):
```
PT30M   → 30 minutes
PT1H    → 1 hour
P1D     → 1 day
PT1H30M → 1 hour 30 minutes
P2DT4H  → 2 days 4 hours
```

---

## String Operations

### Concatenate
```
concat('Hello ', triggerBody()?['name'])
// Output: Hello Sachin
```

### Check if string contains value
```
contains(triggerBody()?['subject'], 'urgent')
// Output: true/false
```

### Convert to uppercase/lowercase
```
toUpper('hello')   // HELLO
toLower('HELLO')   // hello
```

---

## Conditionals

### Inline if
```
if(equals(triggerBody()?['status'], 'active'), 'Yes', 'No')
```

### Check if empty
```
empty(triggerBody()?['email'])
// Output: true/false
```

---

## Working with JSON / Dynamic Content

### Access a field from HTTP response
```
body('HTTP_action_name')?['fieldName']
```

### Access nested field
```
body('HTTP_action_name')?['user']?['email']
```

### Parse and access array item
```
body('Parse_JSON')?[0]?['title']
// Gets first item's title
```

---

## Numbers

### Convert string to integer
```
int(triggerBody()?['amount'])
```

### Basic math
```
add(10, 5)       // 15
sub(10, 5)       // 5
mul(10, 5)       // 50
div(10, 5)       // 2
```

---

## Testing Expressions

**Fastest way:** Add a **Compose** action to your flow → put the expression in Inputs → run the flow → check the output in run history.

```
// In Compose action inputs:
formatDateTime(utcNow(), 'yyyy MMM dd')

// Run flow → check Compose output → verify result
```

---

## Real Example — Dynamic Teams Card Message

```json
"text": "Good morning! Standup for @{formatDateTime(utcNow(), 'dddd dd MMMM yyyy')} 👇"
// Output: Good morning! Standup for Tuesday 01 April 2026 👇
```
