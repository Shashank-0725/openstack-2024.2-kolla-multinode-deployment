## Appendix A: Configuration Files Reference

### globals.yml (Custom Settings)

The only lines added at the bottom of `/etc/kolla/globals.yml`:

```yaml
kolla_base_distro: "ubuntu"
kolla_internal_vip_address: "10.10.10.200"
network_interface: "ens34"
neutron_external_interface: "ens33"
enable_haproxy: "yes"
enable_neutron_provider_networks: "yes"
enable_prometheus: "yes"
enable_grafana: "yes"
```

### Multinode Inventory (Top Section)

```ini
[control]
10.10.10.10

[network]
10.10.10.10

[compute]
10.10.10.11
10.10.10.12

[monitoring]
10.10.10.10

[storage]
10.10.10.10

[deployment]
localhost       ansible_connection=local
```

---

## Appendix B: Important Paths

| File | Location | Purpose |
|------|----------|---------|
| globals.yml | `/etc/kolla/globals.yml` | Main Kolla configuration |
| passwords.yml | `/etc/kolla/passwords.yml` | Auto-generated service passwords |
| admin-openrc.sh | `/etc/kolla/admin-openrc.sh` | OpenStack admin credentials |
| multinode inventory | `~/multinode` | Ansible inventory for node roles |
| Kolla virtualenv | `~/kolla-venv/` | Python environment with kolla-ansible |
| Prometheus template | `~/kolla-venv/share/kolla-ansible/ansible/roles/prometheus/templates/prometheus-web.yml.j2` | Patched for bcrypt fix |

---

## Appendix C: Service Endpoints

| Service | Internal URL | External URL |
|---------|-------------|--------------|
| Keystone (Identity) | http://10.10.10.200:5000 | http://10.10.10.200:5000 |
| Nova (Compute) | http://10.10.10.200:8774 | http://10.10.10.200:8774 |
| Neutron (Network) | http://10.10.10.200:9696 | http://10.10.10.200:9696 |
| Glance (Image) | http://10.10.10.200:9292 | http://10.10.10.200:9292 |
| Placement | http://10.10.10.200:8780 | http://10.10.10.200:8780 |
| Heat (Orchestration) | http://10.10.10.200:8004 | http://10.10.10.200:8004 |
| Horizon (Dashboard) | http://10.10.10.200:80 | http://10.10.10.200:80 |
| Grafana | http://10.10.10.200:3000 | http://10.10.10.200:3000 |
| Prometheus | http://10.10.10.200:9091 | http://10.10.10.200:9091 |
