# Design: Multiport IP Restriction (Bolivia, Azure, Cloudflare, AWS, Google)

## Technical Analysis & Rationale
We need to restrict access on ports `80`, `443`, and `5001` (SSH) to a predefined list of allowed sources: Bolivia, Azure, Cloudflare, AWS, and Google.
- **Firewall Choice:** Since `ufw` is not installed, we use `iptables` and `ipset`. This matches the established pattern in this VM (e.g. `geoblock-ssh.sh` was using `iptables` and `ipset`).
- **Dynamic Insertion Index:** Previously, `geoblock-ssh-restore.sh` failed during boot because it used a static index (`-I INPUT 2`) which resulted in `iptables: Index of insertion too big` when the chain had fewer rules. We will dynamically compute the correct insertion index by finding the position of `CROWDSEC_CHAIN` (if present) and inserting directly after it.
- **Manual Whitelists:** We will reuse `geoblock_whitelist` (IPv4) and `geoblock_whitelist6` (IPv6). This ensures the existing Telegram bot commands (such as `/desbanear <IP>`) will work seamlessly to allow temporary/permanent access for debugging.

## Architecture

### IP Sets
We will define two main `ipset` sets to store the allowed IP ranges:
1. `allowed_multiport` (IPv4, `hash:net` type, maxelem 1,500,000)
2. `allowed_multiport6` (IPv6, `hash:net` type, maxelem 500,000)

These sets will be populated by:
- **Bolivia**: Country zones downloaded from `ipdeny.com`.
- **Cloudflare**: IP ranges downloaded from `cloudflare.com/ips-v4` and `cloudflare.com/ips-v6`.
- **Microsoft Azure**: Announced prefixes for ASN 8075 from RIPE NCC API.
- **AWS**: Parsed prefixes from Amazon's public JSON.
- **Google / Google Cloud**: Parsed prefixes from Google's public and customer IP JSON feeds.

### iptables Rules
For both IPv4 and IPv6:
1. **Rule 1 (Accept Whitelisted):** Allow any IP listed in `geoblock_whitelist` on ports 80, 443, 5001.
2. **Rule 2 (Log Blocked):** Log any connections not in `allowed_multiport` on ports 80, 443, 5001 to `/var/log/kern.log` (prefix `"GEOBLOCK_MULTIPORT: "`).
3. **Rule 3 (Drop Blocked):** Drop any connections not in `allowed_multiport` on ports 80, 443, 5001.

### Scripts
- `/home/soporte/geoblock-multiport.sh`: Complete script to build/download the IP sets and apply the rules.
- `/home/soporte/geoblock-multiport-restore.sh`: Fast restore script for boot time, reading saved IP set dumps to avoid waiting for network/external API downloads.

### Systemd Service
`/etc/systemd/system/geoblock-multiport.service` will replace the existing `geoblock-ssh.service` to ensure rules are loaded on system boot.
