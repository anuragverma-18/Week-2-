# Network Reconnaissance Reports

This folder holds two reports from a security-learning session on **12 November 2026**. Both document the tools used, the screenshots captured and what the output means.

> **Authorized use only.** Run these tools only against networks and domains you own or have written permission to test.

## Contents

| File | What it covers | Target |
|---|---|---|
| `Network_Scan_Report.docx` | Local network scan with Nmap/Zenmap on Windows | Private subnet `10.107.172.0/24` |
| `networkwalks_recon_report.docx` | Passive and light active recon on a public website, run from Kali Linux | `networkwalks.com` |

---

## 1. Network_Scan_Report.docx

**Goal:** Discover the hosts on the local Wi-Fi network and see which ports they expose.

**Environment:** Windows PC (Realtek RTL8822CE Wi-Fi adapter), Zenmap/Nmap 7.993.

**Command:** `nmap -T4 -F 10.107.172.0/24` (Quick scan profile)

**Sections**
1. Summary
2. Local network configuration (`ipconfig` output)
3. Scan results (two scans, 19:34 and 19:40)
4. Topology (Zenmap topology views)
5. Observations and recommendations

**Key findings**
- 4 of 256 addresses were up.
- `10.107.172.91` (the scanning PC) has ports **135, 139 and 445** open (msrpc, netbios-ssn, microsoft-ds), and NetBIOS over TCP/IP is enabled.
- `10.107.172.4` is the gateway, DHCP server and DNS server, with only port 53 open.
- `10.107.172.194` and `10.107.172.238` have all scanned ports closed and use randomized MAC addresses, so they are probably phones or tablets.
- Port 7 (echo) on `.194` was filtered in the first scan but closed in the second, so it was likely transient.

**Recommendations:** Set the network profile to *Public*, and disable NetBIOS and file sharing if they are not needed.

---

## 2. networkwalks_recon_report.docx

**Goal:** Gather public information about a website's registration, DNS, technology stack and protections.

**Environment:** Kali Linux. Scope was passive and light active information gathering.

**Tools and results**

| # | Tool | Purpose | Key result |
|---|---|---|---|
| 1 | `whois` | Domain registration | Registered via GoDaddy, name servers at HostGator, expires 2027-11-06 |
| 2 | `whatweb` | Technology fingerprinting | WordPress on Apache, plus the Download Manager plugin, IP 192.232.216.135 |
| 3 | `nslookup` | DNS resolution | A record 192.232.216.135; the AAAA is a NAT64-synthesized address |
| 4 | `curl -I` | HTTP headers | HTTP/2 200; WordPress REST API advertised in headers |
| 5 | `wafw00f` | WAF detection | Behind ModSecurity (SpiderLabs) |
| 6 | `dnsrecon -d` | DNS enumeration | SOA, NS, MX, A, AAAA and SRV records found |

**Overall picture:** The domain is at GoDaddy, with DNS and hosting at HostGator (cPanel, shared). The stack is Apache with nginx caching running WordPress. HTTP redirects to HTTPS, and a WAF is in place.

**Points to review**
- DNSSEC is not enabled.
- The name servers disclose their BIND version.
- The WordPress and plugin versions are exposed, so check the plugin against known CVEs.
- The REST API is advertised, so verify that user enumeration is restricted.
- HSTS, CSP, X-Frame-Options and X-Content-Type-Options are not visible in the headers.
- `info@networkwalks.com` is publicly exposed.

**Suggested next steps (with authorization):** subdomain and certificate-transparency checks, `wpscan`, a security-header review, and SPF/DKIM/DMARC checks.

---

## Opening the reports

Both files are Word documents with embedded screenshots. To get PDFs, open either one in Word or WPS and use **File > Export as PDF**, or ask for a converted copy.

## Tools referenced

`nmap` / Zenmap · `ipconfig` · `whois` · `whatweb` · `nslookup` · `curl` · `wafw00f` · `dnsrecon` · `wpscan` (suggested)
