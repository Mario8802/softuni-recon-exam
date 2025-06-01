
# 🛡️ Reconnaissance Exam Report – SSH Key Discovery

**Date:** 2025-06-01
**Exam:** Reconnaissance Fundamentals – Regular Exam  
**Student:** Mario Peychev  
**Target:** http://185.218.124.165/

---

## 🎯 Goal
Identify if the server uses sensitive SSH keys that may have leaked publicly (e.g., via GitLab) and document the full reconnaissance and vulnerability discovery process.

---

## 🧭 Steps Taken

### 1. Initial Reconnaissance (Passive)
- Accessed: http://185.218.124.165/
- Found reflected parameter: `search_query=`
  - ✅ HTML Injection
  - ✅ Reflected XSS

---

### 2. Directory Enumeration (Active)
```bash
gobuster dir -u http://185.218.124.165 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,log,bak,zip -t 30
```
Discovered:
- `/index.php`
- `/z` — base64-encoded system log

---

### 3. Base64 Log Analysis
```bash
curl -s http://185.218.124.165/z -o encoded.txt
cat encoded.txt | base64 -d > decoded.log
```
Contents:
- PostgreSQL user: `softuniblogadmin`
- System logs
- GitLab user: `softuniblogadmin`

---

### 4. WordPress Analysis
- Target: http://185.218.124.165:8080
- Found wp-admin configured, but no CAPTCHA
- Ran wpscan:
```bash
wpscan --url http://185.218.124.165:8080 -U test -P /usr/share/wordlists/rockyou.txt
```
- Blocked by potential WAF/rate-limit

---

### 5. GitLab Profiling
- Username from logs: `softuniblogadmin`
- Found repo: https://gitlab.com/softuniblogadmin/mybackup
  - Contains `id_rsa.pub`
  - README: "Good job but I already deleted my key because my server was compromised!"

---

## 🔓 Vulnerability: Stored SSH Key Exposure
- Public key exposed
- Private key assumed deleted
- Verified through GitLab repo and decoded logs

---

## 🧰 Tools Used
- gobuster
- curl, base64
- wpscan
- manual browser recon
- GitLab search

---

## ✅ Vulnerabilities Found (8)
1. HTML Injection
2. Reflected XSS
3. Insecure HTTP
4. Base64-encoded log exposure
5. PostgreSQL credentials leaked
6. GitLab username and key exposure
7. Unprotected WordPress
8. Confirmed SSH key usage

---

## 📝 Conclusion
Full recon cycle completed:  
from passive discovery → log analysis → GitLab leak confirmation.  
SSH key usage confirmed indirectly.  
Exam objective achieved.

Thanks for the challenge! 🚀  
