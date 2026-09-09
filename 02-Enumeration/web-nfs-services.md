# Phase 2: Service Enumeration & Web Fuzzing

## 2.1 Web Fingerprinting (Port 80 & 8080)
Initial technology stack identification was conducted using `whatweb`.

```bash
whatweb http://192.168.120.145
```
The investigation revealed a **Bolt CMS** web application framework. Inspecting the `robots.txt` file provided early layout indicators.

### Directory Brute-Forcing
**Gobuster** was launched against both port 80 and port 8080 to map internal directories using a standard directory wordlist.

```bash
# Directory discovery on main application
gobuster dir -u http://192.168.120.145 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -re -t 200

# Directory discovery on secondary web application (Port 8080)
gobuster dir -u http://192.168.120.145:8080 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

#### HTTP Response Status Codes Observed:
* `200 OK`: Resource found and fetched successfully.
* `301 Moved Permanently`: The resource has a new permanent URL.
* `403 Forbidden`: Server refuses to authorize request.
* `404 Not Found`: Resource does not exist.

**Critical Findings:** 
* `http://192.168.120.145/app/config/config.yml` (Configuration leakage potential).
* `http://192.168.120.145:8080/dev/` (Development endpoint exposed running **BoltWire**).

## 2.2 NFS Share Enumeration (Port 2049)
The Nmap scan showed port 2049 (`nfs_acl`) active. The remote RPC target was queried to extract public mount points.

```bash
# Query active mount lists
showmount -e 192.168.120.145
```

### Output:
```text
Export list for 192.168.120.145:
/srv/nfs 172.16.0.0/12,10.0.0.0/8,192.168.0.0/16
```
**Analysis:** The host machine is exporting `/srv/nfs` to wide internal network subnets, including the attacker's scope (`192.168.0.0/16`).

---

## 🛡️ Remediation & Defensive Hardening
1. **Restrict NFS Exports:** Modify `/etc/exports` to allow only specific, authorized IP addresses instead of entire Class B/C networks.
2. **Information Disclosure Prevention:** Disable public access to sensitive web paths like `/app/config/` and generic development `/dev/` roots via `.htaccess` or server block configurations.
