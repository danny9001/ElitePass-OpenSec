# Proposal: Unified Optimization & Security Implementation Plan

## Business Intent
Following the comparative analyses of `elitepass-pos` (against InvenTree) and `elitepass-reservas` (against shiyutim/tickets), we have identified critical maturity gaps in inventory logistics (batch control, pack conversions) and reservation security (high-concurrency SQL locking bottlenecks, lack of rate limiters, and bot-grabbing vulnerabilities).

This proposal outlines the strategy to execute these optimizations. Our priority is to implement these upgrades:
1. **Without breaking the live systems** (guaranteeing backward compatibility).
2. **With zero production downtime** (phased migrations and blue-green deployments).
3. **Respecting the multi-tenant context** (ensuring all SQL and Redis namespaces strictly isolate tenant data).

## Scope of Work
- **Security Upgrades (Reservas)**: Cloudflare Turnstile CAPTCHA validation, sliding-window rate limiters, and migration of locking engines from PostgreSQL to Redis.
- **Inventory Logistics Upgrades (POS)**: Schema modifications for batch/expiry tracking (Lotes/FIFO) and product packaging conversions.
- **UI/UX Polish**: Integrating PWA camera scanning, nested category grid navigation, and non-intrusive CAPTCHA validation.
- **Rollout Strategy**: Testing frameworks, database migration policies, and server deployment staging.
