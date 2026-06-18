# Functional Specification: Firewall & Geoblocking

## Overview
To protect the servers of the ElitePass ecosystem from automated scanning, brute-forcing, and distributed attacks, access to critical public ports is restricted at the network layer using `iptables` and `ipset`.

The ports subject to geoblocking are:
- `80` (HTTP - Nginx)
- `443` (HTTPS - Nginx)
- `5001` (SSH)

Traffic to these ports is filtered based on IP block lists, allowing only verified, necessary infrastructure providers, local networks, loopback, and specific geographic locations.

## Scenarios

### Scenario 1: Traffic from Loopback or Local Private Networks
- **Given** incoming traffic is destined for port 80, 443, or 5001
- **When** the incoming interface is loopback (`lo`), or the source IP address belongs to RFC 1918 private subnets (e.g. `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`)
- **Then** the traffic must be accepted and bypassed from geoblocking rules

### Scenario 2: Traffic from Cloudflare, AWS, Google, or Azure
- **Given** incoming traffic is destined for port 80, 443, or 5001
- **When** the source IP address belongs to Cloudflare, AWS, Google Cloud/Google, or Microsoft Azure IP ranges
- **Then** the traffic must be accepted and forwarded to the local service

### Scenario 3: Traffic from Bolivia
- **Given** incoming traffic is destined for port 80, 443, or 5001
- **When** the source IP address is registered within the Bolivia (BO) country code ranges
- **Then** the traffic must be accepted and forwarded to the local service

### Scenario 4: Traffic from Blocked Regions/Providers
- **Given** incoming traffic is destined for port 80, 443, or 5001
- **When** the source IP address does NOT belong to any of the allowed providers or Bolivia, and is not local
- **Then** the connection must be logged to `/var/log/kern.log` with the prefix `"GEOBLOCK_MULTIPORT: "`
- **And** the connection must be dropped (`DROP`) immediately

### Scenario 5: Bypassing via Manual Whitelist (CIDR Subnets)
- **Given** a source IP address or subnet (CIDR) is added to the manual whitelist (`geoblock_whitelist` or `geoblock_whitelist6` of type `hash:net`)
- **When** incoming traffic is received from this IP/subnet on port 80, 443, or 5001
- **Then** the traffic must be accepted even if it originates from a blocked country or provider

### Scenario 6: Safeguard Against Empty IP Sets
- **Given** the geoblock IP sets are being loaded or downloaded
- **When** the download fails or the resulting IP set contains zero entries
- **Then** the geoblocking script must abort before applying the firewall rules and preserving previous valid configurations, preventing a complete traffic block

### Scenario 7: Other Ports
- **Given** incoming traffic is destined for ports other than 80, 443, or 5001
- **When** the firewall processes this packet
- **Then** the geoblock rules must not apply to this traffic

