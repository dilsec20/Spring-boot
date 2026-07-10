# 🐧 Linux — Complete In-Depth Guide for Developers

> **"Every server, every cloud instance, every Docker container runs Linux. As a backend developer, Linux mastery is non-negotiable."**

---

## 📑 Table of Contents

1. [Why Linux for Developers?](#1-why-linux-for-developers)
2. [File System Structure](#2-file-system-structure)
3. [Essential Commands](#3-essential-commands)
4. [File Permissions](#4-file-permissions)
5. [Process Management](#5-process-management)
6. [Text Processing](#6-text-processing)
7. [Networking](#7-networking)
8. [Package Management](#8-package-management)
9. [Shell Scripting](#9-shell-scripting)
10. [SSH & Remote Access](#10-ssh--remote-access)
11. [Systemd & Services](#11-systemd--services)
12. [Cron Jobs](#12-cron-jobs)
13. [Disk & Storage](#13-disk--storage)
14. [Environment Variables](#14-environment-variables)
15. [Linux for Spring Boot](#15-linux-for-spring-boot)
16. [Interview Questions & Answers (50+)](#16-interview-questions--answers-50)

---

## 1. Why Linux for Developers?

```
Where Linux runs:
  ✅ AWS EC2, GCP, Azure VMs → Linux
  ✅ Docker containers → Linux
  ✅ Kubernetes → Linux
  ✅ CI/CD servers (Jenkins, GitHub Actions) → Linux
  ✅ 96% of web servers → Linux
  ✅ Android → Linux kernel

Every backend developer MUST know Linux commands!
```

---

## 2. File System Structure

```
/                    Root (top of everything)
├── /home            User home directories (/home/dilip)
├── /root            Root user's home
├── /etc             Configuration files (nginx.conf, hosts)
├── /var             Variable data (logs, databases)
│   ├── /var/log     System and application logs
│   └── /var/www     Web server files
├── /tmp             Temporary files (cleared on reboot)
├── /opt             Optional software (Tomcat, custom apps)
├── /usr             User programs and utilities
│   ├── /usr/bin     User binaries (most commands)
│   └── /usr/local   Locally installed software
├── /bin             Essential binaries (ls, cp, mv)
├── /sbin            System binaries (systemctl, fdisk)
├── /dev             Device files (disks, terminals)
├── /proc            Process information (virtual filesystem)
└── /mnt, /media     Mount points for drives
```

---

## 3. Essential Commands

```bash
# ═══ NAVIGATION ═══
pwd                          # Print working directory
ls                           # List files
ls -la                       # Long format + hidden files
ls -lh                       # Human-readable sizes (KB, MB)
cd /path/to/dir              # Change directory
cd ~                         # Go to home directory
cd ..                        # Go up one level
cd -                         # Go to previous directory

# ═══ FILE OPERATIONS ═══
touch file.txt               # Create empty file
mkdir mydir                  # Create directory
mkdir -p a/b/c               # Create nested directories
cp file.txt backup.txt       # Copy file
cp -r dir1 dir2              # Copy directory recursively
mv file.txt newname.txt      # Rename/move file
rm file.txt                  # Delete file
rm -rf directory/            # Delete directory (CAREFUL!)
ln -s /path/to/file link     # Create symbolic link

# ═══ VIEWING FILES ═══
cat file.txt                 # Display entire file
less file.txt                # Page through file (q to quit)
head -20 file.txt            # First 20 lines
tail -20 file.txt            # Last 20 lines
tail -f /var/log/app.log     # Follow log in real-time
wc -l file.txt               # Count lines

# ═══ SEARCHING ═══
find / -name "*.log"         # Find files by name
find /home -size +100M       # Files larger than 100MB
find . -mtime -7             # Modified in last 7 days
grep "error" file.txt        # Search text in file
grep -r "TODO" /src/         # Search recursively in directory
grep -i "error" file.txt     # Case-insensitive search
grep -n "error" file.txt     # Show line numbers
which java                   # Find command location

# ═══ COMPRESSION ═══
tar -czf archive.tar.gz dir/ # Compress directory
tar -xzf archive.tar.gz      # Extract archive
zip -r archive.zip dir/      # Create zip
unzip archive.zip             # Extract zip

# ═══ DISK USAGE ═══
df -h                        # Disk space usage
du -sh /var/log               # Directory size
du -h --max-depth=1 /        # Size of each top-level directory
```

---

## 4. File Permissions

```
ls -la output:
-rwxr-xr-- 1 dilip developers 4096 Jan 15 10:30 script.sh
│││││││││      │       │        │        │          │
│││││││││      │       │        │        │          └── Filename
│││││││││      │       │        │        └── Last modified
│││││││││      │       │        └── Size (bytes)
│││││││││      │       └── Group
│││││││││      └── Owner
│││││││││
│├─┤├─┤├─┤
│ │  │  └── Others:  r-- (read only)
│ │  └──── Group:    r-x (read + execute)
│ └─────── Owner:    rwx (read + write + execute)
└───────── Type:     - (file), d (directory), l (link)

Permission values:
  r (read)    = 4
  w (write)   = 2
  x (execute) = 1

Examples:
  chmod 755 script.sh    → rwxr-xr-x (owner: all, group+others: read+execute)
  chmod 644 config.yml   → rw-r--r-- (owner: read+write, others: read only)
  chmod 700 secret.key   → rwx------ (owner only!)
  chmod +x script.sh     → Add execute permission
  chown dilip:dev file   → Change owner and group
```

---

## 5. Process Management

```bash
ps aux                       # All running processes
ps aux | grep java           # Find Java processes
top                          # Live process monitor
htop                         # Better process monitor (install first)

kill PID                     # Graceful stop (SIGTERM)
kill -9 PID                  # Force kill (SIGKILL)
killall java                 # Kill all Java processes

# Background processes
./script.sh &                # Run in background
nohup java -jar app.jar &    # Run even after logout
jobs                         # List background jobs
fg %1                        # Bring job 1 to foreground

# Find what's using a port
lsof -i :8080                # Process using port 8080
netstat -tlnp | grep 8080    # Same (older systems)
ss -tlnp | grep 8080         # Same (newer systems)
fuser -k 8080/tcp            # Kill process on port 8080
```

---

## 6. Text Processing

```bash
# ═══ GREP (search) ═══
grep "ERROR" /var/log/app.log              # Find errors
grep -c "ERROR" /var/log/app.log           # Count errors
grep -A 3 "Exception" app.log             # Show 3 lines AFTER match
grep -B 2 "Exception" app.log             # Show 2 lines BEFORE match

# ═══ AWK (column processing) ═══
awk '{print $1, $4}' access.log           # Print columns 1 and 4
awk -F: '{print $1}' /etc/passwd          # Print usernames (: delimiter)
awk '$9 == 500' access.log                # Lines where column 9 = 500

# ═══ SED (find/replace) ═══
sed 's/old/new/g' file.txt                # Replace all occurrences
sed -i 's/8080/9090/g' config.yml         # In-place replace
sed -n '10,20p' file.txt                  # Print lines 10-20

# ═══ PIPE & REDIRECT ═══
cat file.txt | grep "error" | wc -l      # Count error lines
ls -la | sort -k5 -n                      # Sort by size (column 5)
echo "hello" > file.txt                   # Write (overwrite)
echo "world" >> file.txt                  # Append
command 2>&1 | tee output.log             # Output to screen AND file
```

---

## 7. Networking

```bash
# ═══ NETWORK INFO ═══
ip addr                      # Show IP addresses
ifconfig                     # Show IP (older systems)
hostname -I                  # Show IP address(es)
curl ifconfig.me             # Public IP

# ═══ CONNECTIVITY ═══
ping google.com              # Test connectivity
curl -v http://localhost:8080 # HTTP request with details
wget https://example.com/file # Download file
telnet host 5432             # Test if port is reachable
nc -zv host 5432             # Test port (netcat)

# ═══ DNS ═══
nslookup google.com          # DNS lookup
dig google.com               # Detailed DNS info
cat /etc/hosts               # Local DNS entries
cat /etc/resolv.conf         # DNS server config

# ═══ FIREWALL ═══
ufw allow 8080               # Allow port (Ubuntu)
ufw status                   # Show firewall rules
firewall-cmd --add-port=8080/tcp  # Allow port (CentOS)
iptables -L                  # List all rules
```

---

## 8. Package Management

```bash
# ═══ Ubuntu/Debian (apt) ═══
sudo apt update                    # Update package list
sudo apt install nginx             # Install package
sudo apt remove nginx              # Remove package
sudo apt upgrade                   # Upgrade all packages
apt search openjdk                 # Search for packages

# ═══ CentOS/RHEL (yum/dnf) ═══
sudo yum install nginx             # Install
sudo yum remove nginx              # Remove
sudo yum update                    # Update all
dnf install java-21-openjdk        # Newer CentOS (dnf)

# ═══ Install Java ═══
sudo apt install openjdk-21-jdk    # Ubuntu
sudo yum install java-21-openjdk   # CentOS
java -version                       # Verify
```

---

## 9. Shell Scripting

```bash
#!/bin/bash
# deploy.sh — Deployment script for Spring Boot

# Variables
APP_NAME="myapp"
JAR_FILE="target/${APP_NAME}.jar"
LOG_FILE="/var/log/${APP_NAME}.log"
PID_FILE="/var/run/${APP_NAME}.pid"

# Functions
start() {
    echo "Starting ${APP_NAME}..."
    
    if [ -f "$PID_FILE" ]; then
        echo "Already running (PID: $(cat $PID_FILE))"
        return 1
    fi
    
    nohup java -jar "$JAR_FILE" \
        --spring.profiles.active=prod \
        -Xmx512m -Xms256m \
        > "$LOG_FILE" 2>&1 &
    
    echo $! > "$PID_FILE"
    echo "Started with PID: $(cat $PID_FILE)"
}

stop() {
    if [ -f "$PID_FILE" ]; then
        kill $(cat "$PID_FILE")
        rm "$PID_FILE"
        echo "Stopped"
    else
        echo "Not running"
    fi
}

status() {
    if [ -f "$PID_FILE" ] && kill -0 $(cat "$PID_FILE") 2>/dev/null; then
        echo "Running (PID: $(cat $PID_FILE))"
    else
        echo "Not running"
    fi
}

# Script entry point
case "$1" in
    start)  start ;;
    stop)   stop ;;
    restart) stop && sleep 2 && start ;;
    status) status ;;
    *)      echo "Usage: $0 {start|stop|restart|status}" ;;
esac
```

---

## 10. SSH & Remote Access

```bash
# ═══ Connect ═══
ssh user@server-ip                    # Password auth
ssh -i mykey.pem ec2-user@server-ip   # Key auth (AWS)
ssh -p 2222 user@server               # Custom port

# ═══ SSH Keys ═══
ssh-keygen -t ed25519                 # Generate key pair
ssh-copy-id user@server               # Copy public key to server

# ═══ File Transfer ═══
scp file.txt user@server:/path/       # Copy TO server
scp user@server:/path/file.txt .      # Copy FROM server
scp -r directory/ user@server:/path/  # Copy directory

# ═══ SSH Tunnel (Port Forwarding) ═══
ssh -L 3307:localhost:3306 user@server
# Access remote MySQL (port 3306) via local port 3307
```

---

## 11. Systemd & Services

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Spring Boot App
After=network.target mysql.service

[Service]
User=spring
Group=spring
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar --spring.profiles.active=prod
SuccessExitStatus=143
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl start myapp       # Start service
sudo systemctl stop myapp        # Stop service
sudo systemctl restart myapp     # Restart
sudo systemctl status myapp      # Check status
sudo systemctl enable myapp      # Start on boot
sudo systemctl disable myapp     # Don't start on boot
journalctl -u myapp -f           # View service logs
```

---

## 12. Cron Jobs

```bash
# Edit crontab
crontab -e

# Format: minute hour day month weekday command
# ┌─── minute (0-59)
# │ ┌─── hour (0-23)
# │ │ ┌─── day of month (1-31)
# │ │ │ ┌─── month (1-12)
# │ │ │ │ ┌─── day of week (0-7, 0 and 7 = Sunday)
# │ │ │ │ │
# * * * * * command

# Examples:
0 2 * * * /opt/backup.sh              # Daily at 2:00 AM
*/5 * * * * /opt/health-check.sh       # Every 5 minutes
0 0 * * 0 /opt/weekly-report.sh        # Every Sunday midnight
0 9 1 * * /opt/monthly-cleanup.sh      # 1st of every month at 9 AM

crontab -l                              # List cron jobs
```

---

## 13. Disk & Storage

```bash
df -h                                  # Disk space
du -sh /var/log                        # Directory size
lsblk                                 # List block devices
mount /dev/sdb1 /mnt/data             # Mount disk
umount /mnt/data                      # Unmount
```

---

## 14. Environment Variables

```bash
# Set temporarily
export JAVA_HOME=/usr/lib/jvm/java-21
export PATH=$JAVA_HOME/bin:$PATH

# Set permanently
echo 'export JAVA_HOME=/usr/lib/jvm/java-21' >> ~/.bashrc
source ~/.bashrc

# View
echo $JAVA_HOME
printenv                               # All env variables
env | grep JAVA                        # Filter
```

---

## 15. Linux for Spring Boot

```bash
# Deploy Spring Boot to Linux server

# 1. Install Java
sudo apt install openjdk-21-jre

# 2. Create app user (don't run as root!)
sudo useradd -m -s /bin/bash spring
sudo mkdir /opt/myapp
sudo chown spring:spring /opt/myapp

# 3. Copy JAR
scp target/myapp.jar spring@server:/opt/myapp/

# 4. Create systemd service (see section 11)
# 5. Start and enable
sudo systemctl start myapp
sudo systemctl enable myapp

# 6. View logs
journalctl -u myapp -f
tail -f /opt/myapp/logs/application.log
```

---

## 16. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Linux?** Open-source operating system kernel. Used on servers, cloud, containers, Android.

**Q2: What is the root user?** Superuser with unlimited access. UID 0. Use `sudo` instead of logging in as root.

**Q3: `chmod 755` means?** Owner: rwx (7), Group: r-x (5), Others: r-x (5).

**Q4: `ls -la` shows what?** All files (including hidden), permissions, owner, size, modification date.

**Q5: How to find a file?** `find / -name "filename"` or `locate filename`.

**Q6: What is `grep`?** Search text patterns in files. `grep "error" log.txt`.

**Q7: What is `|` (pipe)?** Sends output of one command as input to another. `cat file | grep error | wc -l`.

**Q8: How to check disk space?** `df -h` for overall, `du -sh /path` for specific directory.

---

### Intermediate

**Q9: What is `nohup`?** No Hang Up. Process continues after terminal closes. `nohup command &`.

**Q10: How to find what's using port 8080?** `lsof -i :8080` or `ss -tlnp | grep 8080`.

**Q11: What is systemd?** Init system managing services. `systemctl start/stop/enable`.

**Q12: What is a cron job?** Scheduled task. `crontab -e` to edit. `0 2 * * * /script.sh` = 2 AM daily.

**Q13: Hard link vs soft link?** Hard: same inode, file persists if original deleted. Soft (symbolic): pointer to path, breaks if original deleted.

**Q14: What is `/proc`?** Virtual filesystem with process/system info. `/proc/cpuinfo`, `/proc/meminfo`.

**Q15: What is `awk`?** Text processing tool. Column-based operations. `awk '{print $1}' file`.

---

### Rapid-Fire (Q16–Q50)

**Q16: `>` vs `>>`?** `>` overwrites. `>>` appends.

**Q17: `2>&1` means?** Redirect stderr to stdout.

**Q18: What is `/dev/null`?** Discards output. `command > /dev/null 2>&1`.

**Q19: `ps aux`?** Show all running processes.

**Q20: `kill -9`?** Force kill (SIGKILL). Use as last resort.

**Q21: `top` command?** Live process monitor. CPU, memory usage.

**Q22: `tail -f`?** Follow file in real-time. Essential for log monitoring.

**Q23: `wget` vs `curl`?** wget: download files. curl: flexible HTTP client.

**Q24: `scp` command?** Secure copy over SSH.

**Q25: `tar -czf`?** Create compressed archive (.tar.gz).

**Q26: `tar -xzf`?** Extract compressed archive.

**Q27: What is `sudo`?** Execute command as superuser.

**Q28: What is `.bashrc`?** Bash config file. Loaded for interactive shells.

**Q29: What is `source`?** Execute script in current shell. `source ~/.bashrc`.

**Q30: `which java`?** Shows path of java command.

**Q31: What is `$PATH`?** Directories searched for commands.

**Q32: `cat /etc/os-release`?** Shows OS version info.

**Q33: `free -h`?** Shows memory usage (RAM).

**Q34: `uptime`?** How long system has been running.

**Q35: `whoami`?** Current username.

**Q36: `uname -a`?** System information (kernel, architecture).

**Q37: `history`?** List previous commands.

**Q38: `man command`?** Manual page for command.

**Q39: `alias`?** Create command shortcut. `alias ll='ls -la'`.

**Q40: `xargs`?** Build command from stdin. `find . -name "*.log" | xargs rm`.

**Q41: `tee`?** Output to screen AND file. `command | tee output.log`.

**Q42: `diff file1 file2`?** Show differences between files.

**Q43: What is swap?** Virtual memory on disk when RAM is full.

**Q44: What is inode?** Data structure storing file metadata (permissions, size, pointers).

**Q45: What is `/etc/hosts`?** Local DNS mapping. `127.0.0.1 localhost`.

**Q46: `chown` command?** Change file owner. `chown user:group file`.

**Q47: What is `stdin`, `stdout`, `stderr`?** Standard input (0), output (1), error (2).

**Q48: What is a daemon?** Background process (service). Like systemd services.

**Q49: `journalctl -u myapp -f`?** Follow logs of a systemd service.

**Q50: Linux distros for servers?** Ubuntu Server, CentOS/RHEL, Amazon Linux, Debian, Alpine.

---

## 📚 References

- [Linux Command Reference](https://linuxcommand.org/)
- [The Linux Documentation Project](https://tldp.org/)
- [Linux Journey](https://linuxjourney.com/)
- [Bash Scripting Guide](https://tldp.org/LDP/abs/html/)

---

> **Previous Topic:** [← 27 - Kafka](../27-springboot-kafka/README.md)  
> **Next Topic:** [29 - Ansible →](../29-ansible/README.md)
