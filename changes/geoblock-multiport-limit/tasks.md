# Tasks: Multiport IP Restriction

- [x] Stop and disable existing `geoblock-ssh` systemd service
- [x] Create `/home/soporte/geoblock-multiport.sh` to download IP ranges and apply rules
- [x] Create `/home/soporte/geoblock-multiport-restore.sh` for boot-time restoration
- [x] Create `/etc/systemd/system/geoblock-multiport.service`
- [x] Make scripts executable (`chmod +x`)
- [x] Execute `/home/soporte/geoblock-multiport.sh` to initialize sets and apply firewall rules
- [x] Enable and start `geoblock-multiport.service`
- [x] Verify that iptables / ip6tables rules are correctly loaded
- [x] Create or update the functional spec file in `openspec/specs/security/spec.md`
