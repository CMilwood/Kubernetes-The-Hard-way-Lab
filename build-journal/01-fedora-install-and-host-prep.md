# Fedora Server Installation & Initial Host Preparation

This journal documents the initial setup and preparation of Fedora Server
hosts used for building a Kubernetes cluster using the *Kubernetes The Hard Way*
approach. This phase focuses on establishing a clean, predictable Linux baseline
before any Kubernetes components are installed.

---

## Environment Overview

### Hardware
- Dell T310
  - CPU: 1 × quad-core (no SMT)
  - RAM: 16 GB
  - Storage: 148 GB HDD
- Lenovo G50 laptop (control plane)
- Arch Linux laptop (jumpbox VM host)

### Operating System
- Fedora Server (Fedora 43) on all Kubernetes-related machines

### Roles
- Jumpbox: Fedora Server VM (hosted on Arch Linux laptop)
- Control Plane: Fedora Server on Lenovo G50
- Worker Nodes: Fedora Server on Dell T310 (up to 3 nodes)

---

## Fedora Server Installation

Fedora 43 Server was installed on bare metal and virtual machines using the
default server profile (no GUI).

Post-installation checks:
- Verified network connectivity
- Confirmed system received a LAN IP address
- Enabled and accessed Cockpit web console:https://<assigned IP>:9090/
Initial system updates were applied immediately after installation.

---

## Initial Access Issue (User Account)

During Fedora installation, the primary user account was accidentally created with an incorrect username: chis
instead of the intended: chris

This caused initial login confusion when accessing the system remotely.

Resolution:
- Identified the existing user account
- Continued using the correct account with administrative privileges
- Verified shell and Cockpit access

Lesson learned:
> Always verify usernames carefully during OS installation, especially on
> headless or remote-managed systems.

---

## Firewall Configuration

Fedora Server enables `firewalld` by default.

For this Kubernetes lab environment, dynamic firewall management interferes with:
- static routing
- iptables rules
- Kubernetes networking assumptions

Action taken:
  sudo systemctl disable --now firewalld
verified
  systemctl status firewalld
Result:
firewalld disabled
Service inactive (dead)

Swap & zram Handling

After installation, active zram-based swap was detected:
  /dev/zram0

Kubernetes requires swap to be fully disabled.
Actions taken:
  sudo swapoff -a
  
Further investigation showed Fedora uses a systemd generator
(zram-generator) rather than a traditional service to enable zram swap.
Installed packages:
zram-generator
zram-generator-defaults

To prevent swap from re-enabling on reboot, the generator was masked:
  sudo systemctl mask zram-generator

Verification:
  swapon --show

Result:
No active swap devices
zram generator masked
Swap permanently disabled
SELinux Configuration
Fedora defaults to SELinux Enforcing mode.

Kubernetes networking and container workloads can be disrupted by SELinux
enforcement without custom policies.

Actions taken:
Set runtime mode to permissive:
  sudo setenforce 0

Persist permissive mode across reboots:
  sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config

Verification:
getenforce

Result:
SELinux running in Permissive mode
Policies retained but not enforced

Current Host State Summary
At the end of this phase, the Fedora Server hosts meet Kubernetes prerequisites:
Fedora 43 Server installed and fully updated
Cockpit remote management enabled
firewalld disabled
Swap fully disabled (including zram)
SELinux set to permissive
Systems ready for Kubernetes-specific kernel and networking configuration


