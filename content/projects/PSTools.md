---
Date: 2025-09-06
Title: 'PowerShell Tools & Scripts'
Description: 'A collection of PowerShell tools and scripts I have developed for various tasks.'
---
Welcome to my collection of PowerShell tools and scripts. These are simple tools I've created with some System Administration tasks in mind. Feel free to explore, use, and modify them as needed.

## PSInfraTools  
A PowerShell module that provides random functions to manage and retrieve information about Windows systems. It includes functions to get Bitlocker keys from Active Directory, Disk usage, check backup status, User bad password attempts, and more.

Even though the module is not completely finished, they have proven to be useful in my daily tasks. I plan to continue adding more functions and improving existing ones. I also experimented with using Gitversion to automatically version the module based on my git history and commit keywords. It was also an opportunity to use Github actions to build and test the functions. I used the Pester framework to write unit tests.

[GitHub Repository](https://github.com/adrimus/PS-InfraTools)

You can find the functions under [PS-InfraTools Public folder](https://github.com/adrimus/PS-InfraTools/tree/main/source/Public), Below is a rundown of each function and what they can be used for:

---

### 🔐 `Get-ADBitlockerRecoveryPwd.ps1`
Retrieves BitLocker recovery keys from Active Directory.

**Use Cases:**
- Helpdesk recovery during user lockouts
- Compliance audits for encrypted endpoints
- Automated reporting of missing recovery keys

---

### 👤 `Get-ADUserBadPasswords.ps1`
Identifies AD users with bad password attempts or lockouts.

**Use Cases:**
- Detect brute-force attempts or compromised accounts
- Alerting for high-risk user behavior

---

### 💾 `Get-BackupJobInfo.ps1`
Checks status and metadata of backup jobs.

**Use Cases:**
- Daily health checks of backup infrastructure
- Alerting on failed or incomplete jobs

---

### 💽 `Get-DiskSizeInfo.ps1`
Retrieves disk size and usage stats across systems.

**Use Cases:**
- Capacity planning for storage upgrades
- Pre-migration disk audits
- Alerting on low disk space thresholds

---

### 🧠 `Get-GPPolicyData.ps1`
Extracts Group Policy Object (GPO) data.

**Use Cases:**
- Documenting applied policies for compliance
- Comparing GPO settings across environments

---

### 📦 `Get-InstalledSoftware.ps1`
Lists installed software on target machines.

**Use Cases:**
- Software inventory for licensing audits
- Detecting unauthorized installations
- Pre-upgrade compatibility checks

---

### 🔗 `FindGPOLinks.ps1`
Finds GPO links across AD structure.

**Use Cases:**
- Visualizing GPO application across OUs
- Identifying redundant or conflicting links
- Cleanup of legacy policy assignments

---