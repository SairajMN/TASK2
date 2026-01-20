# OS Security Checklist & Hardening Document


## Platforms Covered: Linux (Ubuntu), Windows


---

## Table of Contents

1. [Linux Virtual Machine Setup](#linux-virtual-machine-setup)
2. [User Accounts & Access Control](#user-accounts--access-control)
3. [File Permissions in Detail](#file-permissions-in-detail)
4. [Administrator vs Standard User](#administrator-vs-standard-user)
5. [Firewall Configuration](#firewall-configuration)
6. [Running Processes & Services](#running-processes--services)
7. [Disabling Unnecessary Services](#disabling-unnecessary-services)
8. [OS Hardening Best Practices](#os-hardening-best-practices)
9. [Final OS Security Checklist](#final-os-security-checklist)
10. [Conclusion](#conclusion)

---

## Linux Virtual Machine Setup

### Understanding Virtual Machines

A Virtual Machine (VM) is a software emulation of a physical computer that runs an operating system and applications just like a real computer. VMs allow multiple operating systems to run simultaneously on a single physical machine, each isolated from the host system.

### Why Use VirtualBox?

VirtualBox is a free, open-source virtualization software that creates and manages VMs. It's ideal for security labs because:

- It provides complete isolation between the VM and host system
- Snapshots allow reverting to clean states after security testing
- It's lightweight and doesn't require expensive hardware
- Cross-platform support (works on Windows, Linux, macOS)

### Steps to Install Ubuntu Linux on VirtualBox

1. **Download VirtualBox**: Visit virtualbox.org and download the latest version for your host OS.

2. **Download Ubuntu ISO**: Go to ubuntu.com/download/desktop and download the latest Ubuntu Desktop ISO.

3. **Create New VM**:
   - Open VirtualBox and click "New"
   - Name: "Ubuntu Security Lab"
   - Type: Linux
   - Version: Ubuntu (64-bit)
   - Allocate at least 2GB RAM and 20GB virtual disk

4. **Install Ubuntu**:
   - Select the downloaded ISO as the installation media
   - Follow the installation wizard
   - Create a user account with a strong password

5. **Post-Installation**:
   - Install VirtualBox Guest Additions for better performance
   - Update the system: `sudo apt update && sudo apt upgrade`

### Alternative: Using Windows Security Instead of VM

If VM setup is not feasible, you can perform security hardening on a Windows host system using built-in tools. However, VMs provide better isolation for learning without risking your main system.

### Security Advantages of VM Isolation

- **Containment**: Malware or security experiments stay within the VM
- **Snapshot Recovery**: Easy rollback to clean states
- **Testing Environment**: Safe space to practice potentially dangerous commands
- **Resource Control**: Limits resource usage to prevent system-wide impact

---

## User Accounts & Access Control

### What are User Accounts in OS?

User accounts are identities that allow individuals to access an operating system. Each account has unique credentials and permissions that determine what actions the user can perform on the system.

### Root User vs Normal User

- **Root User (Linux)**: The superuser with unlimited access to all system files and commands. Equivalent to Administrator in Windows.
- **Normal User**: Limited account with restricted permissions for everyday tasks.

### User Groups and Their Role

Groups organize users with similar permissions. For example:

- `sudo` group: Members can execute administrative commands
- `wheel` group: Similar to sudo group in some Linux distributions

### Access Control Concepts

- **Authentication**: Verifying user identity (username/password, biometrics)
- **Authorization**: Determining what authenticated users can access

### Essential Commands

**Check current user:**

```bash
whoami
```

Output: Your username

**Display user and group information:**

```bash
id
```

Output: uid=1000(username), gid=1000(group), groups=...

**List user's groups:**

```bash
groups
```

**Understanding /etc/passwd:**
This file contains user account information. Each line represents a user:

```
username:x:uid:gid:comment:home_directory:shell
```

Example: `student:x:1000:1000:Student User:/home/student:/bin/bash`

---

## File Permissions in Detail

### Understanding Permissions

File permissions control who can read, write, or execute files. Linux uses three permission types:

- **Read (r)**: View file contents or list directory contents
- **Write (w)**: Modify file contents or create/delete files in directory
- **Execute (x)**: Run executable files or access directory contents

### Permission Structure

Every file has three permission sets:

- **Owner**: The user who owns the file
- **Group**: Users in the file's group
- **Others**: All other users

### Permission Notation

- **Symbolic**: rwx (read-write-execute), rw- (read-write), etc.
- **Numeric**: r=4, w=2, x=1, so rw-=6, rwx=7

Example permission string: `-rw-r--r--`

- File type: `-` (regular file)
- Owner: `rw-` (read+write = 6)
- Group: `r--` (read only = 4)
- Others: `r--` (read only = 4)

### Essential Commands

**List files with permissions:**

```bash
ls -l
```

Sample output:

```
-rw-r--r-- 1 student student 1024 Jan 20 23:00 example.txt
drwxr-xr-x 2 student student 4096 Jan 20 23:00 documents/
```

**Change permissions symbolically:**

```bash
chmod u+x script.sh    # Add execute for owner
chmod g-w file.txt     # Remove write for group
chmod o+r file.txt     # Add read for others
```

**Change permissions numerically:**

```bash
chmod 755 script.sh    # rwxr-xr-x
chmod 644 file.txt     # rw-r--r--
chmod 600 secret.txt   # rw-------
```

**Change ownership:**

```bash
chown student file.txt              # Change owner only
chown student:developers file.txt   # Change owner and group
chown :developers file.txt          # Change group only
```

### Real-World Security Examples

- **Sensitive Files**: Set 600 permissions on SSH private keys
- **Web Directories**: Use 755 for directories, 644 for files
- **Scripts**: Ensure only authorized users can execute critical scripts

---

## Administrator vs Standard User

### Why Admin/Root Access is Dangerous

Administrator (Windows) or root (Linux) privileges allow complete system control. Misuse can:

- Install malware system-wide
- Delete critical system files
- Modify security settings
- Compromise all user data

### Principle of Least Privilege (POLP)

Users should have only the minimum permissions needed for their tasks. This limits damage if accounts are compromised.

### The sudo Command

`sudo` allows temporary elevation to root privileges for specific commands:

```bash
sudo apt update    # Run apt update as root
sudo -i            # Start root shell (use carefully!)
```

### Real-World Attack Examples

- **WannaCry Ransomware**: Spread via admin credential misuse
- **SolarWinds Hack**: Compromised admin accounts led to widespread breaches
- **Local Privilege Escalation**: Attackers exploit admin software vulnerabilities

**Best Practice**: Never use admin accounts for daily tasks. Use `sudo` only when necessary.

---

## Firewall Configuration

### Linux: UFW (Uncomplicated Firewall)

**What is UFW?**
UFW is a user-friendly frontend for iptables, simplifying firewall management.

**Basic Commands:**

```bash
sudo ufw status                    # Check firewall status
sudo ufw enable                    # Enable firewall
sudo ufw disable                   # Disable firewall (not recommended)
```

**Managing Rules:**

```bash
sudo ufw allow 22/tcp              # Allow SSH (TCP port 22)
sudo ufw allow from 192.168.1.0/24 # Allow from specific subnet
sudo ufw deny 80/tcp               # Block HTTP (TCP port 80)
sudo ufw delete allow 22/tcp       # Remove rule
```

**Common Rules:**

```bash
sudo ufw default deny incoming     # Deny all incoming by default
sudo ufw default allow outgoing    # Allow all outgoing by default
sudo ufw allow ssh                 # Allow SSH
sudo ufw allow http                # Allow web traffic
```

### Windows: Windows Defender Firewall

**Turning Firewall ON/OFF:**

1. Open Windows Security → Firewall & network protection
2. Enable "Domain network", "Private network", and "Public network" firewalls

**Managing Rules:**

1. Click "Advanced settings"
2. Create inbound/outbound rules for specific ports/programs

**Security Benefits:**

- Blocks unauthorized network access
- Prevents exploitation of open ports
- Isolates infected systems from network

---

## Running Processes & Services

### Understanding Processes and Services

- **Process**: A running instance of a program in memory
- **Service/Daemon**: Background processes that provide system functions

### Essential Commands

**Linux Process Management:**

```bash
ps aux                    # List all processes
top                       # Interactive process viewer
htop                      # Enhanced top (install with apt)
systemctl list-units --type=service  # List services
```

**Windows Process Management:**

```cmd
tasklist                  # List running processes
taskmgr                   # Task Manager GUI
```

### How Attackers Exploit Running Services

- **Buffer Overflow**: Attack open ports/services
- **Unpatched Vulnerabilities**: Exploit known service weaknesses
- **Privilege Escalation**: Compromise service accounts
- **Denial of Service**: Overload services

**Security Practice**: Regularly audit running processes and minimize exposed services.

---

## Disabling Unnecessary Services

### Understanding Attack Surface

Attack surface is the sum of all potential vulnerabilities an attacker can exploit. Unused services increase attack surface unnecessarily.

### Why Unused Services are Risky

- Each service represents a potential entry point
- Unpatched services harbor known vulnerabilities
- Default configurations may be insecure
- Resource consumption without benefit

### Listing and Managing Services

**Linux:**

```bash
systemctl list-units --type=service --state=active  # List active services
systemctl stop service_name                         # Stop service
systemctl disable service_name                      # Prevent auto-start
systemctl mask service_name                         # Completely disable
```

**Windows:**

1. Open Services (services.msc)
2. Right-click service → Properties → Stop/Disable

### Example Services to Disable

- **Telnet**: Insecure remote access (use SSH instead)
- **FTP**: Unencrypted file transfer (use SFTP/SCP)
- **Unused Web Servers**: Apache/Nginx if not needed
- **Remote Desktop**: If not required for administration

**Caution**: Only disable services you understand. Test thoroughly after changes.

---

## OS Hardening Best Practices

### Strong Password Policies

- Minimum 12 characters with complexity
- Regular password changes
- Use password managers
- Avoid password reuse

### Regular System Updates

```bash
# Linux
sudo apt update && sudo apt upgrade

# Windows: Settings → Update & Security → Windows Update
```

### Secure Boot & Disk Encryption

- **Secure Boot**: Prevents unauthorized OS loading
- **Full Disk Encryption**: Protects data if device is lost/stolen
  - Linux: LUKS encryption
  - Windows: BitLocker

### Firewall Enforcement

- Keep firewall enabled at all times
- Use default-deny policies
- Regularly audit firewall rules

### User Privilege Separation

- Use separate accounts for different roles
- Implement sudo/sudoers for granular control
- Avoid shared admin accounts

### Logging & Monitoring

- Enable system logging
- Monitor authentication attempts
- Use tools like journalctl (Linux) or Event Viewer (Windows)

### Backup Strategy

- Regular automated backups
- Test backup restoration
- Store backups securely (encrypted, offsite)
- Follow 3-2-1 rule: 3 copies, 2 media types, 1 offsite

---

## Final OS Security Checklist

Use this checklist to verify your system's security hardening:

### Virtual Machine Setup

- [X] VM created with appropriate resources (2GB+ RAM, 20GB+ disk)
- [X] Ubuntu installed with strong user password
- [X] VirtualBox Guest Additions installed
- [X] System fully updated

### User Accounts

- [X] Root/admin access restricted to necessary tasks only
- [X] Strong passwords implemented for all accounts
- [X] Unnecessary user accounts removed
- [X] User groups configured appropriately

### File Permissions

- [X] Sensitive files have restrictive permissions (600/700)
- [X] Shared directories use appropriate group permissions
- [x] No world-writable files exist
- [x] Critical system files protected

### Access Control

- [x] sudo configured for admin tasks
- [x] SSH configured securely (if applicable)
- [x] Remote access restricted
- [X] Principle of least privilege applied

### Firewall

- [x] Firewall enabled and running
- [x] Default deny incoming policy set
- [x] Only necessary ports open
- [X] Rules audited regularly

### Services & Processes

- [x] Unnecessary services disabled
- [X] Running processes audited
- [X] System resource usage monitored
- [x] Automatic service startup minimized

### System Hardening

- [x] System updates applied regularly
- [x] Secure boot enabled (if supported)
- [X] Disk encryption implemented
- [x] Logging enabled and monitored

### Monitoring & Maintenance

- [x] Backup strategy implemented and tested
- [x] Security logs reviewed regularly
- [x] Antivirus/antimalware active (Windows)
- [x] Network traffic monitored

---

## Conclusion

Operating system security is a continuous process that requires understanding core concepts, implementing best practices, and maintaining vigilance. This document provides a foundation for securing Linux and Windows systems through:

- **Isolation**: Using virtual machines for safe learning
- **Access Control**: Proper user management and permissions
- **Defense in Depth**: Multiple security layers (firewalls, updates, monitoring)
- **Minimal Exposure**: Disabling unnecessary services and features

Remember that security is not a one-time task but an ongoing commitment. Regular audits, updates, and staying informed about threats are essential for maintaining system security.

**Key Takeaway**: The most secure system is one that follows the principle of least privilege, keeps software updated, and minimizes attack surface through careful configuration.

---
