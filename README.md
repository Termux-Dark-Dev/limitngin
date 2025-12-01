# limitngin

A lightweight, zero-dependency rate limiter middleware for **Express**.  
Implements a simple and efficient fixed-interval request counter using an in-memory store.

The package exports a default function that internally creates a `LimitNgin` instance and returns a ready-to-use Express middleware.

---

## Installation

```bash
npm install limitngin
```

---

## Quick Start (Express)

```ts
import express from "express";
import limitNgin from "limitngin";

const app = express();

app.use(
  limitNgin({
    intervalInSec: 60,          // 1-minute window
    allowedNoOfRequests: 100    // max 100 requests/min per IP
  })
);

app.get("/", (req, res) => {
  res.json({ message: "OK" });
});

app.listen(3000, () => console.log("server running"));
```

---

## API

### Default Export
```ts
limitNgin(config?: LimitNginConfig)
```

Returns an Express middleware function that handles rate limiting.

---

## Configuration

```ts
export type LimitNginConfig = {
  intervalInSec: number;        // time window in seconds
  allowedNoOfRequests: number;  // max requests allowed within the window
};
```

### Default Values
```ts
intervalInSec: 60
allowedNoOfRequests: 100
```

---

## How It Works

A simple in-memory store tracks request counts per IP:

```ts
{
  "<ip>": {
    req_count: number;
    created_at: number; // timestamp in ms
  }
}
```

### Processing Steps

1. Extract client IP from `req.ip`
2. If IP not seen before → create entry
3. If inside the interval:
   - increment `req_count` until reaching the limit
4. If interval expired:
   - reset the entry for that IP
5. If limit exceeded → respond with:

```json
{
  "message": "too many request"
}
```

HTTP Status: `429`

---

## Automatic Cleanup

A cleanup runs every:

```
max(intervalInSec, 60) seconds
```

Expired IP entries are removed to prevent memory growth.

---

## Example Blocked Response

```json
{
  "message": "too many request"
}
```

---

## Internal Overview

Internally, your middleware behaves as:

```ts
const limiter = new LimitNgin(config);

return (req, res, next) => {
  limiter.listen(req, res, next);
};
```

Core methods:

- `#shouldBlock(ip)` → checks if request should be blocked  
- `#calculateTimeDiff()` → computes elapsed time  
- `#cleanup()` → removes stale entries  

---

## Upcoming

- More detailed response metadata (e.g., retry-after)  
- Enhanced cleanup logic  
- Optional pluggable storage adapters  
- Framework-agnostic mode  

---

## License

MIT
