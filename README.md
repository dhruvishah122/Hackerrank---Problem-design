# Event Throttling Service
[📄 Problem Statement](Problem_statement.md) | [🧪 Testcases](TestCases.md)

> **High-performance in-memory event throttler**  
> Control millions of events per minute with precision and reliability.

---

## Overview

Modern systems — email servers, push notifications, API gateways—must process **massive event streams** without overloading services.  
This project implements a **scalable, in-memory event throttling service**, allowing each event type to be regulated dynamically using **sliding-window rate limits**.

**Key principles:**

- ✅ Fast and fully in-memory  
- ✅ Configurable per event type  
- ✅ Strict enforcement of max events per time window  
- ✅ Automatically discards outdated events  

---

## Why This Matters

Without throttling, high-frequency events can:

- Overload servers and databases  
- Delay critical processing  
- Cause service outages or degraded performance  

This service ensures **stability and fairness**, controlling bursts while keeping the system responsive.

---

## Core Features

- **Dynamic Rate Limits:** Configure event thresholds on the fly.  
- **Sliding-Window Enforcement:** Only recent events are counted.  
- **High Scalability:** Designed for millions of events per minute.  
- **Memory Efficient:** Up to 100 event types, 10,000 timestamps each.  
- **Edge Case Handling:** Invalid timestamps, unknown event types, duplicate rules.  

---

## How It Works

Each event type is associated with:

- `maxEvents` — maximum allowed events  
- `perMillis` — time window in milliseconds  

**Example:**

```

eventType = "sync"
maxEvents = 3
perMillis = 10000

```

### Event Flow Visualization

```

Time(ms): 1000   2000   3000   4000           12000
Event:     ●      ●      ●      ×              ●
Status:  allowed allowed allowed denied       allowed
Window:  <------10,000ms------>              new window

````

- `●` → allowed  
- `×` → denied  
- Only events **inside the current window** are counted  

**Explanation:**  
- Events beyond the window expire automatically.  
- Denied events don’t increment allowed counts.  
- Ensures **burst control** and fair processing.

---

## Performance Highlights

| Metric                     | Limit / Target |
|-----------------------------|----------------|
| Event types supported       | 100            |
| Stored timestamps per type  | 10,000         |
| Throughput                  | 1M+ events/min |
| Memory usage                | Minimal        |
| Response latency            | Low / constant |

**Designed for real-world high-scale systems.**

---

## Status Codes & Behavior

| Endpoint           | Status | Meaning |
|-------------------|--------|---------|
| Configure Limit    | 200    | Rule created successfully |
|                    | 400    | Invalid input |
|                    | 409    | Duplicate rule |
| Evaluate Event     | 200    | Event evaluated (allowed/denied) |
|                    | 400    | Invalid timestamp |
|                    | 404    | Unknown event type |
| List Rules         | 200    | Returns all rules |
| Delete Rule        | 200    | Rule deleted successfully |
|                    | 404    | Rule not found |

---

## Project Highlights

- **Fully in-memory**, no external DB or cache  
- **Accurate sliding-window throttling**  
- Handles **millions of events/minute** without performance degradation  
- **Edge-case safe:** invalid input, unknown event type, duplicate rules  
- Clean, maintainable, testable code  

---

## Ideal Use Cases

- Email servers limiting send rate  
- Push notification systems  
- API request throttling for multi-tenant systems  
- Any real-time event-driven system needing burst control
