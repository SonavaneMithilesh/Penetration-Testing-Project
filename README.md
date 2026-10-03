# Penetration-Testing-Project

## Penetration Testing on Mediroza General Hospital Website

---

# ⚠️ Liability Disclaimer

I have performed these activities only on the website where I had secured written permission or on devices/systems that I own myself. All these materials are for educational and research purposes only.

Do not use anything from this project to break the law. The instructor, the authors, and NetworkWalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility.

Misuse can lead to criminal charges, heavy fines, loss of your job, and a permanent record. In most countries, unauthorized access is a crime even when nothing is damaged.

**Author:** Mithilesh Vijay Sonavane
**Batch:** B083 | Week 4
**Target:** https://medirozahospital.com
**Engagement Type:** Black-box Penetration Test
**Duration:** 5 Days
**Authorization:** Written permission granted by Mediroza General Hospital via NetworkWalks training engagement

---

# 📌 1. Executive Summary

This engagement identified three significant vulnerabilities in the Mediroza General Hospital web infrastructure:

1. Authentication bypass through SQL Injection on the patient portal.
2. A publicly exposed database backup containing staff and shareholder records.
3. Weak passwords protecting confidential patient laboratory reports.

Together, these findings could allow an unauthorized attacker to access sensitive patient medical records and internal organizational information without valid credentials.

The assessment was conducted as part of an authorized NetworkWalks training engagement and was limited to the defined target and approved testing activities.

---

# 📌 2. Scope and Methodology

### Scope

The assessment was limited to:

* `medirozahospital.com`
* Its publicly accessible subpaths and resources within the authorized scope.

The following activities were excluded:

* Social engineering
* Denial-of-Service testing
* Destructive testing
* Any out-of-scope systems

### Tools Used

The following tools and utilities were used during the assessment:

* `whois`
* `nslookup`
* `dnsrecon`
* `whatweb`
* `wafw00f`
* `curl`
* NetworkWalks Hash Calculator
* NetworkWalks Password Cracker
* `pdfcrack`
* `john`

### Testing Approach

The penetration test followed the following methodology:

1. Passive reconnaissance
2. DNS and domain enumeration
3. Web technology fingerprinting
4. HTTP header analysis
5. WAF detection
6. Content discovery
7. Review of `robots.txt`
8. Directory and file enumeration
9. Authentication testing
10. Vulnerability validation
11. Controlled retrieval of exposed files
12. Offline password-strength testing
13. Risk assessment
14. Remediation recommendations

---

# 🔎 3. Findings and Proof of Exploitation

## 3.1 Reconnaissance & Scanning

Reconnaissance was performed to collect publicly available information about the target using authorized Kali Linux tools.

### 🔹 WHOIS

WHOIS was used to gather domain registration information, including registrar details, registration/expiry information, and name servers.



---

### 🔹 WhatWeb

WhatWeb was used to identify technologies and software components used by the target website.


---

### 🔹 NSLookup

NSLookup was used to resolve the target domain and identify its associated IP address and DNS information.



---

### 🔹 cURL

cURL was used to inspect HTTP response headers and identify information disclosed by the web server.

This can help identify:

* Web server technologies
* HTTP security headers
* Server information
* Redirects
* Other HTTP-level information



---

### 🔹 WAFW00F

WAFW00F was used to determine whether a Web Application Firewall (WAF) was present and detectable on the target.



---

### 🔹 DNSRecon

DNSRecon was used to gather DNS-related information about the target, including:

* DNS records
* Mail servers
* Name servers
* SPF information
* Other publicly available DNS configuration details



---

# 🔐 3.2 Gaining Access

The following vulnerabilities were identified and validated within the authorized testing environment.

---

## 🔴 Finding 1: SQL Injection — Authentication Bypass

**Severity:** Critical
**Location:** `/patient/login.php`

The patient portal login form was found to be vulnerable to SQL Injection.

The application appeared to construct a database query using user-controlled input without proper parameterization.

Example query structure:

```sql
SELECT * FROM users WHERE username='$username' AND password='$password'
```

During the authorized test, an SQL Injection payload was used to demonstrate that the authentication mechanism could be bypassed.

**Test Payload:**

```text
username=admin'-- -&password=pass
```

The payload caused the remainder of the SQL statement to be treated as a comment, allowing the authentication check to be bypassed.

### Evidence

The application returned an HTTP `302` redirect to `portal.php` and issued a new `PHPSESSID` session cookie, while a legitimate failed login attempt returned:

```text
Username not found
```


### Impact

Successful exploitation could allow an unauthenticated attacker to access the patient portal and potentially view sensitive patient information.

---

# 🔐 Finding 2: Weak PDF Encryption Passwords

**Severity:** Medium
**Location:** Patient Laboratory Report PDFs

Several patient laboratory reports were protected using PDF passwords.

During the authorized assessment, the password protection was tested offline using password-cracking tools and a standard dictionary wordlist.

