# WN10-CC-000365
# 🎙️ Vulnerability Management Lab – WN10-CC-000365

**Title:** Windows Apps Must Not Be Activated by Voice While System Is Locked  
**STIG ID:** WN10-CC-000365  
**Compliance Standards:** DISA STIG, NIST 800-53, HIPAA, PCI DSS  
**Environment:** Azure VM + Tenable + PowerShell  
**Lab Type:** Vulnerability Simulation → Authenticated Scan → PowerShell Remediation → Verification

---

## 📋 Lab Objective

This lab demonstrates how to identify, simulate, and remediate a security misconfiguration where **Windows voice assistants (e.g., Cortana)** can be activated even when the device is locked. This violates STIG control **WN10-CC-000365**, which helps prevent unauthorized access via voice commands on unattended machines.

You will:
- Provision a Windows 10 VM in Azure  
- Simulate the vulnerability via PowerShell  
- Perform a Tenable STIG compliance scan  
- Remediate with registry modifications  
- Re-scan and validate compliance  

---

## 📁 Table of Contents

1. [Azure VM Setup](#azure-vm-setup)  
2. [Vulnerability Simulation](#vulnerability-simulation)  
3. [Tenable Scan Configuration](#tenable-scan-configuration)  
4. [Initial Vulnerability Scan](#initial-vulnerability-scan)  
5. [Remediation via PowerShell](#remediation-via-powershell)  
6. [Post-Remediation Verification](#post-remediation-verification)  
7. [Security Rationale](#security-rationale)  
8. [Appendix: PowerShell Commands](#appendix-powershell-commands)

---

## ☁️ Azure VM Setup

### 🔹 VM Provisioning

| Parameter           | Value                              |
|---------------------|-------------------------------------|
| VM Name             | `Win10-STIGLab-VoiceLock`           |
| OS                  | Windows 10 Pro (Gen 2)              |
| Size                | Standard D2s v3                     |
| Resource Group      | `vm-lab-voicelock`                  |
| Region              | Your nearest Azure region           |

### 🔹 Credential Standards

> ⚠️ Avoid using weak lab credentials like `labuser / Cyberlab123!`  
> ✅ Use strong, unique admin passwords for authentication testing

### 🔹 Network Security Group (NSG) Rules

| Port | Protocol | Direction | Status  |
|------|----------|-----------|---------|
| 3389 | RDP      | Inbound   | ✅ Allowed |
| 5985 | WinRM    | Inbound   | ✅ Allowed (optional) |
| Any  | All      | Inbound   | ❌ Denied (default)   |

### 🔹 Local Configuration

#### Disable Windows Firewall (Lab Only):
- Run `wf.msc` → Disable Domain, Private, and Public firewalls

#### Enable Remote PowerShell:
```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

---

## ⚠️ Vulnerability Simulation

### 🔸 Overview

This vulnerability allows voice-activated apps to run while the system is **locked**, potentially exposing sensitive information or allowing unauthorized actions via Cortana or similar apps.

### 🔸 Simulate the Misconfiguration

```powershell
# Simulate vulnerability: Allow voice-activated apps while locked
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "DisableLockWorkstation" -Value 0
```

> `0` = Voice apps **can** be activated on lock screen (non-compliant)  
> `1` = Voice apps **blocked** while locked (compliant)

📸 **Screenshot Placeholder:** `Screenshot_01_VoiceLock_Disabled.png`

---

## 🔍 Tenable Scan Configuration

### 🔸 Template: Advanced Network Scan

#### ✅ General Settings
- **Scan Name:** `STIG – Voice Activation While Locked`
- **Target:** Public IP of Azure VM

#### ✅ Discovery Settings
- Ping remote host  
- TCP port scans (full connect)  
- NetBIOS / SMB probes

#### ✅ Assessment Settings
- Enable:
  - Remote Registry
  - Admin Shares
  - Server Service
- Set mode to **Thorough**
- Use **Authenticated Scan** (local admin creds)

#### ✅ Compliance Content
- Upload audit file:  
  `DISA STIG – Microsoft Windows 10 v3r4.audit`

---

## 🧪 Initial Vulnerability Scan

Upon completing the scan in Tenable:

| STIG Control        | WN10-CC-000365                           |
|---------------------|-------------------------------------------|
| Description         | Prevent voice-activated apps while locked |
| Scan Result         | ❌ **Fail**                                |
| Detected Value      | `DisableLockWorkstation = 0`             |
| Required Value      | `DisableLockWorkstation = 1`             |

📸 **Screenshot Placeholder:** `Screenshot_02_Tenable_VoiceLock_Fail.png`

---

## 🛠️ Remediation via PowerShell

### 🔸 Secure Configuration

```powershell
# Enforce secure behavior: Disable voice activation on lock screen
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "DisableLockWorkstation" -Value 1
```

📸 **Screenshot Placeholder:** `Screenshot_03_PowerShell_Remediation_DisableVoiceApps.png`

> ✅ No reboot is required, but it is recommended to apply policy cleanly.

---

## 🔁 Post-Remediation Verification

### 🔸 PowerShell Validation

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "DisableLockWorkstation"
```

**Expected Output:**
```
DisableLockWorkstation : 1
```

### 🔸 Re-scan with Tenable

| STIG Control        | WN10-CC-000365 |
|---------------------|----------------|
| Compliance Status   | ✅ **Pass**     |
| Verified Setting    | `DisableLockWorkstation = 1` |

📸 **Screenshot Placeholder:** `Screenshot_04_Tenable_VoiceLock_Pass.png`

---

## 🔐 Security Rationale

Allowing voice apps to operate while the system is locked introduces risk of **unauthorized access** through voice interactions. Attackers could:

- Activate Cortana and access sensitive data  
- Execute voice-activated commands  
- Circumvent physical security controls

### Compliance Mapping

| Standard       | Requirement Summary                              |
|----------------|---------------------------------------------------|
| **DISA STIG**  | WN10-CC-000365 – Disable voice access on lock     |
| **NIST 800-53**| AC-11, SC-7 – Session Lock, Boundary Protection   |
| **HIPAA**      | §164.312(b) – Device & Session Safeguards         |
| **PCI DSS**    | Req. 8.1.8 – Session Lock                         |

---

## 🧼 Post-Lab Cleanup

- ✅ Reboot the VM (optional but recommended)  
- 🧹 Delete resource group to save cost:

```bash
az group delete --name vm-lab-voicelock --yes --no-wait
```

- 🔐 Remove Tenable credentials from scan profiles

---

## 📎 Appendix: PowerShell Commands

| Task                      | PowerShell Command |
|---------------------------|--------------------|
| Simulate vulnerability    | `Set-ItemProperty -Path HKLM:\...\System -Name DisableLockWorkstation -Value 0` |
| Apply secure configuration| `Set-ItemProperty -Path HKLM:\...\System -Name DisableLockWorkstation -Value 1` |
| Verify registry setting   | `Get-ItemProperty -Path HKLM:\...\System -Name DisableLockWorkstation` |
| Enable remote access      | `Set-ItemProperty -Path HKLM:\...\System -Name LocalAccountTokenFilterPolicy -Value 1` |

---

✅ **Lab Complete**

You’ve successfully demonstrated the detection, remediation, and compliance validation of **WN10-CC-000365** using **Azure, Tenable, and PowerShell**.  
Explore more Windows 10 STIG labs in the `/labs/` directory to continue your compliance mastery.
