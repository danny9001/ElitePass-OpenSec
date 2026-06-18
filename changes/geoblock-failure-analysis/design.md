# Design: Root Cause Analysis of Geoblock Failure

## Identified Issues

### 1. Transition to Bolivia-Only Policy
In the previous script (`geoblock-ssh.sh.bak`), the SSH port (`5001`) was configured to allow connections from a broad list of countries including the USA and Latin America (`us mx gt bz hn sv ni cr pa cu jm ht do tt bb co ve gy sr ec pe bo br cl ar uy py`), plus Starlink, Cloudflare, and Microsoft.
In the new script (`geoblock-multiport.sh`), the policy was changed to restrict access strictly to **Bolivia** (plus AWS, Azure, Google, Cloudflare). Any user or client connecting from other countries (like the USA, other Latin American countries, or other ISPs not listed) was completely blocked from SSH.

### 2. Multiport Blocking of Ports 80 & 443
The new script restricted ports `80` (HTTP) and `443` (HTTPS) alongside `5001` (SSH). By restricting web traffic (`80` and `443`) only to Bolivia and the major Cloud providers, all international web traffic, web crawlers, APIs, and users accessing the web applications from outside Bolivia were completely blocked, which was perceived as "blocking all internet traffic."

### 3. Loopback Interface is Not Exempted
The script applies blocking/dropping rules directly to the general `INPUT` chain for ports `80`, `443`, and `5001`:
```bash
iptables -I INPUT "$IDX_V4" -p tcp -m multiport --dports "$PORTS" -m set ! --match-set "$IPSET_NAME" src -j DROP
```
Since it does not specify an incoming interface (e.g., `-i eth0`), these rules apply to loopback traffic (`lo`) as well. Since `127.0.0.1` and `::1` are not in the Bolivia or Cloud IP sets, local communication on these ports was dropped.

### 4. Private/Local Subnets are Not Whitelisted
The script only populates `allowed_multiport` with public IP blocks. It does not include standard private IP ranges (RFC 1918):
- `10.0.0.0/8` (which includes this server's LAN subnet `10.0.0.0/25` and the remote server `10.0.0.7`)
- `172.16.0.0/12`
- `192.168.0.0/16`

As a result, all local network traffic on ports 80, 443, and 5001 was dropped.

### 5. Invalid `hash:ip` Type for Whitelist Subnet
The user attempted to add the local subnet to the whitelist:
```bash
sudo ipset add geoblock_whitelist 10.0.0.0/25
```
However, the script creates `geoblock_whitelist` as a `hash:ip` set:
```bash
ipset create "$WHITELIST_NAME" hash:ip 2>/dev/null || true
```
A `hash:ip` set only accepts single IP addresses. Attempting to add a CIDR subnet fails with an error and no entries are added.

### 6. Empty IPSET saving due to transient network failures
In `geoblock-multiport.sh`, the download commands for IP ranges (such as `curl` to `ipdeny.com` or calls to the Python helper script) ignore errors or handle them gracefully (returning an empty list):
- Bolivia download: `done < <(curl -sf "${IPDENY_URL}/bo-aggregated.zone" || true)`
- Python helper (Cloudflare, Azure, AWS, Google): Uses `try-except` blocks which output empty lists on HTTP exceptions.

If the script runs during boot when the network interface is not fully initialized, or during a temporary internet drop, all download commands return empty results. The script then proceeds to save these empty ipsets to disk via `ipset save`. On reboot, the restore service loads these empty ipsets. Since `allowed_multiport` contains 0 entries, all incoming packets (including those from Bolivia and Cloudflare) match the `! --match-set allowed_multiport` condition and are dropped.

## Recommended Fixes
1. Create the manual whitelists as `hash:net` instead of `hash:ip` to support CIDR notations (subnets).
2. Explicitly allow all traffic on the loopback (`lo`) interface:
   ```bash
   iptables -I INPUT 1 -i lo -j ACCEPT
   ip6tables -I INPUT 1 -i lo -j ACCEPT
   ```
3. Explicitly allow or whitelist private IP ranges (RFC 1918) in the script so local connections within the LAN are never blocked.
4. Ensure that the geoblocking script aborts and does NOT save or apply rules if the downloaded IP sets are empty or contain fewer entries than a safe threshold.


