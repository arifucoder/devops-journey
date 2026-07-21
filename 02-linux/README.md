# 🐧 Linux Command Cheatsheet

Linux শেখার ধারাবাহিক নোট — প্রতিটা class আলাদা section-এ সাজানো।

## 📚 Table of Contents

- [Class 1 — Basics: Navigation, File & Vim](#class-1--basics-navigation-file--vim)
- [Class 2 — Directory Structure & User Management](#class-2--directory-structure--user-management)
- [Class 3 — Groups, Permissions, SCP & Services](#class-3--groups-permissions-scp--services)
- [Class 4 — Troubleshooting & Shell Scripting](#class-4--troubleshooting--shell-scripting)

---

# Class 1 — Basics: Navigation, File & Vim

## Navigation

| Command | Description |
|---------|-------------|
| `pwd` | বর্তমান directory-র সম্পূর্ণ path দেখায় <br>`pwd` = **print working directory** |
| `whoami` | বর্তমানে logged-in user-এর নাম দেখায় |
| `cd ..` | এক level উপরের (parent) directory-তে যায় |
| `cd /` | Root directory-তে যায় |
| `man mkdir` | কোনো command-এর manual/documentation দেখায় <br>`man` = **manual** |

## File ও Directory দেখা

| Command | Description |
|---------|-------------|
| `ls` | বর্তমান directory-র file ও folder-এর তালিকা দেখায় <br>`ls` = **list** |
| `ls -l` | **Long format**-এ তালিকা দেখায় (permission, owner, size, date সহ) |
| `ls -a` | সব file দেখায়, hidden file (`.` দিয়ে শুরু) সহ <br>`-a` = **all** |
| `ls -la` | Long format-এ সব file দেখায়, hidden file সহ |

## File Content দেখা ও লেখা

| Command | Description |
|---------|-------------|
| `cat index.html` | File-এর পুরো content terminal-এ দেখায় <br>`cat` = **concatenate** |
| `head index.html` | File-এর প্রথম ১০ লাইন দেখায় |
| `head -n 2 index.html` | File-এর প্রথম ২ লাইন দেখায় <br>`-n` = **number of lines** |
| `tail -n 2 index.html` | File-এর শেষ ২ লাইন দেখায় |
| `echo "Hello my friend" >> greeting.txt` | File-এর শেষে নতুন লাইন যোগ করে (**append**) — আগের content থাকে |
| `echo "Hello replaced" > greeting.txt` | File-এর পুরো content মুছে নতুন content লেখে (**overwrite**) |
| `mv infex.html index.html` | File rename করে — `infex.html` থেকে `index.html` <br>`mv` = **move** |
| `mkdir folder_name` | নতুন directory/folder তৈরি করে <br>`mkdir` = **make directory** |

## Vim Editor

| Command | Description |
|---------|-------------|
| `vim linux.txt` | Vim editor-এ file খোলে (না থাকলে নতুন তৈরি হয়) |
| `i` | **Insert mode** চালু করে — এখন লেখা যাবে |
| `Esc` | Insert mode থেকে বের হয়ে **normal mode**-এ ফিরে যায় |
| `:wq` | File save করে quit করে <br>`w` = **write**, `q` = **quit** |
| `:q!` | Save না করেই **force quit** করে |

> **Vim Workflow (সংক্ষেপে):**
> 1. `vim linux.txt` — file খুলুন
> 2. `i` চাপুন — insert mode (এখন লিখুন)
> 3. `Esc` চাপুন — লেখা শেষ হলে
> 4. `:wq` — save করে বের হন **অথবা** `:q!` — save না করে বের হন

---

# Class 2 — Directory Structure & User Management

## Linux Directory Structure

| Directory | Description |
|-----------|-------------|
| `/` | Root directory — এখান থেকেই Linux শুরু হয়, পুরো system-এর সবচেয়ে উপরের level। একে computer-এর brain বলা যায় |
| `/etc` | System-wide configuration file-এর directory — DevOps engineer-দের জন্য most important directory। Docker বা অন্য কিছু install করলে তাদের config file এখানে থাকে |
| `/var` | **Variable** data থাকে — log file গুলো `/var/log` directory-তে থাকে |
| `/usr/bin` | Custom script গুলো এখানে রাখা হয় |
| `/tmp` | **Temporary** file/folder রাখার জায়গা — **important file কখনোই `/tmp`-এ রাখা যাবে না** (system reboot হলে মুছে যেতে পারে) |
| `/dev` | **Device** file-এর directory |

> **Standard মনে রাখার নিয়ম:**
> Config → `/etc` | Log → `/var/log` | Script → `/usr/bin`

## Commands

| Command | Description |
|---------|-------------|
| `cd ~` | সব সময় home directory-তে নিয়ে আসে (যেমন `/home/ubuntu`) |
| `sudo su` | Root (**super user**) হিসেবে switch করে <br>`sudo` = **super user do**, `su` = **switch user** |
| `exit` | Root user থেকে বের হয়ে আসে |
| `tree` | Folder/file-এর visual (tree) representation দেখায় — না থাকলে `sudo apt install tree` দিয়ে install করতে হবে |
| `ping google.com` | Internet/network connection ঠিক আছে কিনা এবং কোনো server (host) reachable কিনা check করে |
| `mkdir folder1 folder2` | এক command-এ একাধিক folder create করে |
| `rm filename` | File remove/delete করে <br>`rm` = **remove** |
| `rm -rf foldername` | Folder (ভিতরের সব content সহ) delete করে <br>`-r` = **recursive**, `-f` = **force** |
| `clear` অথবা `Ctrl + L` | Terminal screen clear করে (Mac terminal-এ `Cmd + K`-ও কাজ করে) |

## Bulk Operation (Brace Expansion)

| Command | Description |
|---------|-------------|
| `mkdir -p DAY{1..120}` | এক command-এ 120টা folder (DAY1 থেকে DAY120) তৈরি করে <br>`-p` = **parents** (parent folder না থাকলেও বানিয়ে নেয়, থাকলে error দেয় না) |
| `touch DAY{1..120}/README.md` | প্রত্যেকটা DAY folder-এর ভিতরে একটা `README.md` file তৈরি করে |
| `mkdir -p DAY{1..120} && touch DAY{1..120}/README.md` | উপরের দুইটা কাজ এক command-এ |
| `rm -rf DAY{1..120}` | সবগুলো DAY folder এক command-এ delete করে |

## User Management

| Command | Description |
|---------|-------------|
| `sudo useradd -m uddin` | নতুন user create করে <br>`useradd` = **user add**, `-m` = **make home directory** |
| `sudo passwd uddin` | ওই user-এর জন্য password set করে — ২ বার password type করতে হবে |
| `su uddin` | `uddin` user হিসেবে switch করে কাজ করা যায় — password দিতে হবে |
| `cat /etc/passwd` | System-এর সব user-এর list দেখায় |

---

# Class 3 — Groups, Permissions, SCP & Services

## Groups কী এবং কেন?

Group হচ্ছে Linux-এ user-দের **organize** করা এবং **permission control** করার একটা উপায়। এক এক করে প্রত্যেক user-কে permission দেওয়ার বদলে আমরা পুরো group-কে permission দিয়ে দিই।

> **কেন group?** ধরা যাক আমাদের `operations`, `tester`, `developer` নামে group আছে। Team-এ নতুন একজন developer join করলে তাকে শুধু `developer` group-এ add করে দিলেই হয়ে যায় — আলাদা করে permission দিতে হয় না। একইভাবে operations team-এ join করলে `operations` group-এ add করব।

> **মনে রাখা:** যখন কোনো user create করি, সাথে সাথে একই নামে একটা group-ও create হয়, আর প্রত্যেক user-এর জন্য home directory-ও তৈরি হয়ে যায়।

## Group Management

| Command | Description |
|---------|-------------|
| `sudo groupadd devops` | নতুন group create করে <br>`groupadd` = **group add** |
| `cat /etc/group` | System-এর সব group-এর list দেখায় |
| `grep "devops_juniors" /etc/group` | নির্দিষ্ট group search করে দেখার জন্য <br>`grep` = **Global Regular Expression Print** |
| `sudo groupdel docker` | Group delete করে <br>`groupdel` = **group delete** |
| `sudo usermod -aG devops_junior uddin` | User-কে group-এ add করে <br>`usermod` = **user modify**, `-a` = **append**, `-G` = **Groups** (append না দিলে user আগের group থেকে বাদ পড়ে যাবে!) |
| `sudo gpasswd -M suresh,jogesh,ganesh devops_junior` | অনেক user-কে একসাথে group-এ add করে <br>`gpasswd` = **group password (group administration tool)**, `-M` = **Members** |
| `sudo su - ramesh` | অন্য user-এ switch করে <br>`-` মানে ওই user-এর full environment (home directory) সহ login |

## Docker Group Example

ধরা যাক Docker install করলাম। এখন `ubuntu` user-কে Docker-এর access দিতে চাইলে:

```bash
sudo usermod -aG docker $USER   # current user-কে docker group-এ append করে
newgrp docker                   # newgrp = new group — logout না করেই group membership activate করে
docker ps                       # এখন sudo ছাড়াই কাজ করবে
```

## File Permissions

`drwxrwxrwx` — এভাবে ভাগ করে পড়তে হয়:

| অংশ | মানে |
|------|------|
| `d` | **Directory** (file হলে `-` থাকে) |
| প্রথম `rwx` | **Owner**-এর permission |
| দ্বিতীয় `rwx` | **Group**-এর permission |
| তৃতীয় `rwx` | **Others**-এর permission |

> `r` = **read**, `w` = **write**, `x` = **execute**

| Command | Description |
|---------|-------------|
| `sudo chown :devops_juniors production.secret` | File-এর group ownership change করে (`:` এর আগে কিছু না দিলে শুধু group বদলায়, owner না) <br>`chown` = **change owner** |
| `sudo chmod 640 production.secret` | File-এর permission change করে <br>`chmod` = **change mode** <br>`640` মানে: owner = `rw-` (6), group = `r--` (4), others = `---` (0) |

> **Permission number:** `r` = 4, `w` = 2, `x` = 1 — যোগ করে number বানানো হয় (যেমন `rw-` = 4+2 = 6)

## ACL (Access Control List)

আরো **fine-grained permission**-এর জন্য ACL best — নির্দিষ্ট user বা group-কে আলাদা করে permission দেওয়া যায়।

| Command | Description |
|---------|-------------|
| `setfacl -m g:groupname:permission filename` | Group-কে নির্দিষ্ট file-এ permission দেয় <br>`setfacl` = **set file ACL**, `-m` = **modify**, `g:` = **group** <br>উদাহরণ: `setfacl -m g:devops_junior:r production.secret` |

## SCP — Local ↔ Server File Transfer

`scp` = **Secure Copy** (SSH-এর উপর দিয়ে file copy করে)

| Command | Description |
|---------|-------------|
| `scp -i pemkey filename ubuntu@serveraddress:/home/ubuntu` | Local থেকে server-এ file **upload** করে <br>`-i` = **identity file** (pem key) |
| `scp -i pemkey ubuntu@serveraddress:/home/ubuntu/aws.txt .` | Server থেকে local-এ file **download** করে (শেষের `.` মানে current directory) |

## Nginx & systemctl

`systemctl` = **system control** — যেকোনো service manage করার জন্য এটা লাগবে।

| Command | Description |
|---------|-------------|
| `sudo apt install nginx -y` | Nginx install করে <br>`-y` = **yes** (সব prompt-এ automatically yes) |
| `nginx -v` | Nginx-এর version check করে <br>`-v` = **version** |
| `sudo systemctl status nginx` | Nginx running আছে কিনা check করে |
| `sudo systemctl status ssh` | SSH service-এর status দেখে |
| `sudo systemctl stop nginx` | Nginx stop করে |
| `sudo systemctl restart nginx` | Nginx restart করে |
| `systemctl list-units` | কোন কোন unit চলছে দেখায় |

## journalctl — Logs দেখা

`journalctl` = **journal control**

> **systemctl vs journalctl:** `systemctl` দিয়ে service start/stop/restart করা হয়, আর `journalctl` **record (log) রাখে** — তাই log দেখার জন্য `journalctl` ব্যবহার করতে হয়।

| Command | Description |
|---------|-------------|
| `sudo journalctl -fu nginx.service` | Nginx-এর live log দেখায় <br>`-f` = **follow** (live update), `-u` = **unit** (নির্দিষ্ট service) |

---

# Class 4 — Troubleshooting & Shell Scripting

## Server Health Check (Disk, Memory)

| Command | Description |
|---------|-------------|
| `df -h` | VPS/server-এর disk-এর কী অবস্থা তা দেখায় <br>`df` = **disk free**, `-h` = **human-readable** (KB/MB/GB আকারে) |
| `cd /var/log` | কোনো app বা website-এ problem হলে প্রথমেই log directory-তে যেতে হয় |
| `du -sh *` | Current directory-র প্রত্যেকটা file/folder কত জায়গা নিচ্ছে দেখায় <br>`du` = **disk usage**, `-s` = **summarize**, `*` = সব file/folder |
| `free -h` | RAM/memory কতটুকু free আছে দেখায় |

## Server Troubleshooting — grep / awk / sed / find

| Tool | কাজ | Full form |
|------|-----|-----------|
| `grep` | Text-এর ভিতরে pattern **search** করা | **Global Regular Expression Print** |
| `awk` | Text processing ও **field manipulation** — log থেকে নির্দিষ্ট column/line বের করা | এর creator-দের নামের আদ্যক্ষর — **A**ho, **W**einberger, **K**ernighan |
| `sed` | Text **edit/replace** করা (line by line stream আকারে) | **Stream EDitor** |
| `find` | File/folder **search** করা | — |

### grep

| Command | Description |
|---------|-------------|
| `grep "error" app.log` | `app.log` file-এ যেসব line-এ `error` আছে সেগুলো দেখায় (case-sensitive) |
| `grep -i "error" app.log` | Case-**insensitive** search — `error`, `ERROR`, `Error` সবই ধরবে <br>`-i` = **ignore case** |

### awk

| Command | Description |
|---------|-------------|
| `awk '/ERROR/ {print $3, $4}' app.log` | যেসব line-এ `ERROR` আছে, সেই line-গুলোর **৩য় ও ৪র্থ field (column)** print করে <br>`$1`, `$2`, `$3`... = column number, `$0` = পুরো line |

### find

| Command | Description |
|---------|-------------|
| `find . -name "*.log"` | Current directory (`.`) ও তার সব sub-directory-তে `.log` extension-এর file খোঁজে <br>`-name` = নাম দিয়ে খোঁজা |
| `find . -mtime 0` | গত **২৪ ঘণ্টার** ভিতরে কোন কোন file change হয়েছে দেখায় <br>`-mtime` = **modification time** (দিনের হিসাবে), `0` = শেষ ১ দিন |

### sed

| Command | Description |
|---------|-------------|
| `sed 's/Hello/obhai/' nure.log` | **On the fly** — screen-এ শুধু replace করা output print করে, আসল file অপরিবর্তিত থাকে <br>`s` = **substitute** |
| `sed -i 's/Hello/obhai/' nure.log` | সরাসরি file-এর ভিতরেই change save করে <br>`-i` = **in-place** |
| `sed 's/Hello/obhai/' nure.log > rafi.log` | Update হওয়া output নতুন একটা file (`rafi.log`)-এ লিখে দেয়, মূল file ঠিক থাকে <br>`>` = **redirect** |

> **Note:** `sed 's/Hello/obhai/'` প্রত্যেক line-এর **প্রথম** match-টাই বদলায়। পুরো line-এর সব match বদলাতে চাইলে শেষে `g` দিতে হয় — `sed 's/Hello/obhai/g' nure.log` <br>`g` = **global**

## Shell Scripting

**Shell** হচ্ছে user আর operating system-এর মাঝখানে কাজ করা একটা **interface**।

### প্রথম Shell Script

Shell file-এর extension হয় **`.sh`**। একটা file বানাই — `app.sh`:

```bash
#!/bin/bash

echo "Hello from shell, DevOps friends"
```

| বিষয় | Description |
|------|-------------|
| `#!/bin/bash` | Shell file সবসময় এটা দিয়ে শুরু করতে হয় — এটাকে বলে **shebang** <br>Programming-এর দুনিয়ায় `#` কে **sha (hash)** আর `!` কে **bang** বলা হয় → **she-bang** <br>এটা system-কে বলে দেয় script-টা কোন interpreter (এখানে `bash`) দিয়ে run হবে |
| `echo` | Screen-এ কিছু print করে |

### Script Run করা

| Command | Description |
|---------|-------------|
| `bash app.sh` | Script run করে (file-এ execute permission না থাকলেও চলবে) |
| `chmod 700 app.sh` | File-কে executable বানায় — owner-এর `rwx` (7), group ও others-এর কিছুই না (0, 0) |
| `./app.sh` | Permission দেওয়ার পর `bash` ছাড়াই সরাসরি run করা যায় |

### Variable

```bash
#!/bin/bash

greetings="kmn acho"
echo $greetings
```

> **গুরুত্বপূর্ণ:** Bash-এ `=` চিহ্নের **আগে-পরে space দেওয়া যাবে না**। `greetings = "kmn acho"` লিখলে error দেবে। <br>Variable-এর মান ব্যবহার করার সময় সামনে `$` দিতে হয় — `$greetings`

### User Input নেওয়া

```bash
#!/bin/bash

echo "Enter your name"
read name
echo "taile tumar name $name"
```

| Command | Description |
|---------|-------------|
| `read name` | User যা টাইপ করবে তা `name` variable-এ জমা রাখে |

### যোগ করা (Arithmetic)

```bash
#!/bin/bash

echo "Enter your number1"
read number1

echo "Enter your number2"
read number2

sum=$((number1 + number2))
echo "Sum is: $sum"
```

| বিষয় | Description |
|------|-------------|
| `$(( ... ))` | **Arithmetic expansion** — এর ভিতরে গাণিতিক হিসাব করা যায় (`+`, `-`, `*`, `/`, `%`) |
| `echo $sum` | `$` ছাড়া শুধু `echo sum` লিখলে literally "sum" শব্দটাই print হবে, মান নয় |

---