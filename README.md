PENETRATION TESTING REPORT
FOOTPRINTING & NETWORK SCANNING PHASES
W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS





Pentester Name (Cybersecurity Professional)
Kwanele Dube
Program/Batch
B083 — Networkwalks Cybersecurity Internship
Date
17 September 2026
Modules Completed
W2-PM1 (Multiple Kali Tools) · W2-PM2 (GHDB) · W2-PM3 (Maltego — documented below) · W2-PM4 (theHarvester) · W2-PM5 (Zenmap)
Client/Target
1. Networkwalks — networkwalks.com (internship-assigned target) · 2. microsoft.com (public OSINT exercise) · 3. Own local LAN network
Permission secured from client?
Yes — proceeding under Networkwalks internship assignment as authorization. Note: Batch B083 LOA not received; only a B082 sample LOA was available. All activities remained within the passive/non-destructive scope defined by the internship program.
Phases covered
W2-PM1 ✅ · W2-PM2 ✅ · W2-PM3 (documented tooling failure) · W2-PM4 ✅ · W2-PM5 ✅ 


1. Liability Disclaimer
I have performed these activities only on the systems and devices where I had secured written permission or the devices/systems that I own myself. All these materials are for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

2. Introduction
This report covers Week 2 of my Networkwalks Cybersecurity Internship (Batch B083). The week focused on footprinting and reconnaissance using multiple tools across five project modules:
W2-PM1: Passive footprinting of networkwalks.com using six Kali Linux tools
W2-PM2: Google Hacking Database (GHDB) dorking against networkwalks.com
W2-PM3: Maltego — attempted but not completed due to a tooling issue (documented below)
W2-PM4: theHarvester OSINT email and subdomain harvesting against microsoft.com
W2-PM5: Zenmap (Nmap GUI) network scan of my local LAN on Windows
Together these modules demonstrate how a security professional moves from gathering publicly available information about a target (Phase 1 — Reconnaissance) to mapping live hosts on a network (Phase 2 — Scanning), before any exploitation is attempted.
All commands were run in Kali Linux 2026.2 (VirtualBox VM on a Lenovo ThinkPad T430) for footprinting activities, and on a Windows 10 PC with Zenmap installed for network scanning. Every step below includes the exact command used, the result observed, and a note on why the finding matters from a security perspective.

3. Tools Used
Tool
Purpose
Kali Linux (VirtualBox)
Primary operating system for all reconnaissance activities
WHOIS
Find domain registration details: owner, registrar, dates, name servers
WhatWeb
Fingerprint web technologies: server, CMS, plugins, frameworks, IP
nslookup
Resolve the domain name to its IP address using DNS
curl -I
Read raw HTTP response headers directly from the web server
wafw00f
Detect whether a Web Application Firewall is protecting the site
dnsrecon
Enumerate all DNS records: NS, MX, A, TXT, SPF, SRV
Google (GHDB dorks)
Search for publicly indexed sensitive files or misconfigurations
Maltego CE v4.12.1
OSINT link-analysis and entity mapping (attempted — see PM3 note)
theHarvester
Harvest emails, subdomains and IPs from public OSINT sources
Zenmap (Nmap 7.991 GUI)
Scan local subnet to discover live hosts, IPs and MAC addresses
Windows PowerShell / CMD
Local IP address and MAC address identification


