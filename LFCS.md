#### operations deployment
- config kernel paramenters, persistent and non-persistent
- diagnose, identify, manage, and troubleshoot processes and services
- manage, schedule jobs for executing commands
- search for, install, valdiate and maintain software packages and repos
- recover from hw, os, filesystem failures 
- manage virtual machines(libvirt)
- config container engines, create and manage containers
- create and enforce MAC using SELinux

#### networking 
- config IPv4 and IPv6 networking and hostname resolution
- set and sync system time using time servers
- monitor and troubleshootin networking
- config OpenSSH server and client
- config packet filtering, port redirection, and NAT
- config static routing
- config bridge and bonding devices
- implement reverse proxies and load balancers

#### storage

- config and manage LVM storage
- manage and config the virtual file system
- create, manage, troubleshoot filesystems
- user remote filesystems and network block devices 
- conifg and manage swap space 
- config filesystem automounters
- monitor storage performance

#### essential commands

- basic git operations
- create, config, and troubleshoot services
- monitor, troubleshoot system performance and services
- determine app and service specific constraints 
- troubleshoot diskspace issues
- work with SSL certs

#### users and groups

- create and manage local user and grp accounts
- manage personal and system-wide env profiles
- config user resource limits
- config and manage ACLs
- config system to use LDAP user and grp accounts 

# LFCS Learning Plan
## Linux Foundation Certified System Administrator Exam Preparation

**Target Date:** Q1 2026 (Schedule exam by March 2026)  
**Study Duration:** 8-10 weeks  
**Daily Commitment:** 2-3 hours  

---

