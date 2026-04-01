# 😅 Power Automate Gotchas

Things that will waste your time if you don't know them.  
Learned the hard way.

---

## 1. MM vs mm — Month vs Minutes

```
// WRONG
formatDateTime(utcNow(), 'yyyy mmm dd')  // gives minutes!

// CORRECT  
formatDateTime(utcNow(), 'yyyy MMM dd')  // gives month name
```
**Rule: Capital M = Month. Small m = Minutes. Always.**

---

## 2. Action Timeout uses ISO 8601 — not plain numbers

You can't just type `30` for 30 minutes. You must use ISO 8601 duration format:

```
PT30M   → 30 minutes
PT1H    → 1 hour
P1D     → 1 day
P1DT4H  → 1 day and 4 hours
```

---

## 3. Parallel Branches cannot be disabled

There is no built-in way to disable a parallel branch for testing.  
**Workaround:** Use Static Results (Testing tab) on actions you don't want to execute.

---

## 4. "Configure run after" is hidden in Settings tab

New users look everywhere for this. It's not on the action card itself.  
**Location:** Click action → **Settings tab** → expand **Run after**

---

## 5. Dynamic content vs Expression — two different tabs

When you click into a field:
- **Dynamic content tab** = outputs from previous actions (click to insert)
- **Expression tab** = write functions like `formatDateTime`, `concat`, `if`

Many beginners miss the Expression tab entirely.

---

## 6. Premium connectors need a premium license

Standard connectors (Teams, Outlook, SharePoint) = free with most M365 licenses.  
Premium connectors (HTTP, Dataverse, SAP) = need **Power Automate Premium** license.

**HTTP connector is premium** — this means calling external APIs directly requires a paid license.  
Enterprise workaround: Use **Azure Function** as a middleware instead.

---

## 7. Apply to Each runs sequentially by default

Looping through 100 items one by one = slow.  
**Fix:** Apply to Each → Settings → **Concurrency Control ON** → set degree of parallelism (max 50)

⚠️ Be careful with concurrency if actions inside the loop write to the same resource.

---

## 8. Flow run history only keeps 28 days

Don't rely on Power Automate run history for long-term audit trails.  
**Fix:** Log important events to SharePoint list or Dataverse table inside your flow.

---

## 9. Adaptive Card responses expire

"Post adaptive card and wait for response" has a default timeout.  
If the user doesn't respond in time → flow times out → configure "Has timed out" in Run after.  
**Set explicit timeout** using ISO 8601 in action timeout field: `PT4H` for 4 hours.

---

## 10. Solution vs Non-solution flows

Flows built outside a Solution = harder to move between environments (Dev → Test → Prod).  
**Best practice:** Always build flows inside a **Solution** from the start.  
Migrating later is painful.
