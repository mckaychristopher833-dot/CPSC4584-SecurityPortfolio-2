# Week 2 Lab Summary: Suspicious File Analysis
**Repository Path:** `maplewood-labs/week02-suspicious-file.md`  
**Incident:** INC-2026-0907-001  
**System:** MHS-C3-NRS-07  

---

## 1. Written Analysis

### Prompt 1 · File System Context
The file location `/home/nurse01/` represents the specific user home directory on workstation `MHS-C3-NRS-07`. From a security standpoint, this location indicates that any process creating or modifying files here operates with the privileges of the `nurse01` account. Since user home directories are writable by the account owner, an attacker with local or remote access to this account could drop files without needing root privileges. However, the location alone does not reveal how the file arrived (e.g., via network transfer, local physical access, or compromised credentials), nor does it confirm if the account itself is fully compromised. To investigate further, I would next examine `/var/log/auth.log` (or `/var/log/secure`) to check for off-hours login events, and `/tmp/` or `/var/tmp/` to look for temporary staging files or execution scripts.

### Prompt 2 · Permission String Analysis
The permission string `-rwxr--r--` breaks down into four parts: standard file type (`-`), owner permissions (`rwx` - read, write, execute), group permissions (`r--` - read-only), and other permissions (`r--` - read-only). The owner execute bit (`x`) is highly unusual for a file named `patient_notes.txt` because plain text documents are meant to be read, not executed as programs. This raises critical security questions: is this actually a binary or script disguised with a `.txt` extension, or was it created by an automated tool that sets default execute rights? Before drawing conclusions about its purpose or intent, an analyst must inspect the file's header/magic bytes using `file`, check its inode metadata with `stat`, and extract readable text with `strings` without executing the file.

### Prompt 3 · Escalation Decision
I would immediately escalate this alert to a Tier 2 analyst. The escalation is justified by key risk factors: the file was created at 3:14 AM outside clinical hours, carries an owner execute bit on a `.txt` file, sits on a workstation connected to the EHR network, and the assigned user denies creating it. Because Tier 1 authority restricts executing or modifying potential malware, Tier 2 escalation is required for sandbox analysis and deeper forensics. In the escalation report, I would include the exact file path, permissions, timestamp metadata, host network context, and the user's statement. Section 3 command practice demonstrates how non-destructive tools (`file`, `stat`, `strings`, `find`) allow an analyst to gather essential evidence safely without triggering execution.

---

## 2. Terminal Investigation Log

*Note: Commands were executed in the practice webshell environment (`CmckayGovst-academy@webshell`) using `README.txt`.*

### Entry 1 · Confirm Location and List Directory
* **Command:** `pwd && ls -la`
* **Output:**
  ```text
  /home/CmckayGovst-academy
  total 28
  drwxr-xr-x 2 CmckayGovst-academy CmckayGovst-academy 4096 Sep 11 20:00 .
  drwxr-xr-x 3 root                root                4096 Sep 11 19:00 ..
  -rw-r--r-- 1 CmckayGovst-academy CmckayGovst-academy  128 Sep 11 20:00 README.txt
