# PENETRATION TESTING REPORT

*Footprinting & Network Scanning Phases*

*W2-PM-FINAL  |  Cybersecurity  |  Networkwalks*

| Pentester Name | [Your Name] (Cybersecurity Professional) |
| --- | --- |
| Program / Batch | B082 - Networkwalks |
| Date | 11 November 2026 |
| Modules completed | W2-PM1 (Multiple Kali Tools)<br>W2-PM5 (Zenmap Scanning) |
| Client / Target | 1. networkwalks.com (written permission secured)<br>2. My own local LAN (192.168.0.0/24) |
| Permission secured? | Yes |
| Phases covered | Phase 1: Reconnaissance & Footprinting<br>Phase 2: Scanning & Network Discovery<br>Phases 3-5: In progress |

## 1. Liability Disclaimer

I have performed these activities only on systems and devices for which I had secured written permission or which I own myself. All material in this report is for education and research purposes only. Do not use anything from it to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

## 2. Introduction

This report covers footprinting of the networkwalks.com domain using multiple Kali Linux tools (W2-PM1) and scanning of my local network with Zenmap (W2-PM5). One module covers the footprinting phase and the other covers the scanning phase, so together they show how an attacker moves from gathering public information to mapping live hosts on a network. It is the Week 2 part of my ongoing internship program at Networkwalks.

Footprinting was performed from Kali Linux 2026.3 running in Oracle VirtualBox. Scanning was performed with Zenmap on a Windows PC. Every step below includes the exact command used, the result observed, a screenshot as evidence, and a short note on why the finding matters from an attacker’s point of view.

## 3. Tools Used

| Tool | Purpose |
| --- | --- |
| Kali Linux & Windows | Operating systems used for reconnaissance activities |
| whois | Find domain registration details (registrar, dates, status). |
| whatweb | Fingerprint web technologies (server, CMS, plugins, IP). |
| nslookup | Resolve the domain name to its IP address using DNS. |
| curl -I | Read the HTTP response headers of the website. |
| wafw00f | Detect whether a Web Application Firewall protects the site. |
| dnsrecon | Enumerate DNS records (SOA, NS, MX, A, TXT/SPF, SRV). |
| Zenmap (Nmap GUI) | Scan the local subnet to find live hosts, open ports, IPs and MAC addresses; draw the topology. |

## 4. Activities Performed

### 4.1 Footprinting & Reconnaissance

Reconnaissance was performed against networkwalks.com using six Kali Linux tools. Each tool was used to collect a different type of information about the target. All of it is passive or low-impact information gathering; no exploitation was attempted.

#### 4.1.1 WHOIS – Domain Registration

**Command:** whois networkwalks.com

**Result:** The domain is registered through GoDaddy.com, LLC (IANA ID 146). It was created on 6 November 2019, last updated on 12 November 2025 and expires on 6 November 2027. The status flags clientDeleteProhibited, clientRenewProhibited and clientTransferProhibited are set. The registrant’s personal details are not shown, only the registrar’s abuse contact.

**Why it matters:** Registration and expiry dates show how long the domain has existed and when it must be renewed. The registrar lock flags are a good security practice because they make unauthorised transfer or deletion harder. Domain ownership details help an attacker plan social-engineering or domain-hijacking attempts, so keeping registrant data private is usefu


https://github.com/kavyag41168-bot/footprinting-network-scanning/blob/c8deb4631aeee44c9ad37e437764b0c4ebe635b2/whois%20%20tool%20screen%20short.jpeg

*Figure 1: WHOIS output for networkwalks.com*

#### 4.1.2 WhatWeb – Web Technology Fingerprinting

**Command:** whatweb networkwalks.com

**Result:** HTTP on port 80 returns 301 Moved Permanently and redirects to HTTPS. The HTTPS site (200 OK) runs Apache, WordPress 7.1.4, the WordPress Download Manager plugin 3.3.76, jQuery 3.7.1 and Google Tag Manager. The IP address 192.232.216.135 is shown, the contact email info@networkwalks.com is exposed, and the site sets a cookie named __wpdm_client with the HttpOnly flag.

**Why it matters:** Knowing the CMS, plugin and version lets an attacker search for known vulnerabilities in exactly those components. The finding is an exposure of information, not a confirmed vulnerability. The HTTP-to-HTTPS redirect and the HttpOnly cookie are positive controls.

![Evidence figure 2](assets/figure-2.png)

*Figure 2: WhatWeb fingerprint of networkwalks.com*

#### 4.1.3 Nslookup – DNS Resolution

**Command:** nslookup networkwalks.com

**Result:** The query was answered by my local DNS resolver 192.168.0.1 (port 53) as a non-authoritative answer. The domain resolves to 192.232.216.135.

