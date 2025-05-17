---
date: '2025-05-17T07:18:22+02:00'
draft: true
title: 'My Homelab: Proxmox & Ansible Automation'
---

I run a homelab environment to experiment with virtualization, automation, and IT infrastructure. My main hypervisor is **Proxmox VE**, which allows me to manage virtual machines and containers efficiently.

## Why Proxmox?

- Open-source and feature-rich virtualization platform
- Easy web-based management interface
- Supports both VMs and LXC containers
- Great community and documentation

## Automation with Ansible

To keep my environment consistent and reproducible, I use **Ansible** for configuration management. With Ansible, I can:

- Automate the provisioning of new servers and services
- Apply configuration changes across multiple machines
- Version-control my infrastructure as code

## Typical Workflow

1. **Provision** a new VM or container in Proxmox.
2. **Configure** the system using Ansible playbooks (install packages, set up users, deploy services).
3. **Test** and iterate on new technologies in a safe, isolated environment.

## Example Projects

- Automated deployment of web servers and monitoring tools
- Testing PowerShell scripts and Windows Server automation
- Experimenting with network configurations and VLANs

---

If you’re interested in the details or want to see my Ansible playbooks and Proxmox templates, let me know!
