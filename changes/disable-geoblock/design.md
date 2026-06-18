# Design: Disable Geoblock Multiport

## Analysis
The system uses a systemd unit called `geoblock-multiport.service` which triggers `/home/soporte/geoblock-multiport-restore.sh` at boot time to:
1. Restore ipsets from `/etc/ipset-geoblock-multiport.save` and `/etc/ipset-geoblock-multiport6.save`.
2. Delete previous rules matching the geoblock signature.
3. Apply iptables and ip6tables rules matching ports 80, 443, and 5001.

To disable the geoblock completely:
1. **Stop and disable the systemd service**:
   ```bash
   sudo systemctl stop geoblock-multiport.service
   sudo systemctl disable geoblock-multiport.service
   ```
2. **Remove active iptables/ip6tables rules**:
   The scripts `/home/soporte/geoblock-multiport.sh` and `/home/soporte/geoblock-multiport-restore.sh` contain functions `delete_rules` and `delete_rules6` which clean up all the geoblocking iptables rules.
   We can invoke these functions or manually execute the `iptables -D` and `ip6tables -D` commands to clean the active firewall rules without needing to reload or restart the entire firewall system.

## Verification
- Run `sudo iptables -L INPUT -n --line-numbers` and verify that no rules matching `allowed_multiport` or `geoblock_whitelist` exist.
- Run `sudo ip6tables -L INPUT -n --line-numbers` and verify the same for IPv6.
