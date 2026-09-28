# 🐧 Linux — Essential Commands

> A practical reference of Linux commands for system administration, networking, troubleshooting and IT support.

---

## 📋 Table of Contents

- [System Information](#-system-information)
- [Users, Groups & Permissions](#-users-groups--permissions)
- [Processes & Services](#-processes--services)
- [Files & Disk](#-files--disk)
- [Network](#-network)
- [SSH & Encryption](#-ssh--encryption)
- [Firewall](#-firewall)
- [Logs & Monitoring](#-logs--monitoring)
- [Package Management](#-package-management)
- [Scheduled Tasks & Scripts](#-scheduled-tasks--scripts)
- [Quick Troubleshooting](#-quick-troubleshooting)
- [Recommended Tools](#-recommended-tools)

---

## 🖥️ System Information

```bash
uname -a                      # kernel and architecture
cat /etc/os-release           # distro version
hostnamectl                   # hostname and OS summary
uptime                        # time running + load average
lscpu                         # CPU details
free -h                       # memory usage
lsblk                         # disks and partitions
lspci                         # PCI devices
lsusb                         # USB devices

# Who is logged in
who
w
last                          # recent login sessions
lastb                         # failed login attempts (root)
```

---

## 👤 Users, Groups & Permissions

```bash
whoami
id
groups
cat /etc/passwd               # list all users
cat /etc/group                # list all groups

useradd -m username           # create user with home directory
passwd username               # set/change password
usermod -aG sudo username     # add user to a group
passwd -l username            # lock account
passwd -u username            # unlock account
userdel -r username           # delete user and home directory

sudo -l                       # what the current user can run with sudo
visudo                        # safely edit sudoers

# Permissions
ls -la /path/to/file
chmod 600 private_key.pem     # owner read/write only
chmod 644 file.txt            # owner rw, others read
chmod 700 /some/directory     # owner full access only
chmod +x script.sh            # make executable
chown user:group file.txt

# Find files by permission / owner
awk -F: '$3 == 0' /etc/passwd                 # accounts with UID 0
find / -perm -4000 -type f 2>/dev/null        # SUID files
find / -perm -o+w -type f 2>/dev/null         # world-writable files
find / -nouser -type f 2>/dev/null            # files with no owner
```

---

## ⚙️ Processes & Services

```bash
ps aux
ps aux | grep nginx
pgrep -a sshd
top
htop

kill <PID>
kill -9 <PID>                 # force kill
pkill processname

# Services (systemd)
systemctl status sshd
systemctl start|stop|restart|reload nginx
systemctl enable nginx        # start at boot
systemctl disable nginx
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed
systemctl list-unit-files --state=enabled
```

---

## 📁 Files & Disk

```bash
ls -lah
cp -r source dest
mv old new
mkdir -p a/b/c
rm -r folder
stat file.txt                 # file metadata
diff file1.txt file2.txt

# Search
find / -name "*.conf" 2>/dev/null
find / -mtime -1 -type f 2>/dev/null      # modified in last 24h
find / -mmin -10 -type f 2>/dev/null      # modified in last 10 min
find / -size +100M -type f 2>/dev/null    # large files
grep -rn "text" /path/                    # search inside files

# Integrity (checksums)
sha256sum file.txt
md5sum file.txt
echo "expectedhash  file.txt" | sha256sum --check

# Disk usage
df -h                         # free space per filesystem
du -sh /*                     # size per directory
du -ah / | sort -rh | head -20
lsof -p <PID>                 # files opened by a process
```

---

## 🌐 Network

```bash
ip a                          # IP addresses
ip route                      # routing table
ping -c 4 8.8.8.8
traceroute google.com
nslookup domain.com
dig domain.com
arp -a                        # IP to MAC table
curl -I https://example.com   # HTTP headers
nmcli device status           # NetworkManager status

# Ports and connections
ss -tulnp                     # listening ports + processes
ss -anp                       # all connections
lsof -i :22                   # who is using port 22

# Scanning and capture
nmap -sV 192.168.1.1          # open ports + service versions
nmap -sn 192.168.1.0/24       # discover hosts on the network
tcpdump -i eth0 port 443
tcpdump -i eth0 -w capture.pcap
```

---

## 🔐 SSH & Encryption

```bash
ssh user@host
ssh -p 2222 user@host
ssh -i ~/.ssh/my_key user@host

ssh-keygen -t ed25519 -C "your@email.com"     # generate key pair
ssh-copy-id user@host
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/my_key
ssh-add -l

scp file.txt user@host:/remote/path/
scp user@host:/remote/file.txt ./local/
rsync -avz folder/ user@host:/backup/

# Server config
cat /etc/ssh/sshd_config
sshd -t                       # validate config
systemctl restart sshd
# Recommended settings in sshd_config:
#   PermitRootLogin no
#   PasswordAuthentication no
#   PubkeyAuthentication yes
#   MaxAuthTries 3

# GPG
gpg --full-generate-key
gpg -c file.txt               # encrypt with a passphrase
gpg file.txt.gpg              # decrypt
```

---

## 🛡️ Firewall

```bash
# UFW (Ubuntu/Debian/Mint)
ufw status verbose
ufw allow ssh                 # do this BEFORE enabling
ufw enable
ufw allow 443/tcp
ufw deny 23/tcp
ufw allow from 192.168.1.100
ufw delete allow 80/tcp

# iptables
iptables -L -v -n
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -s 192.168.1.100 -j DROP
iptables-save > /etc/iptables/rules.v4
```

---

## 📊 Logs & Monitoring

```bash
journalctl -f                          # follow logs live
journalctl -xe                         # recent errors
journalctl -u nginx -n 100
journalctl --since "1 hour ago"
journalctl -b                          # current boot
dmesg | tail -50                       # kernel messages

tail -f /var/log/syslog
cat /var/log/auth.log
grep "Failed password" /var/log/auth.log
grep "Accepted" /var/log/auth.log

# Count failed logins by IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Live resource usage
top
htop
iostat
vmstat 1
free -h
df -h
```

**Zabbix Agent**

```bash
systemctl status zabbix-agent2
tail -f /var/log/zabbix/zabbix_agent2.log
zabbix_agent2 -t agent.ping              # test an item locally
zabbix_get -s <host-ip> -k system.cpu.load   # test from the server
```

---

## 📦 Package Management

```bash
# Debian / Ubuntu / Mint
apt update
apt upgrade -y
apt list --upgradable
apt search packagename
apt install packagename
apt remove --purge packagename
dpkg -l | grep nginx          # is it installed?

# Disable services you don't need
systemctl disable bluetooth
systemctl disable cups
```

---

## ⏰ Scheduled Tasks & Scripts

```bash
crontab -l                    # list your jobs
crontab -e                    # edit
cat /etc/crontab
ls -la /etc/cron.*
# minute hour day month weekday command
# 0 2 * * * /home/user/backup.sh
```

```bash
#!/bin/bash
# Restart a service if it is down
SERVICE="nginx"
if ! systemctl is-active --quiet "$SERVICE"; then
    echo "$(date): $SERVICE down, restarting" >> /var/log/check.log
    systemctl restart "$SERVICE"
fi
```

---

## 🔧 Quick Troubleshooting

| Problem | Where to look |
|---|---|
| No internet | `ip a`, `ip route`, `ping 8.8.8.8`, `nslookup google.com` |
| Service won't start | `systemctl status svc`, `journalctl -u svc -n 50` |
| Slow system | `top`/`htop`, `free -h`, `iostat`, `df -h` |
| Disk full | `df -h`, `du -sh /*`, `find / -size +100M` |
| Can't connect to a port | `ss -tulnp`, `ufw status`, `nmap -p PORT host` |
| Login problems | `last`, `lastb`, `/var/log/auth.log` |
| Boot / driver issues | `journalctl -b -p err`, `dmesg \| tail` |
| Unknown process | `ps aux`, `lsof -p PID`, `ss -anp` |

---

## 🛠️ Recommended Tools

| Tool | Purpose |
|---|---|
| [htop](https://htop.dev/) | Interactive process viewer |
| [Zabbix](https://www.zabbix.com/) | Infrastructure monitoring and alerting |
| [Grafana](https://grafana.com/) | Dashboards and metrics visualization |
| [Wireshark](https://www.wireshark.org/) | Network packet analysis |
| [nmap](https://nmap.org/) | Network discovery and port scanning |
| [Fail2Ban](https://www.fail2ban.org/) | Block IPs after repeated failed logins |
| [Lynis](https://cisofy.com/lynis/) | System auditing and best-practice checks |

---

<div align="center">

*Maintained as a personal reference for IT support and system administration.*  
*Commands tested on Debian/Ubuntu-based distributions (including Linux Mint).*

</div>