**Why it matters:** This gives the public IP address of the web service, which becomes the target for later port scanning. It also confirms that the lab machine sits on the 192.168.0.0/24 network used in Section 4.2.

![Evidence figure 3](assets/figure-3.png)

*Figure 3: nslookup resolving networkwalks.com*

#### 4.1.4 Curl – HTTP Response Headers

**Command:** curl -I https://networkwalks.com

**Result:** The server answered HTTP/2 200 and identified itself as Apache. The headers also reveal x-nginx-cache: WordPress and x-endurance-cache-level: 0 (a caching layer in front of the site), a Link header that advertises the WordPress REST API at /wp-json/ together with a REST route for page 53, a Permissions-Policy header, Referrer-Policy: no-referrer-when-downgrade, and a secure, HttpOnly session cookie.

**Why it matters:** The Link header confirms WordPress and shows the REST API endpoint, which an attacker can use to enumerate content. The caching headers disclose parts of the hosting stack. In the captured response I did not see HSTS, X-Frame-Options, X-Content-Type-Options or Content-Security-Policy headers; these are worth reviewing. The secure and HttpOnly cookie flags are good practice.

![Evidence figure 4](assets/figure-4.png)

*Figure 4: HTTP response headers from curl -I*

#### 4.1.5 Wafw00f – WAF Detection

**Command:** wafw00f networkwalks.com

**Result:** Wafw00f v2.4.2 reported that https://networkwalks.com is behind the ModSecurity (SpiderLabs) Web Application Firewall. Detection took 2 requests.

**Why it matters:** A WAF is a protective control, so its presence is positive. Knowing which WAF is deployed still tells an attacker what filtering to expect and how to tune later testing, so this is low-risk information disclosure.

![Evidence figure 5](assets/figure-5.png)

*Figure 5: wafw00f detecting ModSecurity*

#### 4.1.6 DNSRecon – DNS Enumeration

**Command:** dnsrecon -d networkwalks.com

**Result:** SOA and NS records point to ns6135.hostgator.com (50.87.144.87) and ns6136.hostgator.com (192.232.216.131). Both name servers disclose their BIND version as 9.16.23-RH. The MX record is mail.networkwalks.com (192.232.216.135) and the A record is 192.232.216.135. TXT records include a Google site-verification token and an SPF policy (v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all). SRV records for _autodiscover._tcp point to cpanelemaildiscovery.cpanel.net on port 443. The tool also reported that no DNSSEC answer was returned for the domain.

**Why it matters:** These records map the hosting provider (HostGator), the mail setup and the cPanel environment. Disclosed BIND versions help an attacker look for matching DNS software issues. The SPF record ends in ~all (soft fail), which is weaker than -all against email spoofing. No DNSSEC means DNS answers are not cryptographically signed. The screenshot shows the first part of the output; the SRV list continues beyond the visible window.

![Evidence figure 6](assets/figure-6.png)

*Figure 6: DNSRecon enumeration of networkwalks.com*

### 4.2 Network Scanning with Zenmap

For the second activity I used Zenmap (the Nmap graphical interface) to discover devices on my local network. The subnet 192.168.0.0/24 was taken from the lab machine’s network settings (its DNS resolver is 192.168.0.1, see Figure 3).

**Command:** nmap -T4 -F 192.168.0.0/24   (Profile: Quick scan)

**Result:** Nmap scanned 256 IP addresses and found 91 hosts up in 23.81 seconds. The quick scan checks the 100 most common TCP ports on each host. Most hosts are phones and personal devices whose ports are closed or filtered. A few hosts show open services, summarised below.

| IP address | Host name | Open ports / services | MAC vendor |
| --- | --- | --- | --- |
| 192.168.0.254 | (no host name) | 22 ssh, 80 http, 443 https | Grandstream Networks |
| 192.168.0.129 | kavyaHacker | 135 msrpc, 139 netbios-ssn, 445 microsoft-ds, 3306 mysql | (not shown) |
| 192.168.0.253 | CMF-by-Nothing-Phone-2-Pro | 7/tcp echo filtered; 99 closed | Unknown (randomised) |
| 192.168.0.235 | OPPO-A5-Pro-5G | None (100 closed) | Unknown (randomised) |
| 192.168.0.248 | moto-g67-power-5G | None (100 filtered) | Motorola Mobility |
| 192.168.0.251 | Tejaswini | None (100 filtered) | Cloud Network Technology |

![Evidence figure 7](assets/figure-7.png)

*Figure 7: Zenmap Nmap Output tab (nmap -T4 -F 192.168.0.0/24)*

After the scan I opened the Topology tab, which draws all 91 discovered hosts around the scanning machine. The legend can be switched on and the diagram saved with the Save Graphic button. Because there are so many hosts, the labels overlap in the default view; the Hosts Viewer, Fisheye and Controls options can be used to zoom in and read individual hosts.

