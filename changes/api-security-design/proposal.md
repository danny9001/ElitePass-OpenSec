# Proposal: API Security & Bot Mitigation Design for elitepass-reservas

## Business Context & Intent
ElitePass operates high-demand ticketing and reservation systems. During high-profile events, the system is vulnerable to ticket-snatching bots, scalpers, scrapers, and denial-of-service attempts. To maintain system availability, ensure fair ticket distribution, and prevent database exhaustion, we must design and implement a multi-layered security architecture.

This proposal details the design of three main layers of defense at the application level:
1. **Next.js 15 Middleware Rate Limiting with Redis**: Implementing a sliding window rate limiter to limit requests per IP and per endpoint dynamically.
2. **Cloudflare Turnstile Validation in Server Actions**: Integrating server-side validation of CAPTCHA/Turnstile tokens for critical actions (e.g., reservation submissions, auth).
3. **Device/Agent Analysis**: Introducing middleware-level and client-telemetry-level checks to detect and block headless browsers and automation frameworks (Puppeteer, Playwright, Selenium).

## Core Requirements
- **Sliding-Window Rate Limiting**: The rate limiter must accurately count requests within a sliding time window (e.g., last 60 seconds) rather than fixed-window blocks to prevent "bursting" at window boundaries. Must fall back gracefully if Redis is unavailable.
- **Turnstile Verification**: Server Actions handling reservations or high-risk mutations must enforce Cloudflare Turnstile token validation on the server side.
- **Bot/Headless Detection**: We must detect headless browser fingerprints (User-Agent anomalies, missing headers, client-side indicators) to block automated clients before they execute intensive backend logic.
- **Developer & User Experience**: The security controls should minimize false positives for legitimate users and run with sub-millisecond overhead in the middleware path.
