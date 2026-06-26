# Proposal: High-Level Integration Plan for System Upgrades

## Business Intent
The purpose of this proposal is to establish an architectural blueprint for upgrading the core features of the ElitePass POS (`elitepass-pos`) and Reservations (`elitepass-reservas`) platforms. These upgrades will address critical performance bottlenecks, security vulnerabilities (specifically against automated ticket-grabbing bots), and inventory tracking limitations identified in recent comparative analyses.

The key upgrades are:
1. **Redis-based Locks**: Migrate high-concurrency reservation locking from PostgreSQL to Redis to avoid SQL transactional bottlenecks.
2. **Batch / FIFO Inventory**: Transition from basic aggregate stock totals to expiration-aware, batch-based (Lotes) tracking using FIFO/FEFO algorithms.
3. **Turnstile Verification**: Integrate Cloudflare Turnstile non-intrusive CAPTCHA into reservations and critical actions to block ticket-grabbing bots and automated API scripts.
4. **Packaging Mappings**: Enable advanced supplier-product mapping, supporting different packaging sizes (e.g., boxes, kegs) and automated conversion ratios into base inventory units.

## Core Architectural Guarantees
- **Multi-Tenant Integrity**: All upgraded models and services must enforce strict separation of tenant data via `empresaId` partitioning.
- **Zero-Downtime Deployments**: Schema migrations and lock engine replacements must follow phased rollout strategies (dual-writing, read-fallbacks, feature flags) to ensure zero operational disruption.
- **Strict Code Layer Segregation**: Maintain clean separation between presentation/API, core domain logic, persistence, and external adapters.

## Scope of Integration
- **Platform Scope**: Next.js 15/15.2 (Reservations/Identity) and Node.js 24 + Express 5 (POS).
- **Database Scope**: PostgreSQL 16 managed via Prisma ORM connected via PgBouncer.
- **In-Memory Store**: Redis 7.2 cluster/sentinel for locking, caching, and rate limiting.
- **Security Scope**: Cloudflare Turnstile verification wrapper on APIs/Server Actions.
- **Inventory Engine**: FIFO/FEFO dispatch strategy layer and supplier SKU packaging translator.
