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
#### **The Observation**
In this initial capture, I am establishing a baseline of standard outbound DNS queries. The traffic originates from an internal host (`192.168.10.139`) and is directed to the local DNS resolver (`192.168.10.2`). 

#### **Technical Breakdown**
* **The Filter:** I applied the display filter `dns.flags.response == 0` to isolate client queries and eliminate response noise.
* **Query Diversity:** As seen in the "Info" column, queries are distributed across multiple legitimate domains (`mozilla.net`, `google.com`, `youtube.com`). This high entropy and variety are primary indicators of **human-generated browsing activity**.
* **Record Types:** The traffic consists of standard `A` (IPv4) and `AAAA` (IPv6) records, which is expected behavior for modern OS address resolution.
* **Timing (Aperiodicity):** The "Time" column reveals irregular intervals (jumps between 4s, 7s, and 12s). The lack of a rhythmic "heartbeat" suggests manual user interaction rather than an automated script or bot.

#### **Evidence**
![Normal DNS Baseline](.<img width="1501" height="978" alt="baselin DNS" src="https://github.com/user-attachments/assets/b2b84ba2-50f0-4d85-8197-7a2a18977388" />
/normal%20dns.flags%20re.jpg)
*Figure 1: Wireshark capture highlighting diverse, aperiodic DNS queries.*

---

#### **🕵️ SOC Analyst Technical Insight**
Looking at the packet details for the selected frame in **normal dns.flags re.jpg**, the **Total Length is 81 bytes**. Standard, non-malicious DNS queries are typically small. If this were **DNS Tunneling** for C2 communication, we would expect to see significantly larger packet lengths (e.g., 200+ bytes) as the attacker attempts to "stuff" encoded data into the DNS query fields.![Uploading baselin DNS.jpg…]()


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