4. Activities Performed
4.1 W2-PM1 — Footprinting & Reconnaissance (networkwalks.com)
I performed passive reconnaissance against the networkwalks.com domain using six Kali Linux tools. All activities were non-destructive and used only publicly available information.
WHOIS I ran whois networkwalks.com to obtain publicly available domain registration information. Key findings:
Registrar: GoDaddy.com LLC
Name Servers: NS6135.HOSTGATOR.COM / NS6136.HOSTGATOR.COM
Domain created: 2019-11-06 · Expires: 2027-11-06
DNSSEC: unsigned
All four EPP statuses (clientDeleteProhibited, clientRenewProhibited, clientTransferProhibited, clientUpdateProhibited) are set — domain is locked against unauthorized transfers
WhatWeb I ran whatweb networkwalks.com | tee whatweb-output.txt. Key findings:
CMS: WordPress 7.1
Plugin: WordPress Download Manager 3.3.58
Web server: Apache
IP: 192.232.216.135 (United States)
Frameworks: Bootstrap 7.1, jQuery 3.7.1
Email exposed: info@networkwalks.com
Headers: x-nginx-cache, x-endurance-cache-level (EIG/HostGator caching layer)
HTTP 301 redirect from http:// to https:// confirmed
nslookup I ran nslookup networkwalks.com | tee nslookup-output.txt using Google's DNS server (8.8.8.8). Result: networkwalks.com resolves to 192.232.216.135 — consistent with WhatWeb output.
Note: A DNS configuration issue was identified and resolved during this lab. The VirtualBox NAT Network proxy at 10.0.0.1 was refusing DNS queries. This was fixed by switching Kali's DNS resolver to 8.8.8.8 using the nmcli command.
curl -I I ran curl -I https://networkwalks.com to inspect raw HTTP response headers. Key findings:
server: Apache — confirms Apache web server
x-nginx-cache: WordPress — confirms EIG/HostGator platform-level Nginx caching in front of Apache backend
x-endurance-cache-level: 0 — HostGator's endurance caching platform confirmed
link: <https://networkwalks.com/wp-json/> — WordPress REST API endpoint exposed in headers, which could be used to enumerate users via /wp-json/wp/v2/users
set-cookie: __wpdm_client — WordPress Download Manager plugin cookie confirmed
Note: First attempt used hhttps:// (double h typo) — curl correctly returned "Protocol not supported." Corrected on second attempt.
wafw00f I ran wafw00f networkwalks.com. Result: ModSecurity (SpiderLabs) WAF detected. This is an Apache module-based WAF, fully consistent with the server: Apache header. This finding is significant for later phases — noisy or automated attacks against this target will likely be blocked or logged by ModSecurity.
Note: First attempt used wafwoof (typo) which returned "command not found." Corrected to wafw00f (w-a-f-w-zero-zero-f).
dnsrecon I ran dnsrecon -d networkwalks.com. Key findings:
SOA/NS: ns6135.hostgator.com / ns6136.hostgator.com (Bind 9.16.23-RH)
A record: networkwalks.com → 192.232.216.135
MX: mail.networkwalks.com → 192.232.216.135 (mail hosted on same IP as website)
SPF TXT: v=spf1 +a +mx ip4:50.87.144.87 include:websitewelcome.com ~all (websitewelcome.com is an EIG brand, confirming HostGator/EIG hosting)
Google-site-verification TXT record present (active SEO/Google Search Console management)
SRV _autodiscover._tcp → cpanelemaildiscovery.cpanel.net (cPanel-based email hosting confirmed)
Zone transfer (AXFR): not permitted — correct security configuration

4.2 W2-PM2 — GHDB / Google Dorking (networkwalks.com)
I tested networkwalks.com against five categories of Google dorks to search for publicly indexed sensitive content or misconfigurations.
Dork Used
Result
site:networkwalks.com filetype:pdf
Returned intentionally published training PDFs (ARP cheat sheet, SNMP cheat sheet, IT Diploma course schedule) — all deliberate public marketing content, not sensitive
site:networkwalks.com intitle:"index of"
No results — no open directory listings detected
site:networkwalks.com ext:sql | ext:bak | ext:old | ext:log
No results — no exposed backup or database files indexed
site:networkwalks.com inurl:wp-content/plugins
No results — no exposed WordPress plugin directory listings
site:networkwalks.com inurl:readme.txt
No results — no WordPress/plugin readme files indexed
site:networkwalks.com "wordpress download manager"
No results — no plugin-specific sensitive pages indexed

Summary: GHDB dorking returned only intentionally published public content. No misconfigurations, exposed backup files, open directory listings, or sensitive documents were found via Google indexing. This indicates reasonable hygiene in terms of what the target allows Google to index.

4.3 W2-PM3 — Maltego (networkwalks.com) — TOOLING ISSUE
Attempted: Maltego CE v4.12.1 was installed successfully on Kali Linux via sudo apt install maltego. Account registration on maltego.com was completed (name, email, country, phone number provided). However, the desktop application consistently failed to complete the OAuth browser authentication handshake — it displayed "Logging in... Waiting for authentication signal from your browser" indefinitely, and the browser login form looped back to the login page after correct credentials were entered. Multiple attempts over two sessions were made, including full restarts of both Maltego and the browser.
Resolution: After exhausting reasonable troubleshooting steps (verifying system clock, clearing browser sessions, restarting the wizard), Maltego was skipped to preserve time for remaining modules.
Impact on objectives: The primary PM3 objective was email address enumeration for networkwalks.com. This was covered by other tools: info@networkwalks.com was already identified by WhatWeb (from page content), dnsrecon (from DNS records), and wafw00f. The email and infrastructure relationship mapping that Maltego would have visualized was documented textually in PM1 findings above.

