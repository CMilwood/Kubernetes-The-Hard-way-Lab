
## Summary

Today I focused on **network preparation and host readiness** for the Kubernetes-the-Hard-Way homelab. The goal was to ensure the Dell T310 worker, Lenovo control plane, and Fedora jumpbox VM are on the same subnet and properly configured for the upcoming PKI and Kubernetes setup.

---

## Hosts and Initial State

|Host|Role|Hostname|OS|Network|Notes|
|---|---|---|---|---|---|
|Dell T310|Worker node|`dellkubenodes`|Fedora 43 Server|Ethernet → ISP router|Dual NICs available; first NIC connected; static hostname set|
|Lenovo G50-80|Control plane|`lenovokubeserv`|Fedora 43 Server|Initially NAT via Lenovo Wi-Fi; later moved to Ethernet|Web console initially reachable via old IP; reboot required after physical move|
|Arch laptop|Jumpbox host|—|Arch Linux|Ethernet → ISP router (bridge planned)|Running Fedora 43 VM as `kube-jumpbox`; initial NAT network via libvirt|

---

## Actions and Observations

### Hostname Verification

- On all hosts, `hostnamectl` was used to confirm hostnames:
    
    - Dell: `dellkubenodes`
        
    - Lenovo: `lenovokubeserv`
        
    - Jumpbox VM: `kube-jumpbox`
        
- **Observation:** Prompt still showed `@localhost` on web console sessions — purely cosmetic; hostnames were correct system-wide.
    

### Network Discovery

- IP addresses initially:
    
    - Jumpbox VM: `192.168.122.28` (NAT)
        
    - Dell: `192.168.2.82`
        
    - Lenovo: previously `192.168.2.122`
        
- **Issue discovered:** Jumpbox VM was on a different subnet (NAT) than Dell and Lenovo → direct L2 connectivity impossible.
    

### Cockpit Access Issue (Lenovo)

- After moving Lenovo directly to the ISP router, reboot required.
    
- Web console URL changed to:
    

```text
https://lenovokubeserv:9090/
```

- Access failed due to no IPv4 address on Ethernet interface (only IPv6 link-local)
    
- Diagnosed as missing DHCP assignment via systemd-networkd.
    

### IPv4 DHCP Notes

- systemd-networkd requires a `.network` file to enable DHCP
    
- Interface `UP` but missing IPv4 indicates DHCP configuration is needed:
    

```ini
[Match]
Name=<interface-name>

[Network]
DHCP=yes
```

- Steps to enable:
    

```bash
sudo systemctl enable --now systemd-networkd
sudo systemctl restart systemd-networkd
```

---

## Pivoted Plan

Due to the Lenovo IPv4 issue, decision made to **pivot focus**:

1. Prepare **Dell T310 worker node** for Kubernetes (ensure IP via DHCP, systemd-networkd)
    
2. Prepare **Fedora jumpbox VM** on Arch laptop:
    
    - Switch from NAT to **bridged Ethernet** to match Dell’s subnet
        
    - Verify IPv4 on VM
        
    - Confirm ping connectivity between VM and Dell
        
3. Bring Lenovo control plane back into the subnet **after jumpbox + Dell are confirmed**.
    

---

## Physical Network Plan

- **ISP Router (LAN switch)** will host all three nodes
    
    - Dell T310 → Ethernet
        
    - Lenovo → Ethernet (direct, other room)
        
    - Arch laptop → Ethernet (powerline acceptable)
        
- **Jumpbox VM** bridged to Arch Ethernet interface
    
- Wi-Fi on Arch laptop remains for general internet
    
- All nodes expected to acquire IPs via DHCP from router → same subnet
    

---

## Next Steps

1. Ensure Dell T310 worker node receives IPv4 via systemd-networkd DHCP:
    

```bash
# Check interface name
ip link

# Enable DHCP (replace <interface-name> with actual)
sudo vi /etc/systemd/network/20-wired.network
```

```ini
[Match]
Name=<interface-name>

[Network]
DHCP=yes
```

```bash
# Enable and restart networking
sudo systemctl enable --now systemd-networkd
sudo systemctl restart systemd-networkd

# Verify IP assignment
ip a
```

2. Create bridge on Arch laptop, connect Fedora jumpbox VM to Ethernet
    
3. Verify jumpbox ↔ Dell connectivity:
    

```bash
# From jumpbox VM
ping <Dell-IP>
```

4. Freeze network configuration
    
5. Proceed with **PKI/cert generation** and **kubelet setup**
    
6. Integrate Lenovo control plane after IPv4 issue resolved
    

---

### Notes / Lessons Learned

- Always verify IPs after moving a machine to a new physical network; NAT vs bridged is critical.
    
- Cockpit shows hostnames differently depending on DNS / IP; don’t panic if URL changes.
    
- Fedora Server + systemd-networkd needs explicit `.network` files for DHCP — missed this after moving Lenovo.
    
- Planning physical topology **before starting Kubernetes** saves hours of troubleshooting.
    

---

This version is **ready to paste directly** into a `.md` file or GitHub web editor — it will render perfectly with:

- Headings
    
- Bullets
    
- Tables
    
- Command/code blocks with monospace font
    

---
