---
date: '2025-05-17T07:18:22+02:00'
draft: false
title: 'My Homelab: Proxmox & Ansible Automation'
---

I run a homelab environment to experiment with virtualization, automation, and IT infrastructure. My main hypervisor is **Proxmox VE**, which allows me to manage virtual machines and containers efficiently. I have been trying to improve my skills with Ansible and wanted to apply what I've learned in my homelab. This is also a good way to document things that I normally do when I create VMs like installing the QEMU agent and creating new users on them and it helps me to automate the process of creating new VMs and containers when I need them.

## What I use my homelab for

- **Learning**: I use my homelab to learn new technologies, test software, and practice skills.
- **Docker**: I run Docker containers for various applications, DNS servers, Twingate connectors, and more.
- **Windows Servers**: I have a few Windows Server VMs for testing with Active Directory, Group Policy, and other Windows features. I also use this to test [PowerShell Universal](https://powershelluniversal.com).

## Why Proxmox?

- Open-source and feature-rich virtualization platform
- Easy web-based management interface
- Supports both VMs and LXC containers
- Great community and documentation

## Typical Workflow

1. **Provision** a new VM or container in Proxmox.
2. **Configure** the system using Ansible playbooks (install packages, set up users, deploy services).
3. **Test** and iterate on new technologies in a safe, isolated environment.

## Source Code

You can find my Ansible playbooks and Proxmox automation scripts in this GitHub repository:  
[adrimus/ansible-proxmox-homelab](https://github.com/adrimus/ansible-proxmox-homelab)

I also have a repo for my Pi-hole setup:
[adrimus/pihole](https://github.com/adrimus/pihole)

## Resources

Here are some resources I’ve used to study and learn about Proxmox, Ansible, and homelab automation:

- [Dive into Ansible](https://www.udemy.com/course/diveintoansible/)
- [Become Ansible ](https://joshduffney.gumroad.com/l/become-ansible)
- [Proxmox VE Documentation](https://pve.proxmox.com/wiki/Main_Page)
- [Ansible Documentation](https://docs.ansible.com/)
- YouTube channels: NetworkChuck, Techno Tim

