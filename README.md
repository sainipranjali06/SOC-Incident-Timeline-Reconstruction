# 🛡️ SOC Incident Timeline Reconstruction Project

### 📌 Project Overview
This project simulates a real-world cyber attack scenario to demonstrate the **SANS Incident Response process**: Identification, Containment, and Eradication. Acting as a SOC Analyst, I analyzed raw logs to reconstruct a specific attack timeline involving a Brute Force attempt, Privilege Escalation, and Data Exfiltration.

* **Role:** SOC Analyst (Blue Team)
* **Tools Used:** Python (Log Generation), Excel (Analysis), Windows Event Viewer
* **Outcome:** Successfully identified the attack vector, mapped the kill chain, and provided remediation steps.

---

### 🔍 The Scenario
A Windows Server (`10.0.0.15`) reported a spike in authentication failures. The goal was to determine if a breach occurred and, if so, what the attacker did.

**Evidence Analyzed:**
1.  **Windows Security Events (`windows_events.csv`):** Authentication logs (4624, 4625), Privilege use (4672), and Process Creation (4688).
2.  **Network Logs (`network_logs.csv`):** Firewall traffic logs showing Source/Dest IPs and Ports.

---

### 📊 Incident Timeline (Analysis Findings)

| Time (IST) | Phase | Event ID | Description |
| :--- | :--- | :--- | :--- |
| **12:10:09** | **Attack (Recon)** | 4625 | **Initial Access Attempt:** Detected high-volume failed login attempts (Brute Force) from Source IP `192.168.1.105` against user "Administrator". |
| **12:10:44** | **Compromise** | 4624 | **Account Breach:** Successful login for user "Administrator" from attacker IP. Security barrier breached after ~35 seconds of attempts. |
| **12:10:44** | **Privilege** | 4672 | **Escalation:** "Special Privileges" (Admin/System level) assigned to the compromised session immediately upon login. |
| **12:11:44** | **Execution** | 4688 | **Payload Delivery:** Malicious process spawned: `powershell.exe -enc ...`. The 60-second delay suggests an automated script or manual tool deployment. |
| **12:12:14** | **Exfiltration** | Net Log | **C2 Beaconing:** Outbound HTTPS traffic detected from Victim (`10.0.0.15`) to C2 Server (`45.33.22.11`). Repeated connections observed at 30-second intervals (Beaconing behavior). |

---

### 🚨 Technical Findings & Root Cause

* **Attack Vector:** RDP (Remote Desktop Protocol) Brute Force.
* **Weakness:** The "Administrator" account had a weak password that was cracked in under 40 seconds.
* **Malware Behavior:** The attacker utilized an obfuscated PowerShell script to establish persistence and communicate with a Command & Control (C2) server.

**Indicators of Compromise (IOCs):**
* **Attacker IP:** `192.168.1.105`
* **C2 Server IP:** `45.33.22.11`
* **Malicious Process:** `powershell.exe -enc`

---

### 🛡️ Remediation Recommendations

1.  **Immediate Action:** Block inbound traffic from `192.168.1.105` and outbound traffic to `45.33.22.11`.
2.  **Account Security:** Reset the Administrator password and enforce a complexity policy.
3.  **Network Hardening:** Disable RDP (Port 3389) exposure to the external internet or require VPN access.
4.  **Detection Logic:** Implement a SIEM rule to alert on:
    * `EventID 4625` > 5 attempts in 1 minute.
    * `EventID 4688` containing "powershell -enc".
