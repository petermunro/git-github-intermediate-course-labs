# Network Connectivity Troubleshooting Guide

## Symptoms
- Cannot access network resources
- "Network unreachable" errors
- Intermittent connectivity

## Quick Checks

1. **Verify physical connection**

   ```bash
   # Check network interface status
   ip link show
   # or on Windows: ipconfig /all
   ```

2. **Test local connectivity**

   ```bash
   ping 127.0.0.1  # Test loopback
   ping <gateway-ip>  # Test gateway
   ```

3. **Check DNS resolution**

   ```bash
   nslookup google.com
   # or: dig google.com
   ```

4. **Verify network configuration**
   - Check IP address assignment (DHCP vs static)
   - Confirm subnet mask
   - Verify default gateway

## Common Solutions

### Issue: No IP address assigned
**Solution:**
- Restart network service
- Check DHCP server status
- Verify network cable connection

### Issue: Can ping by IP but not by hostname
**Solution:**
- Check DNS server configuration
- Verify /etc/resolv.conf (Linux) or DNS settings (Windows)
- Flush DNS cache

### Issue: Firewall blocking connectivity
**Solution:**
- Check firewall rules
- Temporarily disable to test (re-enable after testing)
- Add necessary exceptions

## When to Escalate

- Multiple users affected
- Hardware failure suspected
- Problem persists after all checks
- Network infrastructure changes needed

## Related Documents

- [Network Configuration Procedure](../procedures/)
- [Daily Network Health Checklist](../checklists/)
