# Tasks: Disable Geoblock Multiport

- [x] Stop the geoblock systemd service: `sudo systemctl stop geoblock-multiport.service`
- [x] Disable the geoblock systemd service: `sudo systemctl disable geoblock-multiport.service`
- [x] Clean up existing iptables/ip6tables geoblock rules by running the cleanup portion of the script or manual commands.
- [x] Verify that no geoblock rules remain in `iptables -L INPUT -n` and `ip6tables -L INPUT -n`.

