![image](https://github.com/user-attachments/assets/f611f030-65de-4242-bf9a-6a1496ef4764)
![image](https://github.com/user-attachments/assets/e369d0a0-c01b-448d-9497-9b79fcdbaa13)
![image](https://github.com/user-attachments/assets/a0a55e82-b6f8-4f6f-97cd-68c3a10b222f)
![image](https://github.com/user-attachments/assets/5c0b4014-da3e-4681-9e5e-f7bc1ffc7081)
![image](https://github.com/user-attachments/assets/541e68fa-444b-411c-b0a7-581e4ac3d1af)

![image](https://github.com/user-attachments/assets/7207c50a-a34e-439c-b755-4a0ba716864d)


![image](https://github.com/user-attachments/assets/e216de62-6c8a-4a2b-affd-e3e74ce3a415)

![image](https://github.com/user-attachments/assets/725c50c9-5d79-4a7a-9a9b-d9308a415b9d)

![image](https://github.com/user-attachments/assets/c943ca35-bd98-4a2e-8ab2-9886da17bfc0)

![image](https://github.com/user-attachments/assets/1c9e1c9c-db94-4bbc-b939-b7501348ede4)

![image](https://github.com/user-attachments/assets/775d9dd8-3d20-4e9d-867e-ae9d0549fb67)

![image](https://github.com/user-attachments/assets/13122e5e-df64-4731-b33b-065efe1e384b)

![image](https://github.com/user-attachments/assets/93fbd519-42bb-4b7a-9386-1cb64479a9f1)

![image](https://github.com/user-attachments/assets/3732c3e4-609c-42c7-87da-3b4d171cb15f)

![image](https://github.com/user-attachments/assets/d1d5d44f-efb8-4fc2-a345-812b9a640fbb)

![image](https://github.com/user-attachments/assets/d318df7f-efd6-4120-ad17-76009d1cd5d7)

![image](https://github.com/user-attachments/assets/cd2899da-1c33-49fb-88a3-20873ddb391e)

![image](https://github.com/user-attachments/assets/f3503826-4548-4f0f-b8ca-f3517872c6e4)

![image](https://github.com/user-attachments/assets/3ad9a652-2f83-46c2-bd2f-c4d35d84c715)

![image](https://github.com/user-attachments/assets/0f4b3ba6-1246-4d04-b5e8-42e2187eadad)
![image](https://github.com/user-attachments/assets/3a2becb5-3c83-490f-be3b-b6dc765dca33)
![image](https://github.com/user-attachments/assets/e08192b9-6141-4807-b751-3e022ddb95c6)



![image](https://github.com/user-attachments/assets/2049aa3f-0954-40d3-8e48-586b86686873)



![image](https://github.com/user-attachments/assets/1e5627e2-ff9e-4292-9848-79dec83bf57b)

![image](https://github.com/user-attachments/assets/e7246445-508a-4c89-8857-13fece9ab073)


✅ Step-by-Step Summary of My Reconnaissance Process
🔍 Step 1: Initial Scanning and Passive Discovery
I started with passive fingerprinting of the target:
    • Accessed http://185.218.124.165/ manually and located the main web application.
    • Identified the search_query= parameter vulnerable to reflection, which led to HTML Injection and Reflected XSS.

🧭 Step 2: Directory Enumeration with Gobuster
Used gobuster to brute-force directories and file types with common extensions:
bash
CopyEdit
gobuster dir -u http://185.218.124.165 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,log,bak,zip -t 30
Discovered:
    • /index.php – main application
    • /z – Base64-encoded log file

🧾 Step 3: Base64 Log File Analysis
Downloaded and decoded the file found at /z:
bash
CopyEdit
curl -s http://185.218.124.165/z -o encoded.txt
cat encoded.txt | base64 -d > decoded.log
Inside the log:
    • PostgreSQL credentials: softuniblogadmin
    • GitLab activity and system information (CPU, RAM, backups)
    • Logins and environment metadata
✅ This gave me the lead to investigate GitLab.

🛠️ Step 4: WordPress CMS Vulnerabilities
The web application on port 8080 runs WordPress:
    • Initially accessible at /wp-admin/install.php, suggesting it was uninitialized (Critical Risk).
    • Later configured, but still vulnerable to brute-force login via /wp-login.php.
    • No CAPTCHA, IP lockout, or rate limiting.
Attempted brute-force with:
bash
CopyEdit
wpscan --url http://185.218.124.165:8080 -U hgdsfsdasd -P /usr/share/wordlists/rockyou.txt
Eventually blocked with:
pgsql
CopyEdit
Error: No response from remote server. WAF/IPS?

🧑‍💻 Step 5: GitLab Profiling and Repository Analysis
Used the leaked username softuniblogadmin from decoded logs and located the GitLab profile:
🔗 https://gitlab.com/softuniblogadmin/mybackup
Found in the repo:
    • id_rsa.pub (public SSH key)
    • README.md stating:
“Good job but I already deleted my key because my server was compromised!”

🧨 Step 6: The Breakthrough — Private Key Recovered
By reviewing older commits, I found that in commit 8970beca a file named id_rsa was deleted.
This file contained a full SSH private key, proving the system was at one point directly accessible via SSH.
plaintext
CopyEdit
-----BEGIN OPENSSH PRIVATE KEY-----
... [full key body]
-----END OPENSSH PRIVATE KEY-----
🎯 This directly satisfies the primary exam objective — identification of a stored and accessible SSH private key.

🧰 Tools Used:
    • gobuster
    • nmap
    • curl, base64
    • wpscan
    • Browser, GitLab interface

✅ Findings Summary:
    • 10 vulnerabilities total:
        ◦ HTML Injection
        ◦ Reflected XSS
        ◦ WordPress brute-force
        ◦ WordPress install.php exposed
        ◦ No HTTPS (Insecure transport)
        ◦ Base64 log disclosure
        ◦ GitLab account linked via leaked username
        ◦ Public key stored in Git
        ◦ SSH port open (22/tcp)
        ◦ 🔐 Full private SSH key recovered from Git commit history ✅

🏁 Conclusion
This exam demonstrated a full reconnaissance process using both passive and active techniques, combining:
    • Open-source intelligence (OSINT),
    • Directory and service enumeration,
    • Git repository forensics,
    • And log file analysis.
The engagement concluded with direct discovery of an SSH private key, completing the exam objective.
		Thank you, Luchezar – this was a very interesting and rewarding challenge!
