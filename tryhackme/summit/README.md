# TryHackMe — Summit

## SOC Detection Engineering and the Pyramid of Pain

**Platform:** TryHackMe\
**Room:** [Summit](https://tryhackme.com/room/summit)\
**Category:** Blue Team / Security Operations / Detection Engineering\
**Completion:** November 2026\
**Status:** Completed — All 6 challenges\

---

## 1. Executive Summary

This lab focused on understanding and applying the **Pyramid of Pain** framework through a simulated adversarial engagement involving progressively evolving malware and detection techniques.

The exercise took place within PicoSecure, a simulated Security Operations Center (SOC) environment. An adversary named Sphinx conducted a series of simulated attacks, modifying malware samples and attack methodologies to evade previously implemented security controls.

Acting as a security analyst, I investigated malicious executable samples, analyzed network traffic and Windows system artifacts, and configured security controls using several tools, including:

- Malware Sandbox

- Hash Blocklist Management

- Firewall Rule Manager

- DNS Filter

- Sigma Rule Builder

- Simulated Security Information and Event Management (SIEM) system

The objective was to identify malicious activities and progressively develop more resilient detection mechanisms.

The investigation began with basic file-hash detection and advanced through IP blocking, DNS filtering, registry modification monitoring, network beaconing analysis, and detection of attacker tactics, techniques, and procedures (TTPs).

**Outcome:** Successfully completed all six challenges, validated the configured detection controls, and reached the top of the Pyramid of Pain.

The exercise demonstrated how behavior-based detection can improve defensive resilience against adversaries who frequently modify their tools, infrastructure, and malware.

## 2. Objectives

The primary objective was to develop practical SOC investigation and detection engineering skills.

Specific learning objectives included:

1. Analyze suspicious executable files using a malware sandbox.

2. Identify and document indicators of compromise (IOCs).

3. Configure file-hash detection and network security controls.

4. Investigate malicious Windows registry modifications.

5. Identify possible command-and-control (C2) beaconing through network log analysis.

6. Develop Sigma-style detection rules using endpoint and network telemetry.

7. Correlate suspicious activities with the MITRE ATT&CK framework.

8. Understand the strengths and limitations of different detection strategies.

## 3. Lab Environment

The exercise was conducted through the browser-based TryHackMe Summit environment.

| Component                  | Description                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------------- |
| Training platform          | TryHackMe                                                                                   |
| Lab environment            | PicoSecure                                                                                  |
| Operating system simulated | Windows 10 x64                                                                              |
| Threat actor               | Sphinx (simulated adversary)                                                                |
| Malware samples            | sample1.exe through sample6.exe                                                             |
| Security tools             | Malware Sandbox, Hash Manager, Firewall Manager, DNS Filter, Sigma Rule Builder             |
| Telemetry                  | Sandbox reports, network connection logs, Windows registry activity, command execution logs |
| Detection framework        | Pyramid of Pain                                                                             |
| Threat classification      | MITRE ATT&CK                                                                                |

The exercises were performed within an authorized simulation. No production systems were modified.

**Evidence:** `01-lab-introduction.png`, `02-security-tools.png`

## 4. Investigation Overview

The engagement followed six detection challenges, with each successive stage requiring a more sophisticated defensive approach.

| Challenge | Detection approach | Primary finding                                      | Result   |
| --------- | ------------------ | ---------------------------------------------------- | -------- |
| 1         | File hash          | Known malicious executable MD5                       | Blocked  |
| 2         | IP address         | Outbound connection to malicious C2 IP               | Blocked  |
| 3         | Domain name        | Suspicious domain and payload download               | Blocked  |
| 4         | Host artifact      | Windows Defender registry modification               | Detected |
| 5         | Tool behavior      | Periodic 97-byte network beacon                      | Detected |
| 6         | TTPs               | System discovery commands writing to a temporary log | Detected |

Each successful detection forced the simulated adversary to alter their technique or infrastructure, illustrating the increasing cost of evasion as detections moved higher up the Pyramid of Pain.

---

# 5. Detailed Investigation and Findings

## Challenge 1: Malware Detection Using File Hashes

### Objective

Identify a known malicious executable and prevent its execution by adding its MD5 hash to the security system's blocklist.

### Investigation

The engagement began with a message from Sphinx indicating that the initial attack would use an executable named `sample1.exe`.

I opened the PicoSecure Malware Sandbox and submitted the executable for analysis.

The sandbox generated a report containing file metadata, cryptographic hashes, and behavioral findings.

**Key observations:**

- File name: `sample1.exe`

- File type: Windows PE32+ executable

- Sandbox classification: `Trojan.Metasploit.A`

- Malicious behavior: Metasploit-related activity detected

- Suspicious behavior: Connection to an unusual port

The sandbox also reported environment-discovery activities, including checking the computer name, reading the machine GUID, and examining supported languages.

The following MD5 hash was identified:

`cbda8ae000aa9cbe7c8b982bae006c2a`

### Detection Implementation

Using PicoSecure's **Manage Hashes** tool, I created a new detection entry:

| Parameter           | Configuration                         |
| ------------------- | ------------------------------------- |
| Hash algorithm      | MD5                                   |
| Hash value          | `cbda8ae000aa9cbe7c8b982bae006c2a`    |
| Detection mechanism | Hash blocklist                        |
| Intended action     | Prevent execution of matching malware |

After submitting the hash, PicoSecure confirmed that `sample1.exe` had been prevented from executing.

### Result

**Successful detection and prevention.**

The malicious file was identified through its unique hash, and the simulator confirmed that execution was blocked.

Sphinx's next message explained that recompiling the malware generated a different hash, bypassing the original detection mechanism.

### Analysis and Lessons Learned

File hashes are useful for identifying exact copies of known malicious files. However, their primary limitation is sensitivity to file modifications.

Even minor changes to a binary can produce a completely different cryptographic hash.

Consequently, hash-based detection is useful for known threats but insufficient as the sole defense against malware that can be frequently rebuilt or modified.

**Evidence:**

- `03-sample1-analysis.png` — Malware sandbox analysis

- `04-md5-detection-rule.png` — Hash blocklist configuration

- `05-sample1-detection-success.png` — Successful prevention confirmation

- `challenge-1-completed.png` — Adversary response

---

## Challenge 2: Blocking Malicious Command-and-Control Communication

### Objective

Identify the malicious external IP address used by a modified malware sample and prevent outbound communication through an egress firewall rule.

### Investigation

Following the initial detection, Sphinx recompiled the malware and introduced a new executable, `sample2.exe`.

I submitted the new sample to Malware Sandbox and reviewed its behavioral and network activity reports.

The sandbox identified suspicious outbound communication.

**Malicious destination:**

| Indicator         | Value           |
| ----------------- | --------------- |
| Executable        | `sample2.exe`   |
| Destination IP    | `154.35.10.113` |
| Destination port  | `4444`          |
| Protocol observed | HTTP            |
| Connection type   | Outbound        |

The HTTP activity included a request to:

`http://154.35.10.113:4444/uvLk8YI32`

Additional connections were present in the sandbox report, including traffic associated with Microsoft infrastructure.

I focused on the unusual destination and port in the context of the sample's malicious behavioral findings.

### Detection Implementation

Using the **Firewall Rule Manager**, I configured an egress rule with the following parameters:

| Parameter      | Configuration   |
| -------------- | --------------- |
| Rule type      | Egress          |
| Source IP      | Any             |
| Destination IP | `154.35.10.113` |
| Action         | Deny            |

The rule was designed to block outbound communications from internal systems to the identified destination.

### Result

**Successful network communication prevention.**

PicoSecure confirmed that the rule prevented `sample2.exe` from connecting to the simulated attacker's command-and-control server.

### Analysis and Lessons Learned

IP-based blocking provides an effective short-term defensive measure when a malicious destination has been identified.

However, malicious infrastructure can be moved or replaced, allowing attackers to circumvent static IP-based rules.

The next stage of the exercise demonstrated this weakness when Sphinx switched to new IP addresses.

**Evidence:**

- `06-sample2-network-analysis.png` — Suspicious network connection identified

- `07-egress-firewall-rule-1.png` — Firewall rule configuration

- `07-egress-firewall-rule-2.png` — Firewall rule successfully saved

- `08-firewall-detection-success.png` — Adversary response confirming the block

---

## Challenge 3: Domain-Based Threat Detection and DNS Filtering

### Objective

Identify a malicious domain associated with the new malware sample and block communication at the DNS level.

### Investigation

After bypassing the IP-based detection by changing infrastructure, Sphinx introduced `sample3.exe`.

The malware sandbox report revealed suspicious network behavior, including connections to an external domain and the downloading of another executable.

The following domain was identified:

`emudyn.bresonicz.info`

The sandbox associated this domain with IP address `62.123.140.9`.

The report showed HTTP requests using ports `1337` and `80`, including a request for `backdoor.exe`.

The presence of a backdoor download, combined with the malicious behavior identified by the sandbox, indicated that the domain was associated with the simulated attacker's infrastructure.

### Detection Implementation

Using the **DNS Rule Manager**, I created a new DNS filtering rule.

| Parameter | Configuration           |
| --------- | ----------------------- |
| Rule name | Block Sphinx C2 Domain  |
| Category  | Malware                 |
| Domain    | `emudyn.bresonicz.info` |
| Action    | Deny                    |

Unlike the previous firewall rule, which targeted a single IP address, this control targeted the hostname used by the attacker.

### Result

**Successful malicious domain blocking.**

PicoSecure confirmed that the new DNS rule prevented `sample3.exe` from connecting to the simulated C2 server.

### Analysis and Lessons Learned

DNS filtering can provide more durable protection than blocking a single IP address because it can remain effective when a domain resolves to different IP addresses.

However, an attacker can still evade this control by registering or switching to new domains.

This challenge reinforced the importance of combining DNS filtering with other security monitoring techniques rather than relying exclusively on static indicators.

**Evidence:**

- `09-sample3-dns-analysis.png` — Sandbox network and DNS activity

- `10-dns-blocking-rule-1.png` — DNS rule configuration

- `10-dns-blocking-rule-2.png` — Successful DNS filtering confirmation

- `challenge-3-completed.png` — Adversary response

---

## Challenge 4: Detecting Malicious Windows Registry Modifications

### Objective

Detect malware attempting to disable Microsoft Defender real-time monitoring through a Windows registry modification.

### Investigation

With static hashes, IP addresses, and domains proving insufficient, the next challenge required investigating host-based artifacts.

I analyzed `sample4.exe` using the Malware Sandbox.

The behavioral report identified a high-risk activity:

**Disabling Windows Defender real-time monitoring.**

Further examination of the Registry Activity section revealed a modification performed by the malware process.

| Attribute    | Observed value                                                                |
| ------------ | ----------------------------------------------------------------------------- |
| Process      | `sample4.exe`                                                                 |
| Process ID   | `3806`                                                                        |
| Operation    | Registry write                                                                |
| Registry key | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection` |
| Value name   | `DisableRealtimeMonitoring`                                                   |
| New value    | `1`                                                                           |

This modification was consistent with an attempt to weaken endpoint defenses.

### Detection Implementation

I opened **Sigma Rule Builder**, selected Sysmon Event Logs, and chose Registry Modifications.

The rule was configured to monitor the identified registry key and value.

**Rule conditions:**

| Parameter      | Configuration                                                                 |
| -------------- | ----------------------------------------------------------------------------- |
| Log source     | Sysmon Event Logs                                                             |
| Event category | Registry Modifications                                                        |
| Registry key   | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection` |
| Registry name  | `DisableRealtimeMonitoring`                                                   |
| Value          | `1`                                                                           |
| ATT&CK tactic  | Defense Evasion (TA0005)                                                      |

After submitting the rule, PicoSecure generated and validated a Sigma-style detection.

The generated rule was titled:

**Modification of Windows Defender Real-Time Protection**

The simulator marked the rule as high severity.

### Result

**Successful detection rule validation and deployment.**

PicoSecure confirmed that the rule had been validated and automatically deployed to the simulated SIEM.

### MITRE ATT&CK Mapping

The observed behavior aligns with:

**Tactic:** Defense Evasion (TA0005)

**Technique:** [T1562.001 — Impair Defenses: Disable or Modify Tools](https://attack.mitre.org/techniques/T1562/001/)

The attacker attempted to reduce the effectiveness of security monitoring by modifying a protective system setting.

### Analysis and Lessons Learned

Host-based detection provides visibility into malicious actions that may remain consistent even when an attacker changes network infrastructure or executable hashes.

Monitoring sensitive registry modifications can therefore offer stronger protection than relying exclusively on basic IOCs.

However, legitimate administrative activity can also modify security settings. In a production environment, alerts should be correlated with process lineage, user identity, authorization, and other contextual information.

**Evidence:**

- `11-sample4-registry-analysis-1.png` — Sandbox behavioral analysis

- `11-sample4-registry-analysis-2.png` — Registry modification evidence

- `12-registry-sigma-rule-1.png` — Initial Sigma Rule Builder view

- `12-registry-sigma-rule-2.png` — Detection rule configuration

- `13-registry-rule-validation.png` — Successfully validated rule

- `challenge-4-completed.png` — Adversary response

---

## Challenge 5: Detecting Command-and-Control Beaconing

### Objective

Identify recurring suspicious network activity and create a behavioral detection rule capable of identifying potential C2 beaconing.

### Investigation

Sphinx changed the attack methodology again, attempting to avoid the host artifact detections established in the previous challenge.

The adversary provided a new sample, `sample5.exe`, and an attachment containing outbound network connection logs.

I opened `outgoing_connections.log` in the attachment viewer and examined the traffic patterns.

The logs included numerous outbound connections, some displaying varying destinations, packet sizes, and timestamps.

However, one pattern was highly consistent.

**Suspicious beaconing characteristics:**

| Attribute               | Observed value   |
| ----------------------- | ---------------- |
| Source IP               | `10.10.15.12`    |
| Repeated destination IP | `51.102.10.19`   |
| Destination port        | `443`            |
| Connection size         | `97 bytes`       |
| Recurrence interval     | `1,800 seconds`  |
| Recurrence frequency    | Every 30 minutes |

The connections occurred repeatedly at predictable intervals, including 09:00, 09:30, 10:00, and subsequent half-hour timestamps.

This regularity distinguished the activity from the other connections recorded in the log.

### Detection Implementation

I configured a detection rule using **Sigma Rule Builder → Sysmon Event Logs → Network Connections**.

| Parameter     | Configuration                |
| ------------- | ---------------------------- |
| Remote IP     | Any                          |
| Remote Port   | Any                          |
| Size          | 97 bytes                     |
| Frequency     | 1800 seconds                 |
| ATT&CK tactic | Command and Control (TA0011) |

The detection deliberately avoided hardcoding the remote IP address or port.

Instead, it focused on the repeated connection characteristics observed in the logs.

The generated rule was titled:

**Alert on Suspicious Beacon Network Connections**

### Result

**Successful behavioral detection rule validation and deployment.**

PicoSecure confirmed that the rule had been validated and deployed.

### MITRE ATT&CK Mapping

**Tactic:** Command and Control (TA0011)

The observed periodic communication pattern was consistent with potential C2 beaconing, although the log alone did not identify a specific ATT&CK communication technique or prove the contents of the traffic.

### Analysis and Lessons Learned

Beaconing detection is valuable because it focuses on how a malicious tool communicates rather than relying solely on destination addresses.

Indicators such as periodic timing, consistent connection sizes, and repeated communication patterns may reveal automated activity.

An attacker changing IP addresses or ports would not necessarily bypass a detection based on those behavioral characteristics.

In practice, legitimate software can also generate periodic traffic, including health checks, monitoring agents, and scheduled synchronization processes.

Consequently, beaconing detections require contextual analysis and false-positive tuning.

**Evidence:**

- `14-network-beaconing-logs.png` — Outbound connections demonstrating the repeating pattern

- `15-beaconing-sigma-rule-1.png` — Detection configuration

- `15-beaconing-sigma-rule-2.png` — Completed rule parameters

- `16-beaconing-detection-success.png` — Validated and deployed rule

- `challenge-5-completed.png` — Adversary response

---

## Challenge 6: Detecting Attacker Tactics, Techniques, and Procedures

### Objective

Identify recurring attacker behaviors across multiple malware samples and detect information-gathering and local staging activity.

### Investigation

For the final challenge, Sphinx introduced `sample6.exe` and explained that previous detections had become increasingly expensive to evade.

Rather than relying on a particular executable, IP address, or network pattern, the final task required identifying the actions the attacker consistently performed after gaining access to a compromised system.

The adversary provided an attachment named `commands.log`, containing previously executed Windows commands.

I reviewed the command history and identified multiple system discovery activities.

**Observed commands included:**

- `dir C:\` — Enumerates the root directory.

- `dir "C:\Program Files"` — Enumerates installed application directories.

- `net localgroup administrator` — Queries local group information.

- `ver` — Retrieves the Windows version.

- `systeminfo` — Collects operating system and hardware information.

- `ipconfig /all` — Retrieves network configuration details.

- `netstat -ano` — Lists active network connections and associated process identifiers.

- `net start` — Lists started Windows services.

An important common characteristic was that the attacker redirected command output to the same temporary file:

`%temp%\exfiltr8.log`

The commands used the `>>` operator to append information to that file.

For example:

`systeminfo >> %temp%\exfiltr8.log`

`ipconfig /all >> %temp%\exfiltr8.log`

`netstat -ano >> %temp%\exfiltr8.log`

This suggested a repeated procedure for collecting and staging host information.

### Detection Implementation

I created a new Sigma-style rule using:

**Sysmon Event Logs → File Creation and Modification**

The rule was configured as follows:

| Parameter              | Configuration                                                    |
| ---------------------- | ---------------------------------------------------------------- |
| File path              | `%temp%`                                                         |
| File name              | `exfiltr8.log`                                                   |
| ATT&CK tactic selected | Exfiltration (TA0010)                                            |
| Detection purpose      | Identify creation or modification of the attacker's staging file |

After validation, PicoSecure generated a rule titled:

**Alert on Potential Exfiltration through File Creation or Modification**

The generated detection was marked as high severity.

### Result

**Successful detection and completion of the Summit challenge.**

PicoSecure confirmed the rule had been validated and automatically deployed.

The simulator displayed a completion message, and Sphinx's final email, titled *I'm Giving Up*, acknowledged that the defensive controls had reached the top of the Pyramid of Pain.

### MITRE ATT&CK Mapping

Although the simulator required the Exfiltration (TA0010) label for this rule, the command history provides more direct evidence of several Discovery techniques.

| Observed command or action | ATT&CK technique                                                                             |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| `systeminfo`, `ver`        | [T1082 — System Information Discovery](https://attack.mitre.org/techniques/T1082/)           |
| `ipconfig /all`            | [T1016 — System Network Configuration Discovery](https://attack.mitre.org/techniques/T1016/) |
| `netstat -ano`             | [T1049 — System Network Connections Discovery](https://attack.mitre.org/techniques/T1049/)   |
| Directory enumeration      | [T1083 — File and Directory Discovery](https://attack.mitre.org/techniques/T1083/)           |
| `net start`                | [T1007 — System Service Discovery](https://attack.mitre.org/techniques/T1007/)               |

The observed redirection of command output to a temporary file also supports an assessment of local data staging or collection.

**Important distinction:** The logs demonstrate information gathering and local file writing, but do not independently show data leaving the compromised system. Therefore, actual exfiltration cannot be confirmed from these records alone.

### Analysis and Lessons Learned

This challenge demonstrated why attacker TTPs sit at the top of the Pyramid of Pain.

Adversaries may quickly change hashes, network infrastructure, and malware implementations. Changing established operating procedures can be substantially more difficult because it may require new tooling, training, or operational planning.

The detection implemented in this exercise targeted the specific staging filename. Although successful within the simulation, a production-grade behavioral detection would need to go further.

A stronger rule would correlate the execution of multiple discovery commands with their common output destination, process relationships, timing, and subsequent network behavior.

Such an approach would remain more effective if the attacker renamed the staging file.

**Evidence:**

- `17-attacker-commands-log.png` — Attacker command history

- `18-file-creation-sigma-rule-1.png` — Detection builder configuration

- `18-file-creation-sigma-rule-2.png` — File creation and modification rule

- `19-ttp-detection-success.png` — Successful validation and completion confirmation

- `20-summit-completed.png` — Final adversary message

---

# 6. Consolidated Indicators of Compromise

The following indicators were identified during the simulated investigations.

| Indicator type            | Value                              | Associated challenge |
| ------------------------- | ---------------------------------- | -------------------- |
| MD5                       | `cbda8ae000aa9cbe7c8b982bae006c2a` | 1                    |
| Malicious IP              | `154.35.10.113`                    | 2                    |
| Port                      | `4444`                             | 2                    |
| Malicious domain          | `emudyn.bresonicz.info`            | 3                    |
| Associated IP             | `62.123.140.9`                     | 3                    |
| Downloaded payload        | `backdoor.exe`                     | 3–4                  |
| Registry value            | `DisableRealtimeMonitoring`        | 4                    |
| Registry data             | `1`                                | 4                    |
| Recurring connection size | `97 bytes`                         | 5                    |
| Beacon interval           | `1800 seconds`                     | 5                    |
| Staging file              | `%temp%\exfiltr8.log`              | 6                    |

These indicators are specific to the TryHackMe simulation and should not be treated as independently verified real-world threat intelligence.

# 7. Detection Strategy and Pyramid of Pain Analysis

The exercise demonstrated the progression from easily changed indicators toward more behavior-oriented detections.

| Pyramid level  | Detection method                        | Attacker evasion approach                           | Defensive lesson                                           |
| -------------- | --------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------- |
| File hashes    | MD5 blocklist                           | Recompile or modify the binary                      | Exact hashes have limited resilience                       |
| IP addresses   | Egress firewall rule                    | Change hosting provider or server IP                | IP blocking is infrastructure-dependent                    |
| Domain names   | DNS filtering                           | Register new domains                                | Domain controls require maintenance                        |
| Host artifacts | Registry modification monitoring        | Change malware actions or avoid monitored artifacts | Endpoint telemetry provides behavioral context             |
| Tools          | Recurring network beacon detection      | Redesign communication behavior                     | Tool characteristics are harder to change                  |
| TTPs           | Detection of host discovery and staging | Alter operating procedures                          | Behavior-based detection can impose greater attacker costs |

The lab reinforced that no individual control provides comprehensive protection.

Effective security monitoring benefits from combining static IOCs, endpoint telemetry, network analytics, and behavioral correlation.

# 8. Technical Limitations and Real-World Considerations

Although Summit provided an effective introduction to detection engineering, the PicoSecure environment simplified several components that would require additional implementation in a real SOC.

### 8.1 Sigma Rule and Sysmon Compatibility

The simulator automatically generated and validated Sigma-style rules based on user-selected fields.

However, these rules should not be assumed to be directly deployable to a production SIEM without review or modification.

For example:

- The registry monitoring rule referenced Windows Event ID 4663, whereas native Sysmon registry-value modifications are typically recorded under Event ID 13.

- The final file monitoring rule referenced Sysmon Event ID 2, which represents file creation time changes. Sysmon Event ID 11 is more appropriate for monitoring file creation.

- Sysmon Event ID 3 records network connections, but packet size and recurring connection frequency require additional telemetry or correlation logic.

These differences highlight the distinction between the lab's simplified detection builder and real detection-engineering implementations.

### 8.2 Alert Quality and False Positives

Several configured detections could generate false positives in an operational environment.

Examples include legitimate administrative changes to Microsoft Defender settings, automated applications generating periodic network traffic, and authorized scripts collecting system information.

Production detections require appropriate thresholds, context, baselines, and exception handling.

### 8.3 Detection Validation

The lab confirmed successful rule validation and simulated deployment.

However, the screenshots do not establish testing against a production SIEM or real enterprise telemetry.

Additional validation would be required to verify detection accuracy, reliability, performance, and false-positive rates in a live environment.

### 8.4 Opportunities for Improvement

The detection mechanisms could be enhanced by:

1. Incorporating SHA-256 hashes and file reputation intelligence into malware triage.

2. Correlating network destinations with DNS activity and process execution.

3. Monitoring sensitive registry modifications using appropriate Windows and Sysmon event sources.

4. Detecting beaconing with time-series analysis, interval variation, and network-flow or proxy telemetry.

5. Developing broader behavioral detection logic for suspicious command sequences and staging activity.

6. Testing detection rules against both malicious and benign activity to evaluate false positives.

# 9. Skills Developed

Completing this exercise strengthened several practical cybersecurity skills.

| Skill area              | Hands-on experience                                                    |
| ----------------------- | ---------------------------------------------------------------------- |
| Malware analysis        | Investigating executable metadata and sandbox behavioral results       |
| Threat intelligence     | Extracting and assessing file, network, DNS, and host-based IOCs       |
| Network security        | Configuring egress firewall rules and DNS blocking                     |
| Endpoint monitoring     | Identifying suspicious Windows registry modifications                  |
| Log analysis            | Recognizing periodic C2-like communication patterns                    |
| Detection engineering   | Creating and validating Sigma-style detection logic                    |
| MITRE ATT&CK            | Mapping observed behavior to defensive threat classifications          |
| SOC investigations      | Moving from initial evidence to detection configuration and validation |
| Technical documentation | Capturing and organizing investigation evidence                        |

# 10. Key Takeaways

The most important lesson from this exercise was that effective detection engineering should move beyond identifying known malicious files or infrastructure.

Hashes, IP addresses, and domains are valuable indicators, but attackers can often change them quickly.

Host artifacts and recurring communication patterns can provide stronger detection opportunities because they reflect what malware does rather than simply how it is identified.

The final exercise emphasized the value of understanding attacker procedures.

By analyzing the sequence of information-gathering commands and their shared output location, I was able to identify a repeated activity that was independent of the malware's executable hash or command-and-control destination.

The lab also reinforced the importance of interpreting detection evidence carefully, distinguishing confirmed behavior from inferred attacker intent, and understanding the limitations of simulated security tools.

# 11. Final Results and Conclusion

**All six Summit challenges were completed successfully.**

The final Sigma-style rule was validated, and PicoSecure displayed its completion confirmation.

Sphinx's final message acknowledged that the defender had progressively implemented detections at every level of the Pyramid of Pain.

The exercise provided practical experience with malware sandbox analysis, IOC identification, network security controls, endpoint monitoring, and behavioral detection engineering.

**Overall conclusion:** A layered defensive strategy combining multiple sources of telemetry and progressively more behavior-oriented detection mechanisms is more resilient than relying exclusively on static indicators of compromise.

This lab serves as an introductory SOC and detection engineering portfolio exercise, providing a foundation for future hands-on investigations using production-style SIEM tools and endpoint telemetry.

---

# Appendix A: Screenshot Evidence Index

A total of 31 screenshots were collected to document the investigation, detection implementation, and successful outcomes.

| Section      | Evidence files                                                                                                                                                                                             |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Introduction | `01-lab-introduction.png`, `02-security-tools.png`                                                                                                                                                         |
| Challenge 1  | `03-sample1-analysis.png`, `04-md5-detection-rule.png`, `05-sample1-detection-success.png`, `challenge-1-completed.png`                                                                                    |
| Challenge 2  | `06-sample2-network-analysis.png`, `07-egress-firewall-rule-1.png`, `07-egress-firewall-rule-2.png`, `08-firewall-detection-success.png`, `challenge-2-completed.png`                                      |
| Challenge 3  | `09-sample3-dns-analysis.png`, `10-dns-blocking-rule-1.png`, `10-dns-blocking-rule-2.png`, `challenge-3-completed.png`                                                                                     |
| Challenge 4  | `11-sample4-registry-analysis-1.png`, `11-sample4-registry-analysis-2.png`, `12-registry-sigma-rule-1.png`, `12-registry-sigma-rule-2.png`, `13-registry-rule-validation.png`, `challenge-4-completed.png` |
| Challenge 5  | `14-network-beaconing-logs.png`, `15-beaconing-sigma-rule-1.png`, `15-beaconing-sigma-rule-2.png`, `16-beaconing-detection-success.png`, `challenge-5-completed.png`                                       |
| Challenge 6  | `17-attacker-commands-log.png`, `18-file-creation-sigma-rule-1.png`, `18-file-creation-sigma-rule-2.png`, `19-ttp-detection-success.png`, `20-summit-completed.png`                                        |

**Evidence publication status:** All 31 screenshots were captured and prepared in a separate GitHub upload package with challenge flags redacted. The image files are not yet in the repository; the paths listed below are the intended `screenshots/` filenames.

# References

1. [TryHackMe — Summit](https://tryhackme.com/room/summit)

2. [MITRE ATT&CK Framework](https://attack.mitre.org/)

3. [Microsoft Sysinternals — Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

4. [Sigma Detection Rule Specification](https://sigmahq.io/)

---

*This report documents an authorized cybersecurity training exercise conducted in a simulated environment. The observations, controls, and results described are specific to the TryHackMe Summit lab.*