4.4 W2-PM4 — theHarvester (microsoft.com)
theHarvester is a passive OSINT tool that queries public sources (search engines, certificate transparency logs, DNS datasets) to harvest email addresses, subdomains, IPs and ASNs without ever sending packets to the target. Both tasks targeted microsoft.com as a public OSINT exercise.
Task 1 — Baidu source, limit 1000 Command: theHarvester -d microsoft.com -l 1000 -b baidu | tee harvester-baidu-output.txt
Results:
Emails found: 0
IPs found: 0
Hosts found: 2
community.fabric.microsoft.com
wcpstatic.microsoft.com
People found: 0
Baidu's index returned minimal results for microsoft.com — only 2 subdomains and no exposed emails. This reflects both Microsoft's strong email hygiene and Baidu's limited indexing of English-language corporate content.
Task 2 — All sources, limit 50 Command: theHarvester -d microsoft.com -l 50 -b all | tee harvester-all-output.txt
Results:
Emails found: 3
IPs found: 148
Hosts found: 9,964
Interesting URLs found: 19
ASNs found: 7
LinkedIn links: 0
People found: 0
Note: Many sources returned "Missing API key" errors (bevigil, bitbucket, builtwith, securityscorecard, etc.) — these require paid API keys and were skipped automatically. Free sources (certificate transparency logs, DNS datasets, public search engines) still returned substantial results.
Key learning — source comparison:
Metric
Baidu only
All sources
Hosts
2
9,964
Emails
0
3
IPs
0
148

This demonstrates that using a single data source produces severely limited results. Using all available free sources revealed nearly 10,000 subdomains — each representing a potential attack surface for further enumeration in later phases.

4.5 W2-PM5 — Network Scanning with Zenmap (Local LAN)
Zenmap (the official Nmap GUI) was installed on Windows 10 from nmap.org/download.html (nmap-7.991-setup.exe, which includes Npcap and Zenmap). A Ping Scan (nmap -sn) was used to discover live hosts on the local subnet without port scanning.
Task 2 — Local IP and subnet identification Command: ipconfig (Windows PowerShell)
Active adapter: Wireless LAN adapter Wi-Fi 2
IPv4 Address: 192.168.18.167
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.18.1
Scan target: 192.168.18.0/24
Task 3–6 — Ping scan results Target: 192.168.18.0/24 | Profile: Ping scan | Command: nmap -sn 192.168.18.0/24
256 IP addresses scanned in 7.07 seconds
Hosts found: 2
Host
IP Address
MAC Address
Device Type
Router/Gateway
192.168.18.1
20:53:83:27:0D:BE
Huawei Technologies
My PC
192.168.18.167
(local — use ipconfig /all)
Windows 10 PC

Task 7 — Topology Network topology generated in Zenmap's Topology tab and saved as PDF (zenmap-topology.pdf) to the Windows Desktop.

5. Risk Analysis / Impact
#
Risk / Finding
Evidence / Observation
Potential Impact
Risk Level
1
CMS and plugin versions exposed
WhatWeb: WordPress 7.1, WP Download Manager 3.3.58
Attackers can cross-reference these exact versions against CVE databases to identify known exploits
Medium
2
WordPress REST API endpoint exposed in headers
curl -I: link: <https://networkwalks.com/wp-json/>
/wp-json/wp/v2/users can be used to enumerate valid usernames without authentication
Medium
3
Email address indexed publicly
WhatWeb/dnsrecon: info@networkwalks.com
Can be used as phishing target or for password spray attacks against admin accounts
Low
4
Server IP identifiable
nslookup/dnsrecon: 192.232.216.135
Enables infrastructure mapping and targeted scanning
Low
5
WAF technology identifiable
wafw00f: ModSecurity (SpiderLabs)
Knowing the WAF type allows attackers to research specific WAF bypass techniques
Low
6
Hosting platform fingerprinted
curl headers + dnsrecon: HostGator/EIG platform, cPanel email
Platform-specific vulnerabilities or misconfigurations may be researched and targeted
Low
7
DNS infrastructure exposed
dnsrecon: NS, MX, SPF, SRV, Bind version 9.16.23-RH
DNS information assists in building a complete infrastructure map
Low
8
Large subdomain attack surface (microsoft.com)
theHarvester: 9,964 subdomains discovered
Each subdomain is a potential entry point — staging/dev subdomains are often less hardened
Informational
9
Live hosts visible on local LAN
Zenmap: 2 hosts identified
Unknown or unauthorized devices on a network could indicate a security issue
Low

