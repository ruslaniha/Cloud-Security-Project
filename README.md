  # 🛡️ Cloud Security Project — SOC Infrastructure

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Platform](https://img.shields.io/badge/Cloud-AWS-orange)
![Score](https://img.shields.io/badge/SCA%20Score-86%25-brightgreen)
![Domain](https://img.shields.io/badge/Domain-calaris.online-blue)

> Final project for Code Academy cybersecurity program.
> A fully functional cloud-based security infrastructure featuring
> a WAF, SIEM, automated incident response, threat intelligence,
> and enterprise mail security — all deployed on AWS.

---

## 📋 Project Overview

| Component | Technology | Status |
|---|---|---|
| ☁️ Cloud Provider | AWS EC2 + Route 53 | ✅ |
| 🔥 WAF | Nginx + ModSecurity | ✅ |
| 🌐 Web Server | WordPress + Docker | ✅ |
| 📊 SIEM | Wazuh (XDR + SIEM) | ✅ |
| 🎫 Ticketing | TheHive + Cortex | ✅ |
| 🔬 Threat Intel | VirusTotal + AbuseIPDB | ✅ |
| 📧 Mail Security | Google Workspace + SPF/DKIM/DMARC | ✅ |
| 🔒 SCA Score | CIS Ubuntu 22.04 Benchmark | ✅ 86% |

---

## 🏗️ Infrastructure Architecture
```
                        Internet
                           │
                    calaris.online
                    (GoDaddy + Route 53)
                           │
                  ┌────────▼────────┐
                  │   WAF Instance  │
                  │ Nginx + ModSec  │◄──── Wazuh Agent
                  └────────┬────────┘
                           │ (only WAF IP allowed)
                  ┌────────▼────────┐
                  │WordPress Server │
                  │  Docker Compose │
                  └─────────────────┘

                  ┌─────────────────┐
                  │  Wazuh Server   │
                  │   (SIEM/XDR)    │
                  └──┬──────┬───┬───┘
                     │      │   │
              ┌──────▼─┐ ┌──▼──┐ ┌▼──────────┐
              │Virus   │ │Abuse│ │  TheHive  │
              │Total   │ │IPDB │ │ + Cortex  │
              └────────┘ └─────┘ └───────────┘
                              │
                        Gmail Alerts
```

---

## ✅ Requirement 1 — Cloud Environment Setup

Deployed **4 AWS EC2 instances**:

| Instance | OS | Purpose |
|---|---|---|
| WAF Server | Ubuntu 22.04 | Nginx + ModSecurity reverse proxy |
| Web Server | Ubuntu 22.04 | WordPress via Docker Compose |
| Wazuh Server | Ubuntu 22.04 | SIEM + XDR |
| TheHive Server | Ubuntu 22.04 | Ticketing + Cortex |

- WordPress deployed using **Docker Compose**
- ModSecurity WAF installed as Nginx module
- Custom **403 block page** configured for WAF alerts

---

## ✅ Requirement 2 — Firewall & Reverse Proxy Configuration

- Configured **AWS Security Groups** to restrict WordPress server:
  - Port 80/443 accessible **only from WAF instance IP**
  - Direct public access to WordPress completely blocked
  - Only attacker's own public IP allowed for administration
- Nginx configured as **reverse proxy** — all traffic routed through WAF
- **Tested:** XSS injection attempt → WAF blocked with custom 403 page ✅

---

## ✅ Requirement 3 — Domain, SSL & HTTPS

- Registered domain **calaris.online** on GoDaddy
- Created **AWS Route 53** hosted zone
- Added **A record** pointing to WAF instance IP
- Updated GoDaddy nameservers to AWS NS records
- Issued **SSL certificate** via Certbot (Let's Encrypt)
- Configured HTTPS redirect in Nginx
```bash
curl -I https://calaris.online
# HTTP/1.1 200 OK ✅
```

---

## ✅ Requirement 4 — Wazuh Integrations + Active Response + SCA

### 📊 Wazuh Agent
- Installed Wazuh Agent on WAF instance
- Connected to Wazuh Server via port **1514 TCP/UDP**
- Logs visible in **Threat Hunting** dashboard

### 🦠 VirusTotal Integration
- API key configured in `ossec.conf`
- **FIM (File Integrity Monitoring)** enabled via `agent.conf`
- **Test:** EICAR malware file dropped → detected by Wazuh →
  VirusTotal reported **62/68 engines** flagged as malicious ✅
- **Active Response:** malicious file **automatically deleted** ✅

### 🚫 AbuseIPDB Integration
- Custom Python script `custom-abuseipdb.py` deployed
- Custom rules added to `local_rules.xml`
- **Test:** SSH brute force from IP `139.59.93.213` →
  AbuseIPDB returned **100% abuse confidence** (ISP: DigitalOcean) ✅

### 🔥 Active Response — IP Auto-Block
Configured `firewall-drop` active response for web attack rules:

| Rule ID | Trigger |
|---|---|
| 31103 | SQL Injection |
| 31104 | Common web attack |
| 31105 | XSS attempt |
| 31106 | Web attack (200 response) |
| 31171 | Generic web attack |

- **Test:** XSS attack on calaris.online →
  attacker IP **automatically banned** by firewall ✅
- Wazuh confirms: `Host Blocked by firewall-drop Active Response`

### 📧 Mail Integration
- **Postfix** configured as SMTP relay via Gmail App Password
- Wazuh alerts (severity 10+) sent to email automatically
- **Test:** Malware detection triggered email alert to Gmail ✅

### 🔒 SCA — Security Configuration Assessment
- Ran **CIS Ubuntu Linux 22.04 Benchmark** via Wazuh

| Metric | Before | After |
|---|---|---|
| Score | 42% | **86%** ✅ |
| Failed Checks | 103 | 25 |

Key fixes: disabled Telnet, disabled root SSH login,
hardened kernel parameters, applied CIS remediation steps.

---

## ✅ Requirement 5 — TheHive Ticketing

- Deployed **TheHive + Elasticsearch** via Docker Compose
- Created organization **CS201_Code**
- Users: Admin + SOC Analyst
- Integrated **Wazuh → TheHive** via custom Python script
- **Test:** SSH brute force detected → ticket automatically
  created in TheHive → assigned to SOC Analyst ✅

---

## ✅ Requirement 6 — Cortex Analyzers

- Deployed **Cortex** alongside TheHive
- Configured analyzers:
  - **VirusTotal_GetReport** — file/hash analysis
  - **AbuseIPDB** — IP reputation analysis
- **Test:** Ran both analyzers from TheHive alert →
  results returned successfully ✅

---

## ✅ Requirement 7 — Google Workspace Mail Security

- Set up **Google Workspace** trial with domain calaris.online
- Verified domain via **TXT record** in Route 53
- Added **MX records** pointing to Google mail servers
- Configured full email authentication:

| Record | Status |
|---|---|
| SPF | ✅ Configured |
| DKIM | ✅ Configured |
| DMARC | ✅ Configured |

- **Verified with mail-tester.com → Score: 9/10** ✅

---

## 📈 Final Results

| Area | Result |
|---|---|
| WAF Protection | ✅ XSS/SQLi blocked |
| IP Auto-Block | ✅ Active response working |
| Malware Detection | ✅ Real-time via VirusTotal |
| SCA Hardening | ✅ 42% → 86% |
| Incident Response | ✅ Auto-tickets in TheHive |
| Cortex Analysis | ✅ VT + AbuseIPDB integrated |
| Mail Security | ✅ SPF/DKIM/DMARC — 9/10 |

---

## 🛠️ Tools & Technologies

`AWS` `Nginx` `ModSecurity` `Docker` `WordPress`
`Wazuh` `TheHive` `Cortex` `VirusTotal` `AbuseIPDB`
`Postfix` `Google Workspace` `Certbot` `Route 53` `GoDaddy`

---

[📄 View Project PDF](./Cloud%20Security%20Project.pdf)

*Code Academy — Final Cybersecurity Project*
*Cloud Security Infrastructure on AWS • calaris.online*
