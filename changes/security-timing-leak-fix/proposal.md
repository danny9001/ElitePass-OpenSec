# Proposal: Fix Timing Side-Channel Vulnerability in Authorization Endpoints

## Business Intent
Authorization checks for cron, backup, and internal endpoints compare the user-provided Bearer token with server-side secrets using direct comparison or `timingSafeEqual` with a preceding length check. However, they either exit early if lengths differ or perform a non-constant-time string comparison. This creates a timing side-channel vulnerability that allows an attacker to discover the length or characters of the secret. We must ensure token comparisons are fully constant-time regardless of token length.

## Objectives
1. Convert authorization secret comparisons to a constant-time method by hashing both inputs with SHA-256 before using `timingSafeEqual`.
2. Fix this timing leak across all cron, backup, log archiving, and internal rate-limiting endpoints.
