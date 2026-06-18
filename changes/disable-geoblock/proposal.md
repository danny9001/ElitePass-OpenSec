# Proposal: Disable Geoblock Multiport

## Business Intent
The user requested to check and disable the current geoblocking mechanism (`geoblock-multiport`) set up on the system. Disabling the geoblock allows connections to ports 80, 443, and 5001 from any geographic location (removing the restriction that only allows Bolivia, Azure, Cloudflare, AWS, and Google Cloud IP ranges).

## Scope
- Stop the `geoblock-multiport` systemd service if running.
- Disable the `geoblock-multiport` systemd service to prevent it from starting at boot.
- Clean up any active iptables/ip6tables rules inserted by the geoblock scripts.
- Ensure the changes are persistent across reboots.
