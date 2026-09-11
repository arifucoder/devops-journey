# Contabo VPS-এ SSH Key দিয়ে Login Setup

এই গাইডে দেখানো হয়েছে কীভাবে আলাদা একটি folder-এ SSH key বানিয়ে, সেটা VPS-এ যুক্ত করে, আর `ssh config` দিয়ে শুধু `ssh contabo` লিখেই VPS-এ login করা যায়। এটা ম্যাক পিসি থেকে লগইন করা হয়েছে। 

---

## ধাপ ১: Local PC-তে SSH Key বানানো

### ১.১ Key রাখার জন্য আলাদা folder তৈরি করা

সব key এক জায়গায় না রেখে VPS-এর জন্য আলাদা folder বানিয়ে নিলে গোছানো থাকে:

```sh
mkdir -p ~/.ssh/contabo-vps
```

### ১.২ Key generate করা

```sh
ssh-keygen -t ed25519 -f ~/.ssh/contabo-vps/contabo -C "contabo-vps"
```

- `-t ed25519` → modern ও নিরাপদ key type
- `-f` → key কোথায় আর কী নামে save হবে
- `-C` → key-এর comment (চেনার সুবিধার জন্য)

Passphrase চাইলে দিতে পারো, না চাইলে Enter চেপে খালি রাখতে পারো।

### ১.৩ Folder-এ দুটি file দেখা যাবে

```sh
ls ~/.ssh/contabo-vps
```

| File | কাজ |
|------|-----|
| `contabo` | **Private key** — কখনো কাউকে দেওয়া যাবে না |
| `contabo.pub` | **Public key** — এটাই VPS-এ বসাতে হবে |

### ১.৪ Public key copy করা

```sh
cat ~/.ssh/contabo-vps/contabo.pub
```

Output-এর পুরো লাইনটা (`ssh-ed25519 AAAA... contabo-vps`) copy করে রাখো।

---

## ধাপ ২: VPS-এ Public Key যুক্ত করা

### ২.১ Password দিয়ে VPS-এ login করা

```sh
ssh root@81.17.101.34
```

Contabo থেকে পাওয়া password দিয়ে login করো।

### ২.২ System update করা

```sh
sudo apt update
sudo apt upgrade -y
```

### ২.৩ `.ssh` folder আছে কিনা check করা

```sh
ls -la ~/.ssh
```

**যদি folder না থাকে:**

```sh
mkdir -p ~/.ssh
vim ~/.ssh/authorized_keys
```

**যদি folder থাকে:**

```sh
vim ~/.ssh/authorized_keys
```

### ২.৪ Public key paste করা

`vim` খোলার পর:

1. `i` চেপে insert mode-এ যাও
2. Copy করা public key paste করো (একটা নতুন line-এ)
3. `Esc` চাপো, তারপর `:wq` লিখে Enter — save হয়ে বের হবে

### ২.৫ Permission ঠিক করা

Permission ঠিক না থাকলে SSH key কাজ করবে না, তাই এটা জরুরি:

```sh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

এরপর `exit` লিখে VPS থেকে বের হয়ে আসো।

---

## ধাপ ৩: Key দিয়ে Login Test করা

Local PC থেকে:

```sh
ssh -i ~/.ssh/contabo-vps/contabo root@81.17.101.34
```

Password না চেয়ে সরাসরি login হলে বুঝবে key ঠিকমতো কাজ করছে। ✅

---

## ধাপ ৪: SSH Config দিয়ে Shortcut বানানো

প্রতিবার লম্বা command লেখার বদলে একটা shortcut বানিয়ে নেওয়া যায়।

### ৪.১ Config file আছে কিনা check করা

```sh
ls ~/.ssh/config
```

না থাকলে বানিয়ে নাও:

```sh
touch ~/.ssh/config
```

### ৪.২ Permission ঠিক করা

```sh
chmod 600 ~/.ssh/config
```

### ৪.৩ Config file edit করা

```sh
vim ~/.ssh/config
```

নিচের অংশটুকু যোগ করো:

```sh
Host contabo
    HostName 81.17.101.34
    User root
    IdentityFile ~/.ssh/contabo-vps/contabo
    IdentitiesOnly yes
```

| Option | মানে |
|--------|------|
| `Host` | Shortcut-এর নাম (যেটা লিখে ssh করবে) |
| `HostName` | VPS-এর IP address |
| `User` | কোন user হিসেবে login হবে |
| `IdentityFile` | কোন private key ব্যবহার হবে |
| `IdentitiesOnly yes` | শুধু এই key-টাই try করবে, অন্য key না |

`:wq` দিয়ে save করে বের হও।

---

## ধাপ ৫: Shortcut দিয়ে Login

এখন থেকে terminal-এ শুধু এটুকু লিখলেই VPS-এ login হয়ে যাবে:

```sh
ssh contabo
```

🎉 Setup শেষ!

---

## Quick Reference

```sh
# Key বানানো
mkdir -p ~/.ssh/contabo-vps
ssh-keygen -t ed25519 -f ~/.ssh/contabo-vps/contabo -C "contabo-vps"
cat ~/.ssh/contabo-vps/contabo.pub

# VPS-এ (public key paste করার পর)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Local PC-তে config
chmod 600 ~/.ssh/config

# Login
ssh contabo
```