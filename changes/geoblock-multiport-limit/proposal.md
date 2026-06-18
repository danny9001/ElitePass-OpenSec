# Proposal: Multiport IP Restriction (Bolivia, Azure, Cloudflare, AWS, Google)

## Business Intent
The goal of this task is to enhance the server's security posture by restricting incoming connections on ports `80` (HTTP), `443` (HTTPS), and `5001` (SSH) to a strict list of allowed origins.

The allowed origins are:
1. **Bolivia (BO)** IP ranges.
2. **Microsoft Azure** IP ranges.
3. **Cloudflare** IP ranges.
4. **Amazon Web Services (AWS)** IP ranges.
5. **Google / Google Cloud** IP ranges.

All other connections to ports `80`, `443`, and `5001` will be dropped (except for manually whitelisted IPs).

## Benefits
- Prevents malicious scans and exploits from unauthorized countries or cloud providers on critical services (Nginx on ports 80/443, SSH on port 5001).
- Preserves external webhook integrations or proxying (e.g. Cloudflare proxying traffic to Nginx, Google crawlers, Azure/AWS integrations).
- Reuses the existing administrative Telegram bot whitelist command (`/desbanear`), ensuring administrators can easily bypass restrictions when needed.
