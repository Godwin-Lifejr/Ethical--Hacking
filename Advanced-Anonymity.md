# 🕵️ Advanced Anonymity & Anti-Forensics Infrastructure
## A Multi-Layered Defense Lab for Ethical Hacking

This repository documents the architectural setup of a hardened digital identity. We move from standard network obfuscation to an amnesic hardware environment, culminating in a **Whonix-inside-Tails "HiddenVM"** deployment.

---

## 🛠️ Level 1: The Network Layer (Tor Core & Obfuscation)
The foundation of the lab is the Tor (The Onion Router) network. We do not use standard entry nodes; we use **Bridges** to ensure the ISP cannot fingerprint our traffic as Tor.

### 1. Source Acquisition & Installation
* **Tor Browser Bundle:** [Download via TorProject](https://torproject.org)
* **System-Wide Tor Daemon:**

```bash
# Adding the official Tor repository for the latest version
sudo apt update && sudo apt install lsb-release realpath -y
sudo apt install tor tor-geoipdb obfs4proxy -y
```

### 2. Implementing Pluggable Transports (obfs4)
Standard Tor is easily blocked. We use `obfs4` to make traffic look like unidentifiable "white noise."

* **Request Bridges:** Visit [bridges.torproject.org](https://torproject.org) and select obfs4.
* **Hardened Configuration (`/etc/tor/torrc`):**

```bash
sudo nano /etc/tor/torrc
```

Add these configurations:
```text
# Disable standard entry guards
UseBridges 1

# Define the transport plugin path
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy

# Insert your unique bridge lines here
Bridge obfs4 [IP_Address]:[Port] [Fingerprint] cert=[CertID] iat-mode=0
```

* **Bootstrap Check:**
```bash
sudo systemctl restart tor
# Monitor the log for successful circuit creation
sudo journalctl -e -u tor | grep "Bootstrapped 100%"
```

---

## 🐚 Level 2: Command Line Tunneling (Proxychains-NG)
Most hacking tools (Nmap, Metasploit, etc.) leak your real IP. Proxychains forces them through the Tor circuit we built in Level 1.

### 1. Installation
```bash
sudo apt install proxychains4 -y
```

### 2. Configuration Hardening
Edit the config file: `sudo nano /etc/proxychains4.conf`

* **Dynamic Chain:** Enable this by removing the `#`. This allows the tool to skip "dead" proxies in a chain without failing.
* **Proxy DNS:** Ensure `proxy_dns` is active to prevent DNS Leaks.
* **Targeting the Local Daemon:** At the end of the file, set:
```text
socks5 127.0.0.1 9050
```

---

## 🛡️ Level 3: Identity Defense (Panopticlick & ProtonVPN)
We layer a VPN to hide our Tor entry point and use fingerprint analysis to blend in.

### 1. ProtonVPN Implementation (The Entry Guard)
* **Download:** [ProtonVPN Linux Client](https://protonvpn.com)
* **Setup:**
```bash
# Install the Proton repository package
sudo dpkg -i protonvpn-stable-release_1.0.3-3_all.deb
sudo apt update && sudo apt install protonvpn-cli -y

# Connect using "Secure Core" for multi-hop entry
protonvpn-cli login [Account_Name]
protonvpn-cli connect --sc
```

### 2. Fingerprint Mitigation
Using EFF’s **Cover Your Tracks**:
* **Action:** Analyze your "Browser Uniqueness."
* **Hardening:** Disable **WebRTC** and **Canvas rendering** in `about:config` to ensure your browser looks identical to thousands of others.

---

## ❄️ Level 4: The Amnesic Host (Tails OS)
Tails is our primary defense against local forensics. It leaves no trace on the SSD/HDD.

### 1. Flash and Verify
* **ISO Source:** [Tails.net](https://tails.net)
* **Integrity Check:**
```bash
# Verify the ISO signature
gpg --verify tails-amd64-x.x.iso.sig
```
* **Deployment:** Use `dd` for a bit-for-bit raw copy to USB:
```bash
sudo dd if=tails-amd64.img of=/dev/sdX bs=16M status=progress
```

### 2. Persistent Storage Configuration
Boot Tails and open `Applications > Tails > Configure Persistent Storage`.
* **Crucial Modules:** Enable GnuPG, Network Connections, and Bitcoin Client.

---

## 🏰 Level 5: The "HiddenVM" (Gubs & Whonix)
The ultimate configuration: **Running Whonix inside Tails.**

### 1. The Whonix Architecture
* **Whonix-Gateway:** Acts as a dedicated Tor router.
* **Whonix-Workstation:** A sanitized OS for hacking; no access to hardware or real IP.

### 2. Nested Setup on Tails
1. Boot Tails with an **Admin Password**.
2. Mount **Persistence** to store VM images.
3. **Install VirtualBox:**
```bash
sudo apt update
sudo apt install virtualbox -y
```
4. **Import Whonix:** Download `.ova` from [Whonix.org](https://whonix.org) and import.

**The Logic:** `Tails (Hardware) -> VPN -> Tor Bridge -> Whonix Gateway -> Tor Exit -> Target`.

### 3. Final Lab Verification
In the Whonix Workstation, run:
```bash
whonixcheck
```

---

> [!CAUTION]
> **Disclaimer:** This repository is for educational purposes only. Unauthorized access to systems is illegal.
