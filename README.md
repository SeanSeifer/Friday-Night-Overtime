# TryHackMe: Friday Overtime (Write-Up)

A comprehensive write-up for the **[Friday Overtime](https://tryhackme.com)** room on TryHackMe. This challenge is a medium-difficulty Cyber Threat Intelligence (CTI) room included in the **SOC Level 1** learning path.

---

## 📌 Scenario Overview
As a CTI Analyst at **PandaProbe Intelligence**, the objective is to investigate a high-priority incident ticket raised by **SwiftSpend Finance**. The investigation involves evaluating potentially malicious zipped attachments via the **DocIntel** platform and extracting actionable Indicators of Compromise (IoCs) using command-line tools and Open Source Intelligence (OSINT).

* **CTI Platform:** DocIntel
* **Malware Archive Password:** `Panda321!`

---

## 🔍 Investigation & Walkthrough

### Task 1: Who shared the malware samples?
Log into the DocIntel platform with the credentials provided in the virtual machine's browser. Inspect the open incident ticket details from SwiftSpend Finance to find the sender's identity.
* **Answer:** `Oliver Bennett`

### Task 2: What is the SHA1 hash of the file “pRsm.dll” inside samples.zip?
Download `samples.zip` inside the VM environment, open a terminal, and extract it using the discovered password. Generate the SHA1 checksum directly via the command line:
```bash
cd /home/ericatracy/Downloads/
unzip samples.zip
# Password: Panda321!
sha1sum pRsm.dll
```
* **Answer:** `9d1ecbbe8637fed0d89fca1af35ea821277ad2e8`

### Task 3: Which malware framework utilizes these DLLs as add-on modules?
Pivoting to OSINT platforms (like VirusTotal or Threat Intelligence blogs) using the extracted SHA1 hash reveals that this specific `.dll` is a modular plug-in linked to the **Evasive Panda** APT group.
* **Answer:** `MgBot`

### Task 4: Which MITRE ATT&CK technique controls the audio hook functionality?
Analyzing technical threat reports detailing the `MgBot` modular framework reveals a plugin dedicated to capturing audio dynamically from compromised endpoints.
* **Answer:** `T1123`

### Task 5: What is the specific URL that downloaded the malicious executable?
Reviewing the threat intelligence details on DocIntel reveals an external staging URL hosted on a compromised or lookalike domain used to deliver the payload.
* **Answer:** `hxxp[://]update[.]browser[.]qq[.]com/qmbs/QQ/QQUrlMgr_QQ88_4296[.]exe`

### Task 6: What IP address does the malware communicate with?
Cross-referencing the internal system logs and network indicators exposes the hardcoded Command and Control (C2) server destination.
* **Answer:** `122.10.90.12`

### Task 7: What is the MD5 hash of the malware?
Generate the MD5 hash from the extracted executable or pull the artifact metrics straight from the DocIntel ticket summary.
```bash
md5sum <malicious_file>.exe
```
* **Answer:** `951F41930489A8BFE963FCED5D8DFD79`

---

## 🛡️ Key Takeaways & Defense Mitigation
* **Malware Modularization:** Modern APT campaigns leverage frameworks like `MgBot` to dynamically push specific functional modules (like audio recording or keylogging) only when needed, minimizing their on-disk signature.
* **Hash-Based Pivoting:** Defensive teams can completely unmask an unknown campaign simply by pivoting from a single isolated IoC (like a DLL hash) to globally published threat intelligence feeds.

---
*Disclaimer: This write-up is intended solely for educational purposes as part of cybersecurity training on TryHackMe.*
