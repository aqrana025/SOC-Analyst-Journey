# Lab: DNS Behaviour & Analysis 🔍

## 1. Objective
Analyze DNS traffic to identify potential anomalies such as DNS Tunneling or Command & Control (C2) communication.

## 2. Tools Used
* **Wireshark** (Traffic Analysis)
* **Linux Terminal** (dig/nslookup commands)
* **Splunk** (Log Visualization)

## 3. Analysis Steps
1. **Traffic Capture:** Captured DNS queries from a test machine.
2. **Observation:** Noticed high frequency of TXT records to an unknown domain.
3. **Filtering:** Used Wireshark filter `dns.flags.response == 0` to isolate queries.

## 4. Evidence (Screenshots)
> [!IMPORTANT]
> will me update screenshots here!
> `![DNS Analysis](image_link_here)`

## 5. Conclusion
The activity was identified as [Normal/Malicious]. Custom alert rules were created to monitor for similar spikes in the future.