![Evidence figure 8](assets/figure-8.png)

*Figure 8: Zenmap Topology tab for 192.168.0.0/24*

## 5. Risk Analysis / Impact

Based on the information collected, I identified the following potential risks.

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
| --- | --- | --- | --- | --- |
| 1 | CMS and plugin versions exposed | WhatWeb: WordPress 7.1.4, WP Download Manager 3.3.76 | Attackers can match exposed versions against known vulnerabilities for further review | ● Medium |
| 2 | Server IP address identifiable | nslookup / whatweb: 192.232.216.135 | Shows the network location of the web and mail service | ● Low |
| 3 | HTTP technical information exposed | curl: /wp-json/ link, caching headers; no HSTS/CSP seen | Helps fingerprinting and enumeration; missing hardening headers | ● Low |
| 4 | WAF technology identifiable | wafw00f: ModSecurity (SpiderLabs) | Reveals part of the security architecture | ● Low |
| 5 | DNS infrastructure details exposed | dnsrecon: BIND 9.16.23-RH, SPF ~all, no DNSSEC answer | Builds a hosting profile; softer email-spoofing protection | ● Medium |
| 6 | Many live hosts on the local network | Zenmap: 91 hosts up on 192.168.0.0/24 | Unknown or unauthorised devices could be present | ● Medium |
| 7 | Database and file-sharing ports open on a LAN host | 192.168.0.129: 445 SMB, 139 NetBIOS, 135 MSRPC, 3306 MySQL | Exposed services increase attack surface for other devices on the same network | ● Medium |
| 8 | Management services on network device | 192.168.0.254: 22 SSH, 80 HTTP, 443 HTTPS | Admin interfaces should be restricted and use strong credentials | ● Low |

Risk level key:  🔴 Critical  🟠 Medium  🟢 Low

The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities. The practical exercises involved information gathering and host discovery only. No exploitation or vulnerability validation was performed. The presence of a software version, IP address, DNS record or open port does not by itself mean that a system is vulnerable. Further authorised security testing would be required to confirm any actual vulnerability.

## 6. Recommendations

- **Review publicly exposed technology information:** Regularly review what information about web technologies, CMS and plugins is publicly visible.

- **Keep software updated:** Update WordPress core and plugins (including WP Download Manager) and review them against current security advisories.

- **Harden HTTP headers:** Remove unnecessary technical headers, and consider adding HSTS, X-Content-Type-Options, X-Frame-Options and a Content-Security-Policy.

- **Review DNS records and mail policy:** Hide DNS software versions where possible, consider DNSSEC, and consider tightening SPF from ~all to -all together with DKIM and DMARC.

- **Properly configure and monitor the WAF:** Keep ModSecurity enabled and tuned, since it already blocks naive attacks.

- **Perform regular internal network discovery:** Periodically scan your own network to identify active devices.

- **Investigate unknown devices:** Verify any unexpected device found during scanning, and review devices using randomised MAC addresses.

- **Close or restrict unneeded services:** Disable SMB, NetBIOS and MySQL access from the LAN where not needed, bind MySQL to localhost, and restrict router/AP administration to trusted hosts.

- **Maintain network documentation:** Keep the network topology and device inventory documented and up to date.

- **Test only with authorisation:** Reconnaissance and scanning must only be performed against systems and networks where authorisation has been given.

## 7. Conclusion

During Week 2 of my Cybersecurity & Ethical Hacking internship I completed practical activities covering footprinting, reconnaissance and network scanning.

In the footprinting activity I used six Kali Linux tools to collect information about the target domain. WHOIS provided registration details, WhatWeb identified the web technologies, Nslookup resolved the domain to its IP address, Curl exposed the HTTP headers, Wafw00f identified the WAF and DNSRecon revealed the DNS and mail infrastructure.

In the scanning activity I used Zenmap to discover 91 live hosts on 192.168.0.0/24, review open ports and MAC address information, and view the network topology.

The exercises showed that information gathering is an important part of cybersecurity. Even before attempting to exploit a system, a security professional can learn a great deal from public information and network responses. I also learned that findings must be documented clearly: what was done, what was found, what it means, what risk it creates and how to reduce it. Finally, reconnaissance and scanning must always stay within an authorised scope. These activities were completed as part of the assigned educational cybersecurity lab.

## 8. Evidences Collected

Figures 1 to 8 in Section 4 are the evidence for this report: WHOIS, WhatWeb, nslookup, curl, wafw00f and DNSRecon output from Kali Linux, and the Zenmap Nmap Output and Topology views.

---

**- End -**

### Author

Waqas Karim CCIE

Cybersecurity Professional B082

LinkedIn: https://www.linkedin.com/in/waqaskarim/

### Project Information

Program Name: Cybersecurity program at Networkwalks  |  Week: 02  |  Repository: GitHub


