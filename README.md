# Kubernetes-The-Hard-way-Lab
Building a Kubernetes cluster from scratch using Fedora Server across multiple physical and virtual hosts.
----------------------------------------------------------------------------------------------------------------------------------------

This repository documents building a Kubernetes cluster from scratch
using the *Kubernetes The Hard Way* approach, adapted for a personal
homelab environment.

The goal of this project is to demonstrate hands-on understanding of
Kubernetes internals, Linux systems administration, networking,
PKI/certificates, and infrastructure design.

---

## Environment Overview

- OS: Fedora Server (all cluster machines)
- Jumpbox: Fedora Server VM on Arch Linux laptop
- Control Plane: Fedora Server on Lenovo G50
- Worker Nodes: Fedora Server on Dell T310 (3 nodes)

---

## Project Goals

- Build Kubernetes manually without managed tooling
- Understand how core components interact
- Operate a realistic multi-host cluster
- Document decisions, trade-offs, and troubleshooting
- Provide proof-of-work for DevOps / Platform / SysAdmin roles

---

## Repository Structure

- `build-journal/` — chronological build documentation
- `configs/` — Kubernetes and system configuration files
- `scripts/` — helper and automation scripts
- `diagrams/` — architecture and network diagrams
- `notes/` — troubleshooting and lessons learned

---

## Status

🚧 In progress — host preparation and networking standardization
