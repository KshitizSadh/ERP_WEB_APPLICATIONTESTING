

# Vulnerability Assessment Report

**Target:** [https://erp.ramanujancollege.ac.in/login](https://erp.ramanujancollege.ac.in/login)
**Assessment Date:** March 2025
**Assessment Type:** Web Application & Network Security Testing

---

## 1. Executive Summary

The security audit of the Ramanujan College ERP portal identified several potential vulnerabilities related to web security misconfigurations, insecure cookie settings, and an exposed VoIP (SIP) service. No critical SQL Injection vulnerabilities were detected during automated and manual testing.

Overall, the system implements minimal security controls, but the existing weaknesses could expose it to clickjacking, session hijacking, and VoIP-related attacks.

| Status                       | Summary                                                |
| ---------------------------- | ------------------------------------------------------ |
| ✅ No SQL Injection           | Confirmed safe from SQLi attacks                       |
| ⚠️ Web Security Issues Found | Missing security headers and cookie flags              |
| ⚠️ Exposed SIP Service       | VoIP (SIP) service on port 5060 potentially vulnerable |

---

## 2. Assessment Methodology

The following tools and techniques were utilized during the assessment:

* **Network Scanning:** Nmap
* **Web Security Testing:** Nikto, Header Analysis (curl)
* **SQL Injection Testing:** SQLMap, Manual SQLi attempts
* **SIP Service Enumeration:** SIPVicious

---

## 3. Identified Vulnerabilities

### 3.1 Web Security Misconfigurations

**Description:**
Several key security headers are missing from the HTTP responses, increasing risks such as clickjacking, Man-in-the-Middle (MITM) attacks, and MIME-based attacks.

**Evidence:**

```
curl -I https://erp.ramanujancollege.ac.in
❌ X-Frame-Options: Missing  
❌ Strict-Transport-Security (HSTS): Missing  
❌ X-Content-Type-Options: Missing
```

**Impact:**

* Vulnerable to clickjacking by embedding login pages in malicious iframes.
* HTTPS downgrade attacks become easier without HSTS.
* MIME-type sniffing attacks can execute malicious content.

**Recommendations:**

* Add `X-Frame-Options: DENY` to nginx configuration.
* Enable HSTS header to enforce HTTPS connections: `Strict-Transport-Security: max-age=31536000; includeSubDomains`
* Add `X-Content-Type-Options: nosniff` header.

---

### 3.2 Cookie Security Issues

**Description:**
Session cookies (`XSRF-TOKEN`, `laravel_session`) lack `Secure` and `HttpOnly` flags.

**Evidence:**

```
Set-Cookie: XSRF-TOKEN=.; path=/;  
Set-Cookie: laravel_session=.; path=/;
```

* Secure flag missing → Cookies could be sent over insecure HTTP.
* HttpOnly flag missing → Cookies accessible to JavaScript, vulnerable to XSS.

**Impact:**

* Attackers may hijack user sessions via XSS or network sniffing.

**Recommendations:**
Set cookies with `Secure`, `HttpOnly`, and `SameSite=Strict` flags. Example:

```
Set-Cookie: XSRF-TOKEN=.; Secure; HttpOnly; SameSite=Strict
```

---

### 3.3 Outdated Web Server Version (Nginx 1.18.0)

**Description:**
The web server runs Nginx 1.18.0, which has known vulnerabilities.

**Evidence:**

```
Server: nginx/1.18.0 (Ubuntu)
```

**Impact:**
Potentially exploitable vulnerabilities due to outdated software.

**Recommendations:**
Update Nginx to the latest stable release:

```
sudo apt update && sudo apt upgrade nginx
```

---

### 3.4 Exposed SIP (VoIP) Service on Port 5060

**Description:**
SIP service (port 5060) is open and potentially vulnerable to unauthorized access.

**Evidence:**

```
PORT     STATE SERVICE  
5060/tcp open  sip
```

**Impact:**

* Possible VoIP phishing (vishing) and call interception.
* Brute-force or unauthorized use of SIP accounts.

**Recommendations:**

* Restrict SIP access via firewall to trusted IP addresses only.
* Enable SIP authentication and encryption (e.g., SIP over TLS).

---

## 4. Summary of Security Recommendations

| Area                 | Action Item                                                              |
| -------------------- | ------------------------------------------------------------------------ |
| Web Security Headers | Implement missing headers: X-Frame-Options, HSTS, X-Content-Type-Options |
| Cookie Security      | Set Secure, HttpOnly, SameSite flags on session cookies                  |
| Web Server           | Upgrade Nginx to latest stable version                                   |
| SIP Service          | Restrict access and enable SIP authentication & encryption               |

---

## 5. Conclusion & Next Steps

The ERP portal is free from critical SQL injection issues but remains exposed to multiple misconfigurations that could allow session hijacking, clickjacking, and VoIP attacks.

### Next Steps:

* Implement the recommended HTTP security headers.
* Harden session cookie configurations to prevent hijacking.
* Update and patch the web server software promptly.
* Secure and limit access to the SIP service.

Addressing these issues will significantly improve the security posture of the Ramanujan College ERP system.

---

If you want, I can also help prepare remediation guides or scripts to fix these issues step-by-step. Would that be useful?
