# Footprinting and Reconnaissance: A Pentest Report

**Author:** Chukwu PraiseGod

**Date:** 17 September 2026

**Classification:** Educational / Authorised Lab

---

## Overview

This project demonstrates a full **reconnaissance lifecycle** across multiple environments, public web infrastructure, passive OSINT, enterprise-scale attack surface mapping, and local network discovery.

The objective was not exploitation, but **understanding how much intelligence an attacker can gather without triggering defenses**.

---

## Executive Summary

* Passive techniques alone exposed:

  * **10+ unsecured IP cameras** streaming live feeds publicly
  * **~10,000 subdomains** tied to a single enterprise domain
* A standard WordPress site revealed:

  * CMS version, plugins, hosting stack, and defensive controls 
* A simple LAN scan identified:

  * **IoT devices with potential firmware risk**
* No exploitation was required to uncover meaningful attack surface intelligence

> This proves that reconnaissance is not only a preliminary step, but the **most information-rich phase of an attack**.

---

## Methodology

I followed a structured recon workflow:

1. **Passive OSINT**

   * WHOIS, GHDB, Certificate Transparency, Search Engines
2. **Low-Impact Active Recon**

   * DNS queries, HTTP header inspection, fingerprinting
3. **Network Discovery**

   * Local subnet scanning and device identification

**Tools Used:**
`whois`, `whatweb`, `dnsrecon`, `nslookup`, `curl`, `wafw00f`, `theHarvester`, `Nmap`, `Zenmap`, Google GHDB

---

## PM1: Target Reconnaissance (networkwalks.com)

### What I Discovered

* **Technology Stack**

  * WordPress 7.1 (exposed via meta tag)
  * Plugin: Download Manager 3.3.58
  * Apache on shared HostGator infrastructure

* **Security Controls Present**

  * HTTPS enforced
  * ModSecurity WAF detected
  * Secure cookie flags enabled

* **Security Gaps Identified**

  * Missing headers:

    * Content-Security-Policy
    * HSTS
    * X-Frame-Options
  * DNSSEC not enabled
  * SPF uses soft-fail (`~all`)

* **Infrastructure Exposure**

  * BIND version (`9.16.23-RH`) disclosed via DNS
  * cPanel usage inferred via SRV records

### Evidence

**Whois Output** 
![whois](/screenshots/whois-page1.png)

**WhatWeb Output**
![whatweb](/screenshots/whatweb-result.png)

**curl Headers**
![curl](/screenshots/curl-result.png)

**dnsrecon Results**
![dnsrecon](/screenshots/recon-output-files.png)

**nslookup Output**
![nslookup](/screenshots/nslookup-result.png)

**Wafw00f Output**
![wasw00f](/screenshots/wafw00f-result.png)

---

## PM2: Passive OSINT (Google Dorking)

### High-Impact Finding

Over **10 publicly accessible IP cameras** were identified using GHDB dorks such as:

```
intitle:"webcamXP" inurl:8080
```

* No authentication required
* Live feeds accessible over HTTP
* Likely unknown to device owners

### Why This Matters

These exposures enable:

* Physical surveillance
* Behavioral tracking
* Facility reconnaissance

### Evidence
![GHDB](/screenshots/ghdb-cam-result.png)

---

## PM4: Enterprise OSINT (microsoft.com)

### Results Summary

* **Subdomains discovered:** ~9,957
* **Emails identified:** 3
* **IP addresses:** 46
* **ASNs:** 6

### Observations

* Presence of:

  * `-dev`, `-test`, `-ppe`, `-staging` environments
  * Internal-looking `corp` domains (via CT logs)
* Demonstrates how **certificate transparency leaks internal structure**

### Key Lesson

> The attack surface of large organisations is **partially visible by design**, even without direct interaction.

### Evidence

**theHarvester output with Baidu as data source**
![theHarvester baidu](/screenshots/theHarvester-baidu.png)

**theHarvester output with -all data source**
![theHarvester all](/screenshots/theHarvester-all-1.png)

![theHarvester all](/screenshots/theHarvester-all-2.png)

![theHarvester all](/screenshots/theHarvester-all-3.png)

![theHarvester all](/screenshots/theHarvester-all-4.png)

---

## PM5: Local Network Discovery using Zenmap (192.168.1.0/24)

### Scan Results

* **Total hosts discovered:** 4
* **Notable device:** IoT system (Qingdao manufacturer)

### Risk Insight

* IoT devices often:

  * Run outdated firmware
  * Use weak/default credentials
  * Lack segmentation

### Security Concern

The IoT device represents a **pivot point risk** if compromised.

### Evidence 

**Zenmap ping scan result output**
![Zenmap ping scan result](/screenshots/zenmap-scan-result.png)

**Network Topology**
![Network topology](/screenshots/zenmap-network-topology.png)

---

## Risk Summary

| Severity | Count | Key Issues                              |
| -------- | ----- | --------------------------------------- |
| High     | 1     | Exposed IP cameras                      |
| Medium   | 4     | Version disclosure, missing headers     |
| Low      | 3     | SPF misconfig, DNSSEC absence, IoT risk |
| Info     | 5     | OSINT exposure, infrastructure insights |

---

## Recommendations

### Web Security (PM1)

* Remove version exposure from WordPress
* Implement security headers (CSP, HSTS, etc.)
* Suppress DNS version disclosure
* Enable DNSSEC
* Harden SPF and deploy DMARC

### OSINT Exposure (PM2)

* Secure internet-facing devices:

  * Enforce authentication
  * Disable public exposure
  * Use VPN-based access

### Network Security (PM5)

* Isolate IoT devices using VLANs
* Perform service enumeration (`nmap -sV -sC`)
* Apply firmware updates regularly

---

## Conclusion

From a single domain to a global enterprise footprint, this assessment demonstrates how **open-source intelligence and low-noise techniques** can reveal:

* Technology stacks
* Misconfigurations
* Hidden infrastructure
* Real-world security risks

---

## Portfolio Note

This report is part of my ongoing hands-on Cybersecurity Internship at NetworkWalks.

More labs and case studies available on my GitHub.
