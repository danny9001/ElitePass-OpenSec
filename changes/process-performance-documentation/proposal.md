# Proposal: Operational Workflows, Performance Benchmarks, and Success KPIs for ElitePass

## Business Context & Intent
ElitePass coordinates high-throughput event reservations, tickets, access control, and point-of-sale operations. During peak ticket sales and venue entry hours (doors open), the system experiences massive concurrent traffic spikes. Scalability, sub-second latency, and robust offline or fallback operations are critical to preventing entry bottlenecks, double-bookings, API exhaustion, or database failures. 

To ensure operational excellence and establish clear quality gates, this document outlines the business intent and requirements for:
1. **Standardized Operational Workflows**: Standard operating procedures (SOPs) for ticket check-in (individual QR vs. VIP groups), batch processing, barcode/QR scanning, and Cloudflare Turnstile fallback behavior.
2. **Performance Latency Benchmarks**: High-performance constraints on database connection pools, cache locking (Redis vs. Postgres locks), PageSpeed Core Web Vitals, and page hydration.
3. **Success KPIs**: Operational metrics to track throughput, error rates, connection exhaustion, and false-positive security blocks.

---

## Core Requirements & Domains Covered

### 1. Operational Workflows & SOPs
* **QR/Barcode Scanning (Front-Gate Check-In)**: Standard scan-debounce logic, handling of scan modes (GUEST, PARTNER, REWARD, UNIVERSAL), automatic badge assignment, offline check-in, and retry queues.
* **Batch Ticket Management**: Managing high-volume invitations (`InvitationBatch`), including Zod-based deduplication by Document Number (CI) / email, transaction-scoped database insertion, and async processing via queues.
* **Turnstile Fallback**: Non-interactive verification via Cloudflare Turnstile in Edge middleware, with a graceful in-memory fallback mechanism if Turnstile API or Redis rate limiters experience downtime.

### 2. Technical Latency Targets
* **Locking Latencies**: Redis Distributed Lock (redlock) vs. Postgres Row Locking (`FOR UPDATE`). Specific target thresholds in milliseconds to prevent double-booking of event packages or tables.
* **Web Vitals & PageSpeed**: Hard limits on First Contentful Paint (FCP), Largest Contentful Paint (LCP), Total Blocking Time (TBT), Cumulative Layout Shift (CLS), and client-side hydration times.
* **System Capacity**: Overall latency under load (k6 & autocannon testing limits).

### 3. Success Key Performance Indicators (KPIs)
* **Check-In Throughput**: Target check-ins per minute per gate.
* **Resource Utilization**: Database connection limits, PgBouncer pool saturation, CPU/Memory targets.
* **Mitigation Quality**: API blockage rates, rate limiter false-positive rates, WebAuthn fallback rates.
