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
* Duplicate configuration → 409
* Invalid input → 400

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

* If `eventType` not configured → 404
* `timestamp` must be > 0
* Evaluation must consider only the applicable time window
* Outdated event timestamps must not affect decisions

## 3. List Rules

**GET /limits**
Returns all the rules.

## 4. Delete Rule

**DELETE /limits/{eventType}**
Deletes the rule and clears associated event history.

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
