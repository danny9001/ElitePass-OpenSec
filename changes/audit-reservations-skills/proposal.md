# Proposal: Audit Reservations Application and Update Global Skills

## Business Intent
We need to perform a comprehensive hot code audit of the main Reservations application (`elitepass-reservas`) deployed on this server. The objective is to extract accurate, evidence-based details about the following domains:
1. **Core Security**: Token validation, session handling, cloud/server secret resolution, networking settings, and input sanitization/error handling.
2. **Core Events**: Transactional flow, event lifecycle, exact payload schemas, correlation ID patterns, persistence of logs, and retry/idempotency policies.
3. **Core Design System**: Hex color codes, visual theme configuration, visual density metrics (typography, base sizes, line heights, icons/graphs, margins), responsive breakpoints, and user preference persistence/injection.

This extracted knowledge will be used to update the global Antigravity/Gemini skills so that both Gemini and Claude share a unified, up-to-date, and highly precise reference manual for the ecosystem.

## Target Scope
- Analysis of `elitepass-reservas` configurations, environment files, source code under `src/`, middleware, components, styles, and data models.
- Updating existing skill files in `/home/soporte/elitepass-reservas/openspec/skills/`.
- Updating other skill folders as relevant.
