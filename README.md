# TryHackMe: Friday Overtime (Write-Up)

A comprehensive write-up for the Friday Overtime room on TryHackMe. This challenge is a medium-difficulty Cyber Threat Intelligence (CTI) room for SOC1 analysts.

---

##  Scenario Overview
As a CTI Analyst at **PandaProbe Intelligence**, the objective is to investigate a high-priority incident ticket raised by **SwiftSpend Finance**. The investigation involves evaluating potentially malicious zipped attachments via the **DocIntel** platform and extracting actionable Indicators of Compromise (IoCs) using command-line tools and Open Source Intelligence (OSINT).

* **CTI Platform:** DocIntel
* **Malware Archive Password:** `Panda321!`

---

##  Investigation

### Task 1: Who shared the malware samples?
After logging into the DocIntel platform with the credentials provided in the virtual machine's browser I inspected the open ticket details from Swiftspend finance. The sender was shown here.
* **Answer:** `Oliver Bennett`

### Task 2: What is the SHA1 hash of the file “pRsm.dll” inside samples.zip?
Downloading the malicious zip file into a VM allowed me to safely extract and analyze the file contents. Using the password given I generated the SHA1 checksum directly via the command line:
```
cd /home/ericatracy/Downloads/
unzip samples.zip
# Password: Panda321!
sha1sum pRsm.dll
```
* **Answer:** `9d1ecbbe8637fed0d89fca1af35ea821277ad2e8`

### Task 3: Which malware framework utilizes these DLLs as add-on modules?
I entered the hash into VirusTotal. Using the extracted SHA1 hash revealed that there was a malicious plugin linked to the APT group Evasive Panda.
* **Answer:** `MgBot`

### Task 4: Which MITRE ATT&CK technique controls the audio hook functionality?
Analyzing technical threat reports of `MgBot` shows a plugin dedicated to capturing audio dynamically from compromised endpoints.
* **Answer:** `T1123`

### Task 5: What is the specific URL that downloaded the malicious executable?
Reviewing the threat intelligence details on DocIntel reveals an external staging URL hosted on a compromised or lookalike domain used to deliver the payload.
* **Answer:** `hxxp[://]update[.]browser[.]qq[.]com/qmbs/QQ/QQUrlMgr_QQ88_4296[.]exe`

### Task 6: What IP address does the malware communicate with?
Checking internal system logs and network indicators exposed the C2 server destination.
* **Answer:** `122[.]10[.]90[.]12`

### Task 7: What is the MD5 hash of the malware?
Going to VirusTotal and searching for 122.10.90.12 in the Relations tab lead me to the Android spyware dated 2025-07-20. Clicking this gave me the hash I needed.
* **Answer:** `951F41930489A8BFE963FCED5D8DFD79`

---

##  Key Takeaways & Defense Mitigation
This project showed me how to safely extract and analyze malware in  a controlled environment. It also gave me practice in investigating malware using various frameworks and tools like virustotal and MITREattack for discovery and mitigation purposes.

Defensive Recommendations:
1. Implement Strict Network DefensesBlock the Malicious Infrastructure: Immediately sinkhole the C2 IP 122.10.90.12 and block traffic to *.browser.qq.com at your firewall and proxy layers.
Implement DNS Filtering: Use protective DNS filtering to identify and block lookalike or newly registered domains used for staging payloads.
Enforce SSL/TLS Inspection: Decrypt and inspect outbound HTTPS traffic to look for anomalous user-agent strings or beaconing patterns tied to C2 channels.

2. Restrict DLL & Execution EnvironmentsEnforce Application Whitelisting: Use tools like AppLocker or Windows Defender Application Control (WDAC) to block unauthorized .dll files from executing out of user-writable directories (like C:\Users\...\Downloads or \AppData).
Monitor Service & Registry Auditing: MgBot often achieves persistence by installing malicious Windows Services or modifying Run registry keys.
Configure advanced auditing to log changes to:HKLM\SYSTEM\CurrentControlSet\ServicesHKCU\Software\Microsoft\Windows\CurrentVersion\Run3.

3.Endpoint Behavioral Detection (EDR)Monitor Process Injection: Set up your Endpoint Detection and Response (EDR) tool to alert on legitimate system processes (like explorer.exe or svchost.exe) spawning unusual network connections.
Audit Audio Device Access: Since the malware utilizes an audio hook (T1123), create behavioral rules to flag non-communication applications (like untrusted background DLLs) requesting access to system recording devices or microphones.
---
