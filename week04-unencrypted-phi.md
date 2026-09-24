# Week 4: Unencrypted Patient Records on a Shared Drive
**Course:** CPSC 4584 | Special Topics in Information Security
**Date:** September 21, 2026
**Analyst:** Christopher McKay
**Audit ID:** AUD-2026-0921-001

---

## Incident Summary

During a routine IT infrastructure audit, a network administrator discovered 8,247 unencrypted patient records sitting in a shared folder (`PATIENT_DATA_ARCHIVE`) on a clinical server. The folder contained 847 Excel and CSV files accessible to all authenticated users on the clinical network, with no retained access logs prior to discovery.

---

## HIPAA Compliance Assessment

| Requirement | Status | Finding |
|-------------|--------|---------|
| Encryption at Rest | REQUIRES REVIEW | Encryption is an addressable implementation specification under HIPAA. Leaving 8,247 patient records in plaintext without an equivalent documented safeguard violates this requirement and creates severe risk. |
| Access Controls | CONTROL FAILURE | Read and write access permissions were granted broadly to all authenticated users across four clinic locations instead of restricting access to personnel with a valid job-related need to know. |
| Audit Controls | CONTROL FAILURE | No access logs were retained for the folder prior to discovery, preventing analysts from reconstructing who accessed, copied, or modified the patient records. |

---

## Cryptographic Controls Evaluated

**Base64 encoding:** Not encryption. It is a reversible encoding scheme designed for data translation, providing zero confidentiality protection for PHI.
**Caesar cipher:** Obsolete classical cipher. It relies on simple letter substitution with only 25 possible keys, making it trivial to break via brute force or frequency analysis.
**Modern encryption at rest:** Modern AES-256 symmetric encryption at the file, volume, or disk level. Maplewood should evaluate AES-256 to ensure that stolen or exposed storage media cannot be read without the authorized decryption key.

---

## Hashing Commands Practiced

| Command | Purpose | Output Length |
|---------|---------|---------------|
| echo -n "..." \| sha256sum | Demonstrated generating a fixed-length cryptographic digest from string input and verified the avalanche effect when input data changes. | 256 bits (64 hex characters) |
| echo -n "..." \| md5sum | Compared a legacy, broken hash algorithm against SHA-256 to highlight why MD5 is vulnerable to collision attacks and deprecated. | 128 bits (32 hex characters) |
| sha256sum .bashrc | Calculated a file-level hash to demonstrate how security analysts establish an integrity baseline to detect file tampering. | 256 bits (64 hex characters) |

---

## Escalation Summary

An IT audit confirmed that 8,247 unencrypted patient records across 847 files were broadly accessible to all authenticated users with zero access logging retained. The full extent of unauthorized access remains unknown due to missing logs. All original files have been access-restricted and preserved in place without modification. Formal breach determinations, risk assessments, and regulatory notification decisions require escalation to executive leadership, the CISO, and legal counsel.

---
*CPSC 4584 | Governors State University | Fall 2026*
