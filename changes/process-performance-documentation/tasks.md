# Tasks: Operational Workflows, Performance Benchmarks, and Success KPIs

- [x] **Step 1: Define Operational Workflows**
  - [x] Document step-by-step QR/Barcode scanning SOP including Individual vs VIP check-in.
  - [x] Document Batch ticket validation and deduplication processes.
  - [x] Document Turnstile fallback and Edge Rate-limiting fail-safes.

- [x] **Step 2: Technical Benchmarking & Simulation**
  - [x] Establish latency target comparison table for Redis and PostgreSQL locks.
  - [x] Define PageSpeed Core Web Vitals targets and optimization rules.
  - [x] Create k6 and autocannon simulation outline scripts.

- [x] **Step 3: Establish Success KPIs**
  - [x] Define DB connection usage, page hydration, error rates, and check-in throughput metrics.

- [x] **Step 4: Create User-Facing Documentation Artifact**
  - [x] Write the user-facing markdown artifact in the designated Gemini directory containing the consolidated guides, benchmarks, and KPIs.
  - [x] Notify caller agent of completion.

- [x] **Step 5: Phase 4 Performance Verification**
  - [x] Verify benchmark targets (Redis/Postgres lock latencies, Core Web Vitals) alignment across all specs and documents.
  - [x] Confirm test runner script specifications (k6 & autocannon config options) match target loads.
  - [x] Confirm operational guidelines for gate check-ins (800ms debounce, separate VIP check-in logging, SW offline fallback) are finalized and integrated.
