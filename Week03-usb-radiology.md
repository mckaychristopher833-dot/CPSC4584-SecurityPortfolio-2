# Maplewood Health System · SOC Incident Analysis
**Analyst:** Christopher McKay
**Course:** CPSC 4584 - Computer Security  
**Date:** September 14, 2026  
**Incident ID:** INC-2026-0914-001 (Unauthorized USB Drive)

---

### Section 1: Incident Summary
* **Discovery Location:** Workstation MHS-RAD-WS-03, Radiology Imaging Suite, Clinic 3
* **Discovery Time:** Saturday, September 13, 2026 at 11:47 AM
* **Evidence Tag:** MHS-USB-2026-0913-001
* **Device Details:** Unmarked USB flash drive; no manufacturer branding or serial labels.
* **Current Status:** In SOC custody stored in a sealed anti-static bag; workstation isolated from clinical network.

---

### Section 2: Written Analysis

#### Prompt 1 · Chain of Custody
Documenting every custodian, transfer, and action in the chain of custody log is essential to ensure evidence integrity, establish legal admissibility, and verify that no unauthorized tampering occurred. If the facilities technician had connected the drive to another computer to inspect it, it would have created severe questions regarding evidence alteration: operating systems automatically write or update hidden system files (such as `.DS_Store`, `Thumbs.db`, or `System Volume Information`), alter file access timestamps (`atime`), and potentially trigger autorun scripts or malware execution. This uncertainty would make it impossible to prove whether specific files or metadata originated from an external threat actor or were artifacts generated during the technician's inspection, thereby destroying confidence in subsequent legal proceedings or internal HR actions.

#### Prompt 2 · Encoding and Investigation
The string `Y3VybCAtcyAtbyAvZGV2L251bGw=` is encoded using **Base64**. It is identified by its alphanumeric character set (A–Z, a–z, 0–9, `+`, `/`), a length in multiples of four, and trailing equal sign padding (`=`). Decoding the string using the approved standard method (`echo "Y3VybCAtcyAtbyAvZGV2L251bGw=" | base64 -d`) yields:

`curl -s -o /dev/null`

This command instructs `curl` to run silently (`-s`) and discard fetched output to `/dev/null`. From this fragment alone, an analyst cannot conclude that actual network exfiltration or malicious outbound connections occurred on workstation MHS-RAD-WS-03. The string lacks a target URL or domain, represents an isolated command fragment, and was analyzed within a practice environment rather than confirmed as executed memory or disk evidence from the physical Maplewood USB drive.

#### Prompt 3 · Escalation Decision
**Decision: Yes, immediately escalate to Tier 2 for deeper forensic investigation.**

* **Supporting Facts:** An unauthorized, unmarked physical USB device was found plugged into workstation MHS-RAD-WS-03 in a restricted clinical environment (Radiology imaging suite). The workstation has direct logical access to high-sensitivity systems, specifically the Maplewood EHR and PACS imaging infrastructure. Card key access logs identify six individuals in the area during the 72-hour window prior to discovery.
* **Unanswered Questions:** What exact files or payloads exist on the physical USB drive (as forensic acquisition/mounting has not yet occurred)? Did any local script execution or unauthorized activity take place during overnight hours, given the documented gaps in overnight monitoring coverage? Were card keys shared, duplicated, or misused by unauthorized staff or vendors?
* **Relevance of Practice:** The Section 3 practice prepares Tier 1 analysts to safely examine non-executable string outputs, identify obfuscated commands (such as Base64-encoded command-line arguments), and interpret low-level file structure signatures (`xxd` hex dumps, magic bytes) without executing unknown binaries or compromising host safety.

---

### Section 3: Terminal Investigation Log

#### Terminal Entry 1 · Encode a String Using Base64
* **Command:** `echo -n "Maplewood SOC Tier 1 Investigation" | base64`
* **Output:** `TWFwbGV3b29kIFNPQyBUaWVyIDEgSW52ZXN0aWdhdGlvbg==`
* **Analyst Observation:** Converts plaintext into Base64 6-bit index representation. Trailing `==` indicates binary padding to meet 24-bit alignment boundaries.

#### Terminal Entry 2 · Decode a Base64 String
* **Command:** `echo "Y3VybCAtcyAtbyAvZGV2L251bGw=" | base64 -d`
* **Output:** `curl -s -o /dev/null`
* **Analyst Observation:** Decodes Base64 string into a UNIX command fragment (`curl`) with silent (`-s`) and output redirection (`-o /dev/null`) flags. Decoding performed purely as a string conversion without command execution.

#### Terminal Entry 3 · Inspect a File Using xxd
* **Command:** `xxd -g 1 /bin/ls | head -n 5`
* **Output:**
  ```text
  00000000: 7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00  .ELF............
  00000010: 02 00 3e 00 01 00 00 00 70 10 00 00 00 00 00 00  ..>.....p.......
  00000020: 40 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  @...............
  00000030: 00 00 00 00 40 00 38 00 09 00 40 00 1c 00 1b 00  ....@.8...@.....
  00000040: 06 00 00 00 04 00 00 00 40 00 00 00 00 00 00 00  ........@.......
