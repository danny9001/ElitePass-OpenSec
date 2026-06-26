# Technical Design: API Security & Bot Mitigation for elitepass-reservas

## 1. Sliding Window Rate Limiting with Redis

### Fixed Window vs. Sliding Window
- **Fixed Window**: Divides time into static windows (e.g., 1-minute blocks). If a user has a limit of 10 requests/min, they can send 10 requests at 00:59 and 10 requests at 01:01, resulting in 20 requests within a 2-second interval. This causes backend stress.
- **Sliding Window Log / Counter**: Tracks each request individually (or via sub-intervals) within a rolling window. If the window is 60 seconds, it evaluates the exact number of requests made in the *last 60 seconds* relative to the current millisecond.

### Redis Sorted Set Implementation
To implement this atomically and efficiently, we use a Redis Sorted Set (ZSET) per user/IP/endpoint.
- **Key structure**: `rl:sw:{endpoint}:{ip}`
- **Member**: Millisecond timestamp (unique per request, using UUID or microsecond offset if concurrent).
- **Score**: Millisecond timestamp.

For each request:
1. `ZREMRANGEBYSCORE key -inf (now - windowMs)`: Remove elements older than the window.
2. `ZCARD key`: Get the number of remaining elements.
3. If `ZCARD < limit`:
   - `ZADD key now memberIdentifier`
   - `EXPIRE key windowSeconds`
   - Return allowed = true.
4. Else:
   - Return allowed = false.

### Lua Script (Atomic Execution)
Executing these operations in a single Lua script guarantees atomicity, eliminates race conditions, and reduces network roundtrips:

```lua
local key = KEYS[1]
local now = tonumber(ARGV[1])
local windowMs = tonumber(ARGV[2])
local limit = tonumber(ARGV[3])
local memberId = ARGV[4] -- unique identifier to avoid duplicate members with same timestamp

local clearBefore = now - windowMs
redis.call('zremrangebyscore', key, '-inf', clearBefore)
local currentRequests = redis.call('zcard', key)

if currentRequests < limit then
    redis.call('zadd', key, now, memberId)
    redis.call('expire', key, math.ceil(windowMs / 1000))
    return {1, limit - currentRequests - 1} -- {allowed, remaining}
else
    return {0, 0} -- {blocked, remaining}
end
```

### Next.js 15 Middleware Considerations
Vercel Edge middleware does not support standard TCP sockets natively, but elitepass-reservas is deployed in a Node.js-supported server environment where `ioredis` is already configured. 
- If deployed to Edge: Next.js middleware must call Redis via HTTP API (e.g., Upstash Redis REST API) or via an internal API call.
- If deployed in standard Node.js server: Middleware can import `ioredis` and check limits directly, or use the existing `/api/internal/rate-limit` internal bypass to isolate database and Redis operations from the Edge execution path. We design for both.

---

## 2. Cloudflare Turnstile Verification in Server Actions

Cloudflare Turnstile is a non-interactive CAPTCHA replacement. The client generates a token which must be validated on the server before mutating state.

### Server Action Design Pattern
1. The client form submits the Turnstile token (`cf-turnstile-response`) alongside other data.
2. The Server Action reads the token and calls the Cloudflare verify API: `https://challenges.cloudflare.com/turnstile/v0/siteverify`.
3. If validation fails, it throws a localized error or returns a standard failure payload.

```typescript
"use server";

import { headers } from "next/headers";

interface TurnstileResponse {
  success: boolean;
  "error-codes"?: string[];
  challenge_ts?: string;
  hostname?: string;
}

export async function verifyTurnstileToken(token: string): Promise<boolean> {
  const secretKey = process.env.CLOUDFLARE_TURNSTILE_SECRET_KEY;
  if (!secretKey) {
    console.warn("Turnstile secret key is not set. Bypassing validation (insecure).");
    return true; 
  }

  if (!token) return false;

  // Retrieve client IP for verification security
  const reqHeaders = await headers();
  const ip = reqHeaders.get("cf-connecting-ip") || reqHeaders.get("x-real-ip") || "";

  try {
    const formData = new FormData();
    formData.append("secret", secretKey);
    formData.append("response", token);
    if (ip) {
      formData.append("remoteip", ip);
    }

    const response = await fetch("https://challenges.cloudflare.com/turnstile/v0/siteverify", {
      method: "POST",
      body: formData,
    });

    const data: TurnstileResponse = await response.json();
    return data.success;
  } catch (error) {
    console.error("Turnstile verification failed:", error);
    return false;
  }
}
```

