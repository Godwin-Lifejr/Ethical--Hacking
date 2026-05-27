# 🛡️ The Master Guide to Nmap: Concept & Execution

Nmap is more than a port scanner; it is a packet-crafting engine. To be an ethical hacker, you must understand the "handshake" mechanics behind each command.

---

## 1. Network Discovery (Layer 2 & 3)
Before scanning ports, you must find which hosts are "alive."

### Concept: The Ping Sweep
In a local network (like your lab), Nmap uses **ARP requests (Layer 2)** because they are faster and cannot be blocked by host-based firewalls. On external networks, it uses **ICMP Echo Requests (Layer 3)**.

### The Command:
```bash
sudo nmap -sn 192.168.1.0/24
```
*   **-sn (No Port Scan):** This tells Nmap to stop after host discovery. It identifies active IPs without touching the ports.
*   **Best For:** Mapping an office or a cloud subnet quickly.

---

## 2. Scanning Techniques (The Handshake)
This is the core of active reconnaissance.

### Concept A: TCP SYN Scan (The "Stealth" Scan)
This is the default and most popular scan. It follows the Three-Way Handshake but intentionally breaks it.
1. You send a **SYN** (Synchronize).
2. Target sends back **SYN/ACK** (Port is Open).
3. You send a **RST** (Reset) instead of an ACK.

**Why?** Since the connection is never fully established, many application-level logs won't record the connection.

### The Command:
```bash
sudo nmap -sS <Target_IP>
```

### Concept B: TCP Connect Scan
This completes the full 3-way handshake.

**Why?** Use this if you don't have `sudo` privileges on Kali, as SYN scans require raw socket access.
**Downside:** It is very noisy and easily logged by firewalls.

### The Command:
```bash
nmap -sT <Target_IP>
```

---

## 3. Bypassing Defenses (The Windows 10 Challenge)
Windows 10 machines are "hardened." By default, they ignore ICMP pings, making Nmap think they are offline.

### Concept: The No-Ping Bypass
We force Nmap to ignore the lack of a "ping" response and scan the ports anyway.

### The Command:
```bash
sudo nmap -Pn -F <Windows10_IP>
```
*   **-Pn (No Ping):** Tells Nmap "Treat the host as if it is alive."
*   **-F (Fast):** Scans only the top 100 most common ports (e.g., 80, 445, 3389).

---

## 4. Service & OS Enumeration
Knowing a port is "open" isn't enough. You need to know the version of the software.

### Concept: Banner Grabbing
Nmap sends specific probes to an open port and waits for the service to respond with its "Banner" (e.g., Apache 2.4.41).

### The Command:
```bash
sudo nmap -sV -O <Target_IP>
```
*   **-sV (Version Detection):** Analyzes the response to determine the version of the service.
*   **-O (OS Detection):** Nmap examines the "TCP/IP stack fingerprinting." Different Operating Systems handle packet fragmentation and TTL (Time to Live) differently.

---

## 5. Web Infrastructure Recon (External Websites)
When scanning a website, you aren't just scanning a server; you are scanning a Web Stack.

### Concept: Directory & Header Discovery
We use Nmap to find hidden files and see how the web server is configured.

### The Command:
```bash
sudo nmap -p 80,443 --script http-enum,http-headers <Website_URL>
```
*   **http-enum:** This script searches for common directories like `/admin`, `/login`, or `/secret`.
*   **http-headers:** Reveals the server type (e.g., Cloudflare, Nginx) and security headers.

---

## 6. NSE: The Nmap Scripting Engine
NSE turns Nmap into a vulnerability scanner using scripts written in Lua.

### The Command:
```bash
sudo nmap --script vuln <Target_IP>
```
*   **--script vuln:** Checks the target against a database of known CVEs.
*   **Example:** Detects the `vsftpd` backdoor on Port 21 or `ms17-010` (EternalBlue) on Windows.

---

## 🛠️ Summary Table: Flags Cheat Sheet


| Flag | Name | Meaning |
| :--- | :--- | :--- |
| **-sS** | SYN Scan | Stealthy, half-open scan. Default. |
| **-sU** | UDP Scan | Scans for DNS, DHCP, and SNMP. |
| **-p-** | All Ports | Scans all 65,535 ports. |
| **-T4** | Timing | Speeds up the scan (T0-T5). |
| **-oA** | Output All | Saves results in Normal, XML, and Grepable formats. |