## Prerequisites
- Complete [Introduction to Linux (LFS101)](https://trainingportal.linuxfoundation.org/courses/introduction-to-linux-lfs101)
- Complete [Introduction to DevOps and Site Reliability Engineering (LFS162)](https://trainingportal.linuxfoundation.org/courses/introduction-to-devops-and-site-reliability-engineering-lfs162)
- Enroll in [Linux System Administration Essentials (LFS207)](https://trainingportal.linuxfoundation.org/courses/linux-system-administration-essentials-lfs207)

---

## Week 1-2: Essential Commands & Fundamentals

### Focus Areas
- **Basic Git Operations**
  - Initialize repositories, clone, commit, push, pull
  - Branch management, merge conflicts
  - .gitignore, git log, git diff
  - Practice: Set up local git repos, simulate team workflows

- **System Performance Monitoring**
  - `top`, `htop`, `ps`, `free`, `df`, `du`
  - `iostat`, `vmstat`, `sar`, `mpstat`
  - `/proc` filesystem exploration
  - Log analysis: `journalctl`, `/var/log/`

- **SSL Certificates**
  - Generate self-signed certificates with `openssl`
  - Certificate signing requests (CSR)
  - Certificate chain verification
  - Configure SSL/TLS for services

### Practice Labs
- [ ] Set up git workflow with multiple branches
- [ ] Monitor system performance during stress tests
- [ ] Create and verify SSL certificates for Apache/Nginx
- [ ] Troubleshoot high CPU/memory usage scenarios

### Resources
- LFS207 Chapters 1-4
- Linux man pages: `man bash`, `man git`
- Practice: Set up a local VM with monitoring tools

---

## Week 3-4: Users, Groups & Permissions

### Focus Areas
- **User and Group Management**
  - `useradd`, `usermod`, `userdel`, `groupadd`, `groupmod`
  - `/etc/passwd`, `/etc/shadow`, `/etc/group`
  - Password policies: `chage`, `passwd`
  - Home directory management

- **Environment Profiles**
  - System-wide: `/etc/profile`, `/etc/bash.bashrc`, `/etc/environment`
  - User-specific: `~/.bashrc`, `~/.bash_profile`, `~/.profile`
  - Setting PATH, aliases, environment variables

- **Resource Limits**
  - `/etc/security/limits.conf`
  - `ulimit` command
  - Control groups (cgroups)
  - Process priorities: `nice`, `renice`

- **Access Control Lists (ACLs)**
  - `getfacl`, `setfacl`
  - Default ACLs on directories
  - Mask calculations
  - ACL backup and restore

- **LDAP Integration**
  - Install and configure `sssd`, `nss-pam-ldapd`
  - `/etc/nsswitch.conf`, `/etc/pam.d/`
  - LDAP client configuration
  - Troubleshoot authentication issues

### Practice Labs
- [ ] Create user accounts with specific UID/GID
- [ ] Set up resource limits for users
- [ ] Configure complex ACL permissions
- [ ] Integrate system with LDAP server
- [ ] Test profile scripts for different scenarios

### Resources
- LFS207 Chapters 5-7
- Red Hat documentation on Identity Management
- Practice: Set up FreeIPA or OpenLDAP server

---

## Week 5-6: Storage Management

### Focus Areas
- **LVM (Logical Volume Management)**
  - Physical volumes (PV): `pvcreate`, `pvdisplay`
  - Volume groups (VG): `vgcreate`, `vgextend`, `vgreduce`
  - Logical volumes (LV): `lvcreate`, `lvextend`, `lvresize`
  - Snapshots: create, restore, remove
  - LVM troubleshooting and recovery

- **Filesystem Management**
  - Create: `mkfs.ext4`, `mkfs.xfs`, `mkfs.btrfs`
  - Mount options: `/etc/fstab`, UUID/LABEL
  - Filesystem check and repair: `fsck`, `xfs_repair`
  - Tune filesystems: `tune2fs`, `xfs_admin`
  - Disk quotas: `quotacheck`, `edquota`, `repquota`

- **Virtual Filesystem (VFS)**
  - `/proc`, `/sys`, `/dev` filesystems
  - tmpfs, devtmpfs configuration
  - Bind mounts and mount namespaces

- **Remote Filesystems**
  - NFS: server and client configuration
  - CIFS/SMB: mount Windows shares
  - iSCSI: initiator configuration
  - Network block devices (NBD)

- **Automounters**
  - `autofs` configuration
  - `/etc/auto.master`, `/etc/auto.misc`
  - Direct and indirect maps
  - Wildcard and replicated mounts

- **Swap Space**
  - Create swap: partition and file-based
  - `swapon`, `swapoff`, priority settings
  - Configure in `/etc/fstab`
  - Swap performance tuning: `vm.swappiness`

- **Storage Performance**
  - `iotop`, `iostat`, `blktrace`
  - Disk I/O schedulers
  - RAID monitoring: `mdadm`

### Practice Labs
- [ ] Create LVM setup with multiple VGs and LVs
- [ ] Extend root filesystem using LVM
- [ ] Set up NFS server and mount on clients
- [ ] Configure autofs for home directories
- [ ] Create and manage swap space
- [ ] Monitor and optimize disk I/O

### Resources
- LFS207 Chapters 8-12
- Storage tutorials: LVM, NFS, iSCSI
- Practice: Set up multiple VMs to test network storage

---

## Week 7-8: Networking

### Focus Areas
- **IPv4 and IPv6 Configuration**
  - NetworkManager: `nmcli`, `nmtui`
  - Legacy: `ifconfig`, `ip` command
  - `/etc/sysconfig/network-scripts/` (RHEL) or `/etc/netplan/` (Ubuntu)
  - Hostname: `hostnamectl`, `/etc/hosts`, `/etc/hostname`

- **DNS Resolution**
  - `/etc/resolv.conf`, `/etc/nsswitch.conf`
  - `dig`, `nslookup`, `host` commands
  - systemd-resolved configuration

- **Time Synchronization**
  - `chronyd` configuration: `/etc/chrony.conf`
  - `timedatectl` commands
  - NTP pools and stratum levels
  - Troubleshoot time sync issues

- **Network Troubleshooting**
  - `ping`, `traceroute`, `mtr`, `netstat`, `ss`
  - `tcpdump`, `wireshark` (optional)
  - `/proc/net/` statistics
  - Connection state analysis

- **SSH Configuration**
  - Server: `/etc/ssh/sshd_config`
  - Client: `~/.ssh/config`, `~/.ssh/known_hosts`
  - Key-based authentication: `ssh-keygen`, `ssh-copy-id`
  - Port forwarding, tunneling, ProxyJump
  - Security hardening: disable root login, password auth

- **Firewall & Packet Filtering**
  - `firewalld`: zones, services, rich rules
  - `iptables`/`nftables`: chains, rules, NAT
  - Port redirection and forwarding
  - NAT (SNAT, DNAT, masquerading)

- **Static Routing**
  - `ip route` commands
  - `/etc/sysconfig/network-scripts/route-*`
  - Policy-based routing
  - Multiple routing tables

- **Bridge and Bonding**
  - Bridge: `brctl` or `ip link` for VM networking
  - Bonding: modes (active-backup, 802.3ad, etc.)
  - Configuration files and NetworkManager
  - VLAN tagging

- **Reverse Proxy & Load Balancing**
  - Nginx as reverse proxy
  - HAProxy for load balancing
  - Configuration and health checks
  - SSL termination

### Practice Labs
- [ ] Configure static IPv4 and IPv6 addresses
- [ ] Set up chronyd to sync with NTP servers
- [ ] Configure SSH with key authentication and hardening
- [ ] Create firewall rules with firewalld and iptables
- [ ] Set up NAT and port forwarding
- [ ] Configure network bonding and bridging
- [ ] Deploy Nginx reverse proxy with SSL
- [ ] Set up HAProxy with multiple backends

### Resources
- LFS207 Chapters 13-17
- Linux networking guides
- Practice: Multi-VM network topology setup

---

## Week 9-10: Operations & Deployment

### Focus Areas
- **Kernel Parameters**
  - Runtime: `sysctl` command
  - Persistent: `/etc/sysctl.conf`, `/etc/sysctl.d/`
  - Common parameters: network, memory, security
  - `/proc/sys/` hierarchy

- **Process & Service Management**
  - systemd: `systemctl`, unit files
  - Service creation and management
  - Dependencies and ordering
  - Targets (runlevels equivalent)
  - Process signals: `kill`, `pkill`, `killall`
  - Background jobs: `bg`, `fg`, `jobs`, `nohup`

- **Job Scheduling**
  - `cron`: `crontab -e`, `/etc/cron.d/`, `/etc/cron.daily/`
  - `anacron` for systems that aren't always on
  - `at` for one-time jobs: `at`, `atq`, `atrm`
  - systemd timers as cron alternative

- **Package Management**
  - RPM-based: `yum`, `dnf`, `rpm`
  - Debian-based: `apt`, `apt-get`, `dpkg`
  - Repository configuration: `/etc/yum.repos.d/`, `/etc/apt/sources.list`
  - GPG key verification
  - Package queries, dependencies, cleanup

- **System Recovery**
  - Boot process: GRUB, initramfs
  - Single-user mode, rescue mode
  - Filesystem recovery: `fsck`, backups
  - Emergency boot from live media
  - `/etc/fstab` troubleshooting

- **Virtual Machines (libvirt)**
  - `virsh` command basics
  - VM creation: `virt-install`
  - VM management: start, stop, clone
  - Virtual networks and storage pools
  - Snapshots and backups

- **Container Engines**
  - Docker/Podman basics
  - Container lifecycle: run, stop, rm
  - Images: pull, build, push
  - Volumes and networking
  - `docker-compose`/`podman-compose`

- **SELinux (Mandatory Access Control)**
  - Modes: enforcing, permissive, disabled
  - `getenforce`, `setenforce`, `setstatus`
  - Context management: `ls -Z`, `chcon`, `restorecon`
  - Booleans: `getsebool`, `setsebool`
  - Policy management: `semanage`
  - Troubleshooting: `audit2why`, `audit2allow`

### Practice Labs
- [ ] Modify kernel parameters for network tuning
- [ ] Create custom systemd service units
- [ ] Schedule jobs with cron and systemd timers
- [ ] Set up local package repository
- [ ] Simulate system failure and recover
- [ ] Create and manage VMs with virsh
- [ ] Run containerized applications with Podman
- [ ] Configure SELinux policies for custom applications

### Resources
- LFS207 Chapters 18-25
- systemd documentation
- SELinux guides and tutorials
- Practice: Break and fix systems intentionally

---

## Final 2 Weeks: Exam Preparation

### Mock Exams & Practice
- [ ] Complete practice exams from Linux Foundation
- [ ] Set up timed labs (2 hours each)
- [ ] Review weak areas identified in practice tests
- [ ] Simulate exam environment (no Google, only man pages)

### Key Skills to Master
- Speed in using command line (no GUI)
- Efficient use of `man` pages and `/usr/share/doc/`
- Troubleshooting methodology
- Time management (prioritize easy tasks first)

### Exam Day Checklist
- [ ] Test internet connection and system
- [ ] Have government ID ready
- [ ] Clear workspace (exam proctored)
- [ ] Review exam rules and allowed resources
- [ ] Arrive 15 minutes early

### Important Exam Tips
1. **Read questions carefully** - understand what is being asked
2. **Verify your work** - test commands before moving on
3. **Use tab completion** - save time and avoid typos
4. **Skip and return** - don't get stuck on one question
5. **Check persistence** - ensure changes survive reboot when required
6. **Time awareness** - 2 hours goes fast with 15-20 tasks

---

## Study Resources Summary

### Official Resources
- [LFS101: Introduction to Linux](https://trainingportal.linuxfoundation.org/courses/introduction-to-linux-lfs101)
- [LFS162: Introduction to DevOps and SRE](https://trainingportal.linuxfoundation.org/courses/introduction-to-devops-and-site-reliability-engineering-lfs162)
- [LFS207: Linux System Administration Essentials](https://trainingportal.linuxfoundation.org/courses/linux-system-administration-essentials-lfs207)
- [LFCS Exam Domains](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)

### Additional Resources
- Linux man pages (your best friend during exam)
- Documentation in `/usr/share/doc/`
- Distribution-specific wikis (Arch Wiki, RHEL docs, Ubuntu docs)
- YouTube channels: Learn Linux TV, Linux Academy
- Practice platforms: KodeKloud, A Cloud Guru

### Lab Setup
- **Recommended:** 2-3 VMs (VirtualBox or KVM)
- **OS:** CentOS Stream/RHEL/Ubuntu/Debian
- **Resources:** 2GB RAM, 20GB disk per VM
- **Network:** Configure bridged and NAT networking

---

## Weekly Progress Tracking

| Week | Topic | Status | Notes |
|------|-------|--------|-------|
| 1-2  | Essential Commands | ⬜ |  |
| 3-4  | Users & Groups | ⬜ |  |
| 5-6  | Storage | ⬜ |  |
| 7-8  | Networking | ⬜ |  |
| 9-10 | Operations | ⬜ |  |
| 11-12 | Exam Prep | ⬜ |  |

---

## Notes & Insights

_Use this section to document key learnings, gotchas, and personal notes_

---

**Last Updated:** January 5, 2026
