# Footprinting, Reconnaissance & Network Scanning

**Author:** Mayuresh Sangale  
**Program:** Cybersecurity Professional B084 - Networkwalks  
**Week:** 02  
**Report:** W2-PM-FINAL

> **Lab-use note:** The activities documented here were performed for educational purposes against the lab-designated target and local network. Only test systems for which you have explicit authorization.

## Module 1 - Footprinting & Reconnaissance

**Target domain:** `networkwalks.com`  
**Tools:** WHOIS, WhatWeb, NSLookup, cURL, WAFW00F, DNSRecon

### Task 1 - WHOIS

```bash
whois networkwalks.com
```

**Purpose:** Retrieve public domain-registration information, including registrar, registration dates and name servers.

| Field | Observed value |
|---|---|
| Domain | `NETWORKWALKS.COM` |
| Registrar | GoDaddy.com, LLC |
| Creation date | 2019-11-06 |
| Expiry date | 2027-11-06 |
| Name servers | NS6135.HOSTGATOR.COM, NS6136.HOSTGATOR.COM |
| DNSSEC | Unsigned |

![WHOIS terminal output](screenshots01-whois.png)

### Task 2 - WhatWeb

```bash
whatweb networkwalks.com
```

**Purpose:** Identify technologies and services exposed by the target website.

| Item | Observed value |
|---|---|
| Web server | Apache |
| IP address | `192.232.216.135` |
| CMS | WordPress |
| WordPress Download Manager | 3.3.76 |
| jQuery | 3.7.1 |
| Bootstrap | 7.1.4 (as reported by WhatWeb) |
| Other | Google Tag Manager, HTML5 |
| Page title | Networkwalks Academy |
| Redirect | HTTP to HTTPS (301) |

![WhatWeb output](screenshots02-whatweb.png)

### Task 3 - NSLookup

```bash
nslookup networkwalks.com
```

**Purpose:** Resolve the domain name to its IP address.

| Item | Observed value |
|---|---|
| DNS server | `8.8.8.8` |
| Resolved IP | `192.232.216.135` |

![NSLookup output](screenshots03-nslookup.png)

### Task 4 - HTTP response headers with cURL

```bash
curl -I https://networkwalks.com
```

**Purpose:** Inspect the HTTP response headers returned by the target.

- Response shown: `HTTP/2 200` for the HTTPS request.
- Server: Apache.
- `X-nginx-cache: WordPress`.
- Content type: `text/html; charset=UTF-8`.
- WordPress-related cookie: `__wpdm_client` (HttpOnly).
- Referrer policy: `no-referrer-when-downgrade`.
- Permissions-Policy header is present.

![cURL response headers](screenshots04-curl-headers.png)

### Task 5 - Web Application Firewall detection

```bash
wafw00f networkwalks.com
```

**Purpose:** Identify whether a web application firewall is detected.

**Result:** WAFW00F reported that `https://networkwalks.com` is behind **ModSecurity (SpiderLabs)**. The tool reported 2 requests.

![WAFW00F output](screenshots05-wafw00f.png)

### Task 6 - DNS enumeration

```bash
dnsrecon -d networkwalks.com
```

**Purpose:** Enumerate publicly available DNS records.

| Record | Observed value |
|---|---|
| NS | `ns6135.hostgator.com` (50.87.144.87), `ns6136.hostgator.com` (192.232.216.131) |
| MX | `mail.networkwalks.com` (192.232.216.135) |
| A | `networkwalks.com` -> `192.232.216.135` |
| TXT | Google site verification and SPF record (`v=spf1 +a +mx ...`) |
| SRV | `_autodiscover._tcp` -> `cpanelemaildiscovery.cpanel.net` (8 records) |

**Observation:** The SOA lookup timed out and no DNSSEC answer was returned. DNSRecon reported 8 records in the SRV enumeration.

![DNSRecon output](screenshots06-dnsrecon.png)

## Module 4 - Footprinting & Reconnaissance with theHarvester

**Target domain:** `microsoft.com`  
**Tool:** theHarvester 4.11.1 (Kali Linux)

`theHarvester` is an OSINT tool that gathers publicly available information such as email addresses, subdomains, hosts and names from supported public data sources.

### Task 1 - Search using Baidu

```bash
theHarvester -d microsoft.com -l 1000 -b baidu
```

**Purpose:** Search Baidu for public information associated with the target, with a result limit of 1000.

| Item | Result |
|---|---|
| Target | `microsoft.com` |
| Source | Baidu |
| IPs found | None |
| Emails found | None |
| People found | None |
| Hosts/subdomains found | None |

**Observation:** Baidu returned no data for this domain at the time of the scan. Results from third-party sources can vary over time.

![theHarvester Baidu output](screenshots07-theharvester-baidu.png)

### Task 2 - Search using all supported sources

```bash
theHarvester -d microsoft.com -l 50 -b all
```

**Purpose:** Query all supported sources with a result limit of 50.

The output showed that theHarvester read its API-key configuration file at `/etc/theHarvester/api-keys.yaml`. Several sources could not be queried because API keys were not configured.

| Source | Status shown |
|---|---|
| BeVigil | Missing API key |
| Bitbucket | Missing API key |
| BufferOverrun | Missing API key |
| BuiltWith | Missing API key |
| Brave Search | Missing API key |
| Censys | Missing API key (ID and/or Secret) |
| CriminalIP | Missing API key |
| Dehashed | Missing API key |
| DNSDumpster | Missing API key |

**Observation:** Sources requiring credentials return no data until their API keys are configured in the relevant configuration file.

![theHarvester all-sources output](screenshots08-theharvester-all-sources.png)

## Module 5 - Local IP configuration evidence

The supplied Windows `ipconfig` screenshots show these adapter values:

| Adapter / field | Observed value |
|---|---|
| Ethernet IPv4 | `192.168.56.1` |
| Wi-Fi IPv4 | `192.168.29.245` |
| Wi-Fi subnet mask | `255.255.255.0` |
| Wi-Fi default gateway | `192.168.29.1` |
| Wi-Fi LAN subnet | `192.168.29.0/24` |

The Ethernet address appears on a separate adapter. The Wi-Fi adapter is the local LAN configuration shown in the supplied evidence. The screenshots supplied for this revision document IP configuration only; they do **not** include a new Zenmap host-discovery scan or topology screenshot, so no new live-host count or MAC-address results are claimed here.

![Windows ipconfig adapter overview](screenshots09-ipconfig-adapters.png)

![Windows Wi-Fi adapter details](screenshots10-ipconfig-wifi-details.png)

## Skills Practiced

- Domain footprinting and WHOIS analysis
- Web-technology fingerprinting
- DNS resolution and DNS-record enumeration
- HTTP response-header inspection
- Web application firewall detection
- Passive reconnaissance using theHarvester
- Understanding API-key requirements for OSINT data sources
- Reading Windows IP configuration and identifying a local subnet

## Conclusion

The exercises demonstrated how command-line tools in Kali Linux can collect and interpret public domain, web-technology, HTTP-header, WAF and DNS information. theHarvester demonstrated how OSINT results depend on the selected data source and API-key configuration. The Windows network-configuration evidence identified the local Wi-Fi IPv4 address, subnet mask and default gateway. These techniques are useful for understanding an organization's externally visible footprint and documenting a lab reconnaissance workflow.

---

**Author:** Mayuresh Sangale  
**Cybersecurity Professional B084**  
**Program:** Cybersecurity program at Networkwalks | **Week:** 02 | **Repository:** GitHub
