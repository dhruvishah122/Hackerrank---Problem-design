# Problem Description

A large-scale system ingests millions of internal events per minute. To prevent overload, each category of event must be regulated using event throttling.

Implement an in-memory service that determines whether incoming events should be accepted or throttled, based on dynamically configurable rate-limit rules.

Each event consists of:

- `eventType` (String)
- `timestamp` (epoch milliseconds)

Each `eventType` can define a rule:

> Allow at most X events within Y milliseconds.

Your service must track event activity over time and ensure outdated events do not influence future evaluations.

# Functional Requirements

## 1. Configure Rate Limit

**POST /limits**  
Request:

```json
{
  "eventType": "email-send",
  "maxEvents": 100,
  "perMillis": 60000
}
````

**Rules:**

* `eventType`: required, unique, non-empty
* `maxEvents`: integer > 0
* `perMillis`: integer > 0

**Expected Status Codes:**

* `200` → Rule created successfully
* `400` → Invalid input (missing/empty eventType, maxEvents ≤ 0, perMillis ≤ 0)
* `409` → Duplicate configuration (eventType already exists)

## 2. Evaluate Event

**POST /events**
Request:

```json
{
  "eventType": "email-send",
  "timestamp": 1711212150000
}
```

Response:

```json
{ "allowed": true }
```

or

```json
{ "allowed": false }
```

**Rules:**

* `timestamp` must be > 0
* Evaluation must consider only the applicable time window
* Outdated event timestamps must not affect decisions

**Expected Status Codes:**

* `200 `→ Event evaluated successfully (either allowed or blocked)
* `400` → Invalid input (timestamp ≤ 0)
* `404` → eventType not configured

## 3. List Rules

**GET /limits**
Returns all the rules.

**Expected Status Codes:**

* `200` → Returns all configured rules

## 4. Delete Rule

**DELETE /limits/{eventType}**
Deletes the rule and clears associated event history.

**Expected Status Codes:**

* `200` → Rule deleted successfully
* `404` → Rule not found

# Constraints

* Max 100 event types
* Max 10,000 stored timestamps per eventType
* Case-sensitive `eventType`
* Entire solution must be in-memory
* Must maintain performance under high event frequency

# Sample Behavior

**Configuration:**

```
eventType = "sync"
maxEvents = 3
perMillis = 10000
```

**Events:**

```
t=1000  → allowed
t=2000  → allowed
t=3000  → allowed
t=4000  → allowed=false (limit exceeded)
t=12000 → allowed (old events expired)
```