Risk level key: Critical | Medium | Low | Informational
All findings above are observations from passive footprinting exercises, not confirmed vulnerabilities.

6. Recommendations
Review publicly exposed technology information — WordPress version and plugin versions are visible in page meta tags. Removing the MetaGenerator tag in WordPress settings reduces version disclosure.


Restrict WordPress REST API user enumeration — The /wp-json/wp/v2/users endpoint should be disabled or restricted for unauthenticated requests to prevent username enumeration.


Keep CMS and plugins updated — WordPress 7.1 and WordPress Download Manager 3.3.58 should be regularly checked against current CVEs and updated promptly.


Review HTTP headers — Remove or obscure the link: header pointing to the REST API endpoint, and consider adding a X-Robots-Tag: noindex for sensitive admin-related endpoints.


Maintain WAF tuning — ModSecurity is already deployed and provides meaningful protection. Rules should be reviewed and updated regularly to remain effective against new attack patterns.


Review DNS records regularly — DNS records should be checked periodically to ensure only required services are publicly exposed. Bind version disclosure via CHAOS queries should be suppressed.


Perform regular internal network discovery — Organizations should periodically scan their own networks to identify active and unexpected devices.


Investigate unknown devices — Any unexpected device discovered during network scanning should be investigated and verified against an asset register.


Maintain network documentation — Network topology and device information should be documented and updated after every significant infrastructure change.


Perform security testing within authorized scope — Reconnaissance and scanning must only be performed where appropriate written authorization has been provided and documented.



7. Conclusion
During Week 2 of my Cybersecurity & Ethical Hacking internship at Networkwalks (Batch B083), I completed five practical modules covering passive footprinting, OSINT, GHDB dorking, email/subdomain harvesting, and local network scanning.
The footprinting activities (PM1) demonstrated how much information is publicly available about a target before any active probing begins. A single domain revealed its CMS and plugin versions, web server, hosting provider, email infrastructure, caching architecture, WAF technology, and contact email — all through passive, non-intrusive tools. This is exactly the information an attacker collects before deciding whether and how to proceed.
The GHDB activity (PM2) showed that good indexing hygiene matters — networkwalks.com had no sensitive files or misconfigurations indexed by Google, which is a positive security indicator.
The theHarvester exercise (PM4) powerfully demonstrated the value of using multiple data sources. A single search engine (Baidu) returned only 2 subdomains, while querying all free sources together returned nearly 10,000 — each representing a potential attack surface that a defender should be aware of.
The Zenmap exercise (PM5) completed the transition from reconnaissance to active discovery, showing how quickly a network topology can be mapped using a standard ping scan.
One tooling challenge was encountered: Maltego CE (PM3) could not be activated due to a persistent browser OAuth authentication loop. This was documented transparently and the PM3 objectives were addressed through alternative tools already used in PM1.
Overall, Week 2 reinforced that information gathering is not a minor preliminary step — it is a critical phase that directly determines the quality and targeting of everything that follows in a penetration test.

8. Evidence Collected
(Insert screenshots here in the following order:)
whois networkwalks.com output
whatweb networkwalks.com output
nslookup networkwalks.com output
curl -I https://networkwalks.com output
wafw00f networkwalks.com output
dnsrecon -d networkwalks.com output
GHDB dork results (Google browser screenshots)
Maltego installation + login loop error screenshots
theHarvester Baidu output
theHarvester all-sources summary output
Zenmap ping scan results
Zenmap topology PDF

👤 Author
Kwanele Dube Cybersecurity Intern — Batch B083 Networkwalks Cybersecurity Internship
LinkedIn: https://www.linkedin.com/posts/kwanele-dube-b35659377_networkwalks-cybersecurity-ethicalhacking-share-7506314826168786944-nvhD/
📌 Project Information
Program: Networkwalks Cybersecurity Internship | Week: 02 | Repository: GitHub
-End-

