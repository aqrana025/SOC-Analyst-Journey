# 🔍 Lab: DNS Behaviour & Traffic Analysis

## 1. Objective
To baseline normal human DNS activity and contrast it against automated and malicious patterns, specifically identifying DNS brute-forcing and Command & Control (C2) beaconing.

## 2. Tools Used
* **Wireshark:** Traffic capture & Deep Packet Inspection (DPI).
* **Linux Terminal:** Generating queries via `dig`, `nslookup`, and `ping`.
* **Wazuh:** Monitoring telemetry and visualizing security events.

---

## 3. Analysis Steps (Baseline vs. Malicious)

### **Step 1: Normal Human Behaviour (The Baseline)**
* **Scenario:** Casual web browsing.
* **Observation:** DNS queries are sporadic and follow a diverse path (Google, GitHub, LinkedIn). Records are primarily `A` and `AAAA`.
* **Wireshark Filter:** `dns.flags.response == 0`
> **[INSERT SCREENSHOT 1 HERE]**

### **Step 2: Automated Behaviour (The Pulse)**
* **Scenario:** Rhythmic `ping` command to a single domain.
* **Observation:** The "Time" column in Wireshark shows queries at perfect 5-second intervals. This is non-human "heartbeat" traffic.
> **[INSERT SCREENSHOT 2 HERE]**

### **Step 3: DNS Brute-Force Attack**
* **Scenario:** Guessing subdomains to find hidden services.
* **Observation:** A massive spike in `NXDOMAIN` (Non-Existent Domain) responses in a very short window.
> **[INSERT SCREENSHOT 3 HERE]**

### **Step 4: C2 Beaconing (The Advanced Threat)**
* **Scenario:** A compromised host "checking in" to a C2 server.
* **Observation:** Constant communication to one specific, low-reputation domain using `TXT` records to hide data.
> **[INSERT SCREENSHOT 4 HERE]**

---

## 4. The "Diff" Table (SOC Cheat Sheet)

| Indicator | Human | Automated | Brute-Force | C2 Beaconing |
| :--- | :--- | :--- | :--- | :--- |
| **Frequency** | Random | Rhythmic | Rapid Fire | Persistent |
| **Record Type** | A / AAAA | A | A | TXT / NULL |

---

## 5. Conclusion
By comparing these four stages, I developed a Wazuh rule to detect more than 20 `NXDOMAIN` responses from a single internal IP within 1 minute, effectively catching DNS reconnaissance before it leads to a breach.
