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
<img width="1501" height="978" alt="baselin DNS" src="https://github.com/user-attachments/assets/b2b84ba2-50f0-4d85-8197-7a2a18977388" />

*Figure 1: Wireshark capture highlighting diverse, aperiodic DNS queries.*

#### **The Expert "Think" Points (SOC Recap)**
To validate that this traffic is indeed "Normal," I performed a mental audit based on standard SOC triage procedures:

*   **Is the source IP expected?** Yes, `192.168.10.139` is a known internal workstation within the lab environment.
*   **Are the domains reputable?** Yes, the destinations include `Mozilla`, `Google`, and `Youtube`, which are high-reputation, expected domains for a user.
*   **Is there any 'Entropy'?** No. The domains are readable English. There are no signs of DGA (Domain Generation Algorithms) or "gibberish" strings like `asdf88asdf.com`.
*   **Is the timing human-like?** Yes, the irregular spacing between requests confirms manual browsing rather than a rhythmic, automated bot.

> **Baseline Status: COMPLETE.** 🟢
> We have successfully defined what "Normal" looks like. Any deviation from these patterns in the following steps will be treated as a potential security anomaly.

---

#### **🕵️ SOC Analyst Technical Insight**
Looking at the packet details for the selected frame in **figure 1**, the **Total Length is 81 bytes**. Standard, non-malicious DNS queries are typically small. If this were **DNS Tunneling** for C2 communication, we would expect to see significantly larger packet lengths (e.g., 200+ bytes) as the attacker attempts to "stuff" encoded data into the DNS query fields.


### **Step 2: Automated Behaviour (The Pulse)**
* **Scenario:** Rhythmic `ping` command to a single domain.
* **Observation:** The "Time" column in Wireshark shows queries at perfect 5-second intervals. This is non-human "heartbeat" traffic.
> **<img width="1701" height="960" alt="ping" src="https://github.com/user-attachments/assets/7e265af5-89e1-4f58-8ce2-dfe68fd505b6" />
**

---

### **Phase 2.5: Endpoint Verification & The "Visibility Gap"**

#### **The Observation**
After identifying a rhythmic DNS "heartbeat" in Wireshark, I pivoted to the **Wazuh Dashboard** to correlate the network activity with endpoint telemetry. As shown in **waz test.png**, the agent `kali-sowrd` is active and healthy, yet the dashboard shows no security alerts triggered by the automated loop.

#### **Technical Analysis: Why "Quiet" is a Risk**
*   **The Baseline Trap:** This activity is currently classified as "Normal/Quiet" by the system's default ruleset.
*   **Attacker Behavior:** Elite-level threat actors avoid "loud" exploits. They use **Living-off-the-Land (LotL)** techniques—using built-in tools like `ping` or `nslookup`—to blend in with standard administrative traffic.
*   **The Assumption Risk:** If an analyst assumes that "No Alert = No Threat," they may miss a low-and-slow Command & Control (C2) beacon. 

#### **Evidence**
<img width="1853" height="1034" alt="waz test" src="https://github.com/user-attachments/assets/cf646ad9-b0e1-4974-ad69-e6f92d33b64d" />

*Figure 3: Wazuh dashboard showing an active agent but a flat event line for the background heartbeat.*

---

#### **🕵️ SOC Analyst Technical Insight**
In **waz test.png**, the **Events count evolution** is flat despite the active loop. This confirms a **Visibility Gap**. 

As a SOC Analyst, I must verify anything that looks "doubtful" or "periodic." In a real-world scenario, attackers hide their communications inside these mundane loops. By refusing to assume safety, I protect the organization from the massive financial and reputational costs of a long-term breach. 

> **Expert Mindset:** We do not assume safety; we prove it. If a network pattern is periodic but the endpoint is silent, we must investigate the process ID (PID) to confirm legitimacy.

#### **The Expert "Think" Points (Recap)**
*   **Verification over Assumption:** Does a lack of alerts mean the system is safe, or is the "security camera" just pointing the wrong way?
*   **Blending In:** Attackers don't want to stand out; they want to look like a 5-second `ping` loop.
*   **Financial Impact:** Proactive verification saves millions by catching "silent" threats before they escalate

> [!IMPORTANT]
> **Why investigate the PID?**
> In a real-world investigation, the Process ID (PID) is the "DNA" of the activity. While Wireshark tells us *traffic* is flowing, the PID tells us *which specific program* is responsible. 
> 
> By tracing the PID back to its **Parent Process**, I can determine if the activity was started by a human (e.g., via `terminal`) or by a hidden malicious script (e.g., a macro inside a fake PDF).

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