---

## 3. Device/Agent Analysis & Headless Browser Mitigation

We detect automation tools (Puppeteer, Playwright, Selenium, JSDOM) via behavioral indicators, header checks, and client-side telemetry.

### A. Middleware Header Verification
Headless browsers often omit standard headers or present combinations that contradict standard browser behaviors.

1. **User-Agent Filtering**: Block known scraping libraries and automated engines (`HeadlessChrome`, `puppeteer`, `playwright`, `selenium`, `jsdom`, `python-requests`, etc.).
2. **Missing Client Hints (sec-ch-ua)**: Modern browsers (Chrome, Edge, Opera) send Client Hint headers. If a User-Agent claims to be Chrome but lacks `sec-ch-ua`, it is flagged as highly suspicious.
3. **Accept-Language Validation**: Automated scripts often omit `accept-language`. Real user page-level requests must contain an `accept-language` header.
4. **Header Consistency Check**: 
   - A desktop user-agent with mobile Client Hints (`sec-ch-ua-mobile: ?1`).
   - A request claiming to be standard Chrome but lacking `sec-fetch-mode` or `sec-fetch-site`.

### B. Client-side Telemetry & Signature
Pure server-side checks can be bypassed by spoofing headers. We implement a client-side telemetry script that verifies runtime behaviors (e.g., checking for `navigator.webdriver === true`, lack of plugins, or WebGL rendering anomalies) and produces a signed token containing these metrics.
- If `navigator.webdriver` is true, the submission is rejected.
- We check for standard browser properties (e.g., `window.chrome`, `navigator.plugins.length > 0` on desktop).

---

## 4. Integrated Middleware Outline

The updated Next.js Middleware combines these three layers of defense.

```typescript
import { NextRequest, NextResponse } from "next/server";

// 1. Bot & Headless Browser Detection Rules
function isBotRequest(request: NextRequest): { isBot: boolean; reason?: string } {
  const userAgent = request.headers.get("user-agent") || "";
  const uaLower = userAgent.toLowerCase();
  
  // 1.1 User-Agent Keywords Check
  const botKeywords = [
    "headlesschrome", "puppeteer", "playwright", "selenium", 
    "webdriver", "phantomjs", "jsdom", "bot", "crawl", "spider",
    "curl", "wget", "python-requests", "postman", "axios", "scratch"
  ];
  if (botKeywords.some(keyword => uaLower.includes(keyword))) {
    return { isBot: true, reason: "bot_user_agent" };
  }

  // 1.2 Webdriver Header Check
  if (request.headers.has("x-webdriver") || request.headers.has("webdriver")) {
    return { isBot: true, reason: "webdriver_header_present" };
  }

  // 1.3 Client Hints & Browser Inconsistencies for Page/API Mutations
  const path = request.nextUrl.pathname;
  const isPageOrApi = !path.includes(".") && (path.startsWith("/api/") || request.method === "GET");

  if (isPageOrApi) {
    const acceptLang = request.headers.get("accept-language");
    const secChUa = request.headers.get("sec-ch-ua");
    
    // Normal browser requests for HTML/APIs always supply Accept-Language
    if (!acceptLang || acceptLang.length < 2) {
      return { isBot: true, reason: "missing_accept_language" };
    }

    // Chrome/Edge must supply client hints
    if (uaLower.includes("chrome") || uaLower.includes("edge") || uaLower.includes("chromium")) {
      if (!secChUa) {
        return { isBot: true, reason: "missing_client_hints" };
      }
    }
  }

  return { isBot: false };
}
```
