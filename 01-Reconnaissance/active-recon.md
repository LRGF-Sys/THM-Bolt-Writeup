# Phase 1: Active Reconnaissance & Host Discovery

## 1.1 Local Network Discovery
To identify the target IP address within the laboratory environment, layer 2 and layer 3 discovery tools were utilized.

* **ARP Scan (Layer 2 Discovery):**
```bash
arp-scan -l
```
* **Netdiscover (Layer 3 Passive/Active Broadcast):**
```bash
netdiscover -r 192.168.120.0/24
```

## 1.2 Port Scanning & OS Fingerprinting
A multi-stage scanning strategy was deployed using **Nmap** to ensure speed, accuracy, and thorough coverage.

### Full Port Discovery (TCP)
A fast, aggressive Syn-Scan (`-sS`) was executed across all 65,535 ports, forcing a minimum rate of 3,000 packets per second to bypass slow timers.

```bash
nmap -sS -p- -vv 192.168.0.107 --min-rate 3000 -oA allports
```

**Efficiency Trick:** The output was filtered using regular expressions (`grep` + `awk`) to extract only open ports into a clean, comma-separated list for subsequent targeted scans:
```bash
cat allports.nmap | grep -oP "[0-9]{1,5}/tcp" | awk '{print $1}' FS=/ | xargs | tr ' ' ','
```

### Targeted Service & Aggressive Enumeration
Once ports **21, 22, 80, 2049, and 8080** were identified, a focused scan was performed to extract service versions (`-sV`) and run default safe scripts (`-sC`).

```bash
nmap -sV -sC -p 21,22,80,2049,8080 192.168.0.107 -vv -oA servicios
```

### Operating System Fingerprinting (Passive)
A single ICMP echo request (ping) was sent to evaluate the target's operating system based on its network stack defaults.

```bash
ping -c 1 192.168.0.107 | grep -oP 'ttl=\K\d{1,3}'
```
* **Result:** `ttl=64`
* **Analysis:** A Time-to-Live (TTL) value of 64 strongly indicates that the target host is running a **Linux** operating system kernel.

---

## 🛡️ Remediation & Defensive Hardening
1. **Disable ICMP/Ping Responses:** Configure the host firewall (`iptables` / `nftables`) to drop echo-requests to prevent passive OS fingerprinting.
2. **Implement Rate Limiting:** Deploy tools like `Portsentry` or configure firewall rules to detect and drop high-rate port scans (e.g., `--min-rate 3000`).
