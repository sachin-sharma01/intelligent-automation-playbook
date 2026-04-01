# 🛡️ Error Handling in Power Automate

Production flows must handle failures gracefully.  
This is what separates a junior automation builder from a senior one.

---

## The 4 Run States

Every action in Power Automate has 4 possible outcomes:

| State | Icon | Meaning |
|---|---|---|
| **Is successful** | ✅ | Action completed normally |
| **Has failed** | ❌ | Action threw an error |
| **Has timed out** | ⏰ | Action exceeded its timeout duration |
| **Is skipped** | ⏭️ | Previous step failed so this was skipped |

By default — flows only continue on **success**. You must explicitly handle the rest.

---

## Configure Run After

"Configure run after" lets you control WHEN an action runs based on the previous action's state.

### How to access:
Action → **Settings tab** → **Run after**

### Use case — Alert when Teams card times out:

```
Post Adaptive Card → wait for response
    ↓ (timed out OR failed)
Post Teams alert: "⚠️ [Name] did not respond to standup"
```

**On the alert action:**
- ✅ Has timed out
- ✅ Has failed
- ❌ Is successful (uncheck this!)

This means: **only send the alert if something went wrong.**

---

## Scope — Try/Catch Pattern

Scope groups multiple actions together so you can handle any failure with one error handler.

```
[Scope: "Try"]
    → Step 1
    → Step 2
    → Step 3
        ↓ (if ANY step inside Scope fails)
[Post Teams Alert] ← Configure run after: "Has failed"
```

### Why use Scope instead of individual run-after:
- Cleaner flow — one error handler for many steps
- Easier to maintain
- Maps directly to try/catch in code

---

## Retry Policy

Every action has a built-in retry policy for transient failures.

**Location:** Action → Settings → **Networking → Retry policy**

| Option | When to use |
|---|---|
| Default | Most cases — exponential backoff |
| None | When you handle retries yourself |
| Fixed interval | Predictable retry timing |
| Exponential interval | API rate limits |

**Real use case:** Teams API temporarily down → flow retries automatically 4 times → usually recovers without failure.

---

## Static Results — Testing Without Real Execution

Static Results lets you mock an action's output during testing.

**Location:** Action → **Testing tab** → Static result ON

| Mock status | Use case |
|---|---|
| Succeeded | Test happy path |
| Failed | Test your error handler |
| TimedOut | Test timeout branch |

### ⚠️ Important use case:
Testing flows with **parallel branches** that send real messages to real people.  
Turn Static Results ON → flow runs fully → no real messages sent → error branches testable.

---

## Terminate Action

Force stop a flow with a specific status.

```
Terminate
  Status: Failed
  Message: "Invoice amount exceeded threshold — manual review required"
```

Use when a business rule is violated and continuing would cause harm.

---

## Real Pattern — Production Error Handling

```
[Scope: "Main Process"]
    → Get items from SharePoint
    → For each item
        → Call external API
        → Update SharePoint

[Post to Teams] ← run after Scope "Has failed"
    Message: "⚠️ Flow failed at @{utcNow()} — check run history"

[Send Email to Admin] ← run after Scope "Has failed"
    Subject: "Action required: Flow failure detected"
```

This pattern:
- Covers all failures in one place ✅
- Alerts the right people ✅
- Includes timestamp for debugging ✅
- Doesn't require manual monitoring ✅