The passwords were found to be weak and dictionary-based, including common values such as:

```text
123456
password
```

and other short/simple patterns.

### Password Testing Evidence


### File Content Evidence



### Impact

Weak document passwords reduce the effectiveness of PDF encryption and could expose sensitive medical information if an attacker obtains the protected files.

---

# 🔴 Finding 3: Sensitive Data Exposure — Publicly Accessible Database Backup

**Severity:** Critical
**Location:** `/old/mediroza_db_backup_2019.sql`

The website's `robots.txt` file disclosed the existence of an `/old/` directory.

The directory was accessible without authentication and contained an old SQL database backup.

The directory also allowed file listing, making the backup easier to discover and retrieve.

### Data Identified

The database backup contained information associated with:

### `staff` table

* Full names
* Job titles
* Departments
* Email addresses
* Phone numbers
* National identification information
* Monthly salary information

### `shareholders` table

* Names
* Shareholding percentage
* Number of shares
* Share class

### Evidence

Sensitive information should be appropriately redacted when publishing screenshots or project documentation.

### Impact

An unauthenticated attacker could potentially obtain sensitive employee, financial, and organizational information directly from the exposed database backup.

---

# 📊 4. Risk Rating Summary

| Finding                               | Severity | Impact                                                                            |
| ------------------------------------- | -------- | --------------------------------------------------------------------------------- |
| SQL Injection — Authentication Bypass | Critical | Potential unauthorized access to patient portal and sensitive medical information |
| Weak PDF Passwords                    | Medium   | Weak protection of sensitive patient documents                                    |
| Exposed Database Backup               | Critical | Unauthorized disclosure of employee, financial, and organizational information    |

---

# 🛠️ 5. Recommendations and Remediation

## 1. Fix SQL Injection

Use parameterized queries or prepared statements for all database operations.

Avoid constructing SQL queries by directly concatenating user-controlled input.

Example:

```text
User Input
    ↓
Input Validation
    ↓
Parameterized Query
    ↓
Database
```

---

## 2. Remove or Secure `/old/`

Legacy files and database backups should not be stored inside publicly accessible web directories.

Recommended actions:

* Remove old backups from the webroot.
* Store backups outside publicly accessible directories.
* Implement proper access controls.
* Disable directory listing/autoindex.
* Regularly review exposed files and directories.

---

## 3. Improve Password Security

Strong password policies should be implemented for sensitive documents and applications.

Passwords should:

* Be sufficiently long.
* Avoid common dictionary words.
* Avoid predictable patterns.
* Be unique.
* Be protected against automated guessing.

Where possible, sensitive documents should be accessed through an authenticated application rather than relying on static PDF passwords.

---

## 4. Do Not Use `robots.txt` as a Security Mechanism

`robots.txt` is not an access-control mechanism.

It only provides instructions to search engine crawlers.

Sensitive directories must be protected using:

* Authentication
* Authorization
* Server-side access controls
* Proper file permissions

---

## 5. Implement Rate Limiting and WAF Protection

Login endpoints should implement:

* Rate limiting
* Account lockout or progressive delays
* Input validation
* SQL Injection protection
* Security monitoring
* Appropriate WAF rules

---

# 📋 6. Attack Chain Summary

The assessment demonstrated how multiple weaknesses could be chained together:

```text
Reconnaissance
      ↓
robots.txt Discovery
      ↓
Hidden Directory Discovery
      ↓
Exposed Files / Database Backup
      ↓
Authentication Testing
      ↓
SQL Injection
      ↓
Authentication Bypass
      ↓
Patient Portal Access
      ↓
Sensitive Data Exposure
      ↓
Weak PDF Password Testing
```

This demonstrates why individual vulnerabilities should not be considered in isolation during a penetration test.

---

# 🎯 7. Conclusion

This authorized penetration-testing engagement demonstrated several security weaknesses within the assessed web application.

The assessment identified SQL Injection leading to authentication bypass, weak passwords protecting sensitive PDF documents, and an exposed database backup containing sensitive organizational information.

The findings demonstrate the importance of secure coding practices, strong authentication controls, proper backup management, access control, secure configuration, and continuous security testing.

The most important remediation actions are to eliminate SQL Injection vulnerabilities, remove sensitive backups from public directories, disable unnecessary directory listing, and strengthen protection for sensitive documents.

All testing activities described in this project were conducted as part of the authorized NetworkWalks Batch B083 training engagement.

---

## 👨‍💻 Author

**Mithilesh Vijay Sonavane**

**B.Voc Cyber Security & Forensic (CSF)**

**Batch:** B083 | Week 4

---

## ⚠️ Responsible Disclosure

This project is intended for educational, cybersecurity training, and authorized penetration-testing purposes only.

All testing should be performed only against systems for which explicit authorization has been obtained.

**Unauthorized testing or access to systems may be illegal.**
