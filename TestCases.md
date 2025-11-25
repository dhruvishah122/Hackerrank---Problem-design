## 1. Functional Testcases

### 1.1 Under limit → allowed
**Configure Limit:**  
`POST /limits`

**Body:**
```json
{
  "eventType": "email-send",
  "maxEvents": 3,
  "perMillis": 10000
}
````

**Events:**
`POST /events`

```json
{"eventType": "email-send", "timestamp": 1000}
```

`POST /events`

```json
{"eventType": "email-send", "timestamp": 2000}
```

**Expected Output:**

```json
{"allowed": true}
{"allowed": true}
```

---

### 1.2 Exceed limit → not allowed

**Events:**
`POST /events`

```json
{"eventType": "email-send", "timestamp": 3000}
```

`POST /events`

```json
{"eventType": "email-send", "timestamp": 4000}
```

**Expected Output:**

```json
{"allowed": true}
{"allowed": false}
```

---

### 1.3 After waiting enough time → allowed

`POST /events`

```json
{"eventType": "email-send", "timestamp": 12000}
```

**Expected Output:**

```json
{"allowed": true}
```

---

### 1.4 Evaluating unknown eventType → 404

`POST /events`

```json
{"eventType": "sms-send", "timestamp": 15000}
```

**Expected Output:**
HTTP 404 Not Found

---

### 1.5 Delete rule → next evaluation returns 404

`DELETE /limits/email-send`

`POST /events`

```json
{"eventType": "email-send", "timestamp": 16000}
```

**Expected Output:**
HTTP 404 Not Found

---

### 1.6 Listing rules shows all configured limits

`GET /limits`

**Expected Output:**

```json
[
  {
    "eventType": "email-send",
    "maxEvents": 3,
    "perMillis": 10000
  }
]
```

---

## 2. Edge Testcases

### 2.1 maxEvents ≤ 0 → 400

`POST /limits`

```json
{
  "eventType": "email-send",
  "maxEvents": 0,
  "perMillis": 10000
}
```

**Expected Output:**
HTTP 400 Bad Request

---

### 2.2 perMillis ≤ 0 → 400

`POST /limits`

```json
{
  "eventType": "email-send",
  "maxEvents": 10,
  "perMillis": 0
}
```

**Expected Output:**
HTTP 400 Bad Request

---

### 2.3 eventType = "" → 400

`POST /limits`

```json
{
  "eventType": "",
  "maxEvents": 10,
  "perMillis": 10000
}
```

**Expected Output:**
HTTP 400 Bad Request

---

### 2.4 timestamp ≤ 0 → 400

`POST /events`

```json
{"eventType": "email-send", "timestamp": 0}
```

**Expected Output:**
HTTP 400 Bad Request

---

### 2.5 Case sensitivity check (Login vs login)

`POST /limits`

```json
{"eventType": "Login", "maxEvents": 1, "perMillis": 1000}
```

`POST /events`

```json
{"eventType": "login", "timestamp": 100}
```

**Expected Output:**
HTTP 404 Not Found

---

### 2.6 Rapid events at same timestamp

`POST /events`

```json
{"eventType": "email-send", "timestamp": 20000}
```

`POST /events`

```json
{"eventType": "email-send", "timestamp": 20000}
```

`POST /events`

```json
{"eventType": "email-send", "timestamp": 20000}
```

`POST /events`

```json
{"eventType": "email-send", "timestamp": 20000}
```

**Expected Output:**

```json
{"allowed": true}
{"allowed": true}
{"allowed": true}
{"allowed": false}
```

---

## 3. Performance Testcases

### 3.1 10,000 events in one short time window

`POST /limits`

```json
{"eventType": "high-load", "maxEvents": 10000, "perMillis": 1000}
```

**Send:** 10,000 events within 1 second (timestamps 1ms apart)

**Expected Output:**

* First 10,000 events → allowed
* 10,001st event → not allowed

---

### 3.2 100 eventTypes × 1,000 events each

`POST /limits`

```json
{"eventType": "type1", "maxEvents": 1000, "perMillis": 10000}
```

…repeat for type2 … type100 programmatically

**Send:** 1,000 events for each type

**Expected Output:**

* All events within limit → allowed
* 1,001st event for any type → not allowed

---

### 3.3 Continuous add/delete + event evaluation under load

`POST /limits`

```json
{"eventType": "dynamic1", "maxEvents": 5, "perMillis": 1000}
```

`POST /events`

```json
{"eventType": "dynamic1", "timestamp": 1}
```

`DELETE /limits/dynamic1`

`POST /events`

```json
{"eventType": "dynamic1", "timestamp": 2}
```

**Expected Output:**

* First event → allowed
* After delete → HTTP 404 Not Found
