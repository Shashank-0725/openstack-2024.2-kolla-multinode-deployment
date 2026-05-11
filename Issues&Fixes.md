## 7. Known Issues & Fixes

### Issue 1: bcrypt v5 Password Length Error

**Symptom:**
```
AnsibleFilterError: Could not hash the secret.. password cannot be longer 
than 72 bytes, truncate manually if necessary
```

**Root Cause:** bcrypt 5.x strictly enforces the 72-byte password limit for bcrypt hashing.

**Fix:** Applied in Phase 3, Steps 4.8 and 4.9.

---

### Issue 2: OVS Bridge Steals Internet After Reboot

**Symptom:** After rebooting VMs, internet connectivity is lost because OpenVSwitch takes over `ens33` as a port in the `br-ex` bridge.

**Root Cause:** When the `openvswitch_vswitchd` Docker container starts, it reclaims `ens33` into `br-ex`, stripping the IP address from `ens33`.

**Fix:** Move the NAT IP from `ens33` to `br-ex` on each affected VM:

```bash
# Replace <NAT_IP> with the VM's NAT IP and <GATEWAY> with the NAT gateway
sudo ip link set br-ex up
sudo ip addr flush dev ens33
sudo ip addr add <NAT_IP>/24 dev br-ex
sudo ip route del default 2>/dev/null
sudo ip route add default via <GATEWAY> dev br-ex
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

---

### Issue 3: Non-Unique Hostnames Cause Precheck Failure

**Symptom:**
```
Hostname has to resolve uniquely to the IP address of api_interface
```

**Root Cause:** All VMs have the default Ubuntu hostname (e.g., `ubuntu`), causing DNS confusion.

**Fix:** Set unique hostnames on each VM (see Section 3.2), then re-run bootstrap.

---

### Issue 4: Docker Image Pull Fails (DNS Resolution)

**Symptom:**
```
failed to resolve reference "quay.io/openstack.kolla/nova-api:2024.2-ubuntu-noble":
lookup quay.io: server misbehaving
```

**Root Cause:** Internet connectivity lost (usually due to Issue 2 or NAT adapter disconnection).

**Fix:** Restore internet connectivity (see Issue 2), then re-run the deploy command. It's idempotent.

---

## 8. Startup Procedure After Reboot

When the host machine or VMs are restarted:

1. **Power on all 3 VMware VMs** (controller first, then computes)
2. **Wait ~2-3 minutes** for Docker containers to auto-start
3. **Fix internet on each VM** (see Issue 2 in Known Issues):
   ```bash
   # Controller (SSH via 10.10.10.10):
   echo '123' | sudo -S bash -c '
     ip link set br-ex up
     ip addr flush dev ens33
     ip addr add <CONTROLLER_NAT_IP>/24 dev br-ex
     ip route del default 2>/dev/null
     ip route add default via 192.168.137.1 dev br-ex
     echo "nameserver 8.8.8.8" > /etc/resolv.conf'
   ```
   Repeat for both compute nodes with their respective NAT IPs.
4. **Verify services:**
   ```bash
   source /etc/kolla/admin-openrc.sh
   openstack service list
   ```
5. **Check running containers:**
   ```bash
   sudo docker ps | grep -c healthy
   # Should show 20+ healthy containers on the controller
   ```

---