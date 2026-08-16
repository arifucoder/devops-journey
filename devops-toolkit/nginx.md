# Nginx Class Notes — SSH, Nginx, Reverse Proxy ও SSL

> এই নোটে সব ব্যাখ্যা বাংলায়, কিন্তু সব **command, config, file path ও technical term ইংরেজিতে** রাখা হয়েছে।
> ✅ চিহ্ন দেওয়া অংশগুলো সবচেয়ে গুরুত্বপূর্ণ / মনে রাখার মতো পয়েন্ট।

---

## সূচিপত্র

**PART 1 — আমার ক্লাস নোট**
1. [SSH কী](#1-ssh-কী)
2. [Common Network Ports](#2-common-network-ports)
3. [Nginx পরিচিতি](#3-nginx-পরিচিতি)
4. [Nginx-এর প্রয়োজনীয় Command](#4-nginx-এর-প্রয়োজনীয়-command)
5. [Static Site Host করা (figmaland example)](#5-static-site-host-করা)
6. [Express.js Backend Host করা (PM2 + Nginx)](#6-expressjs-backend-host-করা-pm2--nginx)
7. [Domain সেটআপ — A Record ও CNAME](#7-domain-সেটআপ--a-record-ও-cname)
8. [HTTPS / SSL যুক্ত করা (Certbot)](#8-https--ssl-যুক্ত-করা-certbot)

**PART 2 — অতিরিক্ত নোট (নতুন সংযোজন)**

9. [Nginx-এর File Structure](#9-nginx-এর-file-structure)
10. [Server Block-এর প্রতিটি Directive-এর ব্যাখ্যা](#10-server-block-এর-প্রতিটি-directive-এর-ব্যাখ্যা)
11. [একই সার্ভারে ৩–৪টি Application চালানোর উপায়](#11-একই-সার্ভারে-৩৪টি-application-চালানোর-উপায়)
12. [SSL দেওয়ার বিভিন্ন পদ্ধতি](#12-ssl-দেওয়ার-বিভিন্ন-পদ্ধতি)
13. [SSL Auto-Renewal](#13-ssl-auto-renewal)
14. [Production-ready SSL Config (Hardening)](#14-production-ready-ssl-config-hardening)
15. [সাধারণ Error ও তার সমাধান](#15-সাধারণ-error-ও-তার-সমাধান)
16. [এক নজরে মূল পয়েন্ট](#এক-নজরে-মূল-পয়েন্ট)

---

# PART 1 — আমার ক্লাস নোট

## 1. SSH কী

**SSH = Secure Shell.** এটি একটি network protocol যার মাধ্যমে দূরের (remote) কোনো server-এ **encrypted ভাবে** connect করে command line access পাওয়া যায়।

**Definition:** SSH হলো client ও server-এর মধ্যে একটি নিরাপদ, encrypted tunnel, যেখানে data plain text-এ যায় না — ফলে মাঝপথে কেউ পড়তে পারে না।

**Example 1 —** AWS EC2 instance-এ key file দিয়ে login:

```bash
ssh -i my-key.pem ubuntu@13.229.45.10
```

**Example 2 —** password দিয়ে সাধারণ VPS-এ login:

```bash
ssh root@203.0.113.25
```

✅ SSH ব্যবহারের ফলে server-এর সাথে সব communication encrypted থাকে — তাই Telnet বা plain FTP-এর বদলে সবসময় SSH/SFTP ব্যবহার করতে হবে।

---

## 2. Common Network Ports

| Port | Protocol | কাজ |
|------|----------|-----|
| **22** | SSH (Secure Shell) | Linux server-এ নিরাপদে command line access করার জন্য |
| **21** | FTP (File Transfer Protocol) | সাধারণ file transfer (unencrypted) |
| **22** | SFTP (SSH File Transfer Protocol) | SSH-এর উপর দিয়ে নিরাপদ ও encrypted file transfer |
| **80** | HTTP | সাধারণ ও অনিরাপদ web browsing |
| **443** | HTTPS | নিরাপদ ও encrypted web communication |

✅ **SFTP আলাদা কোনো port ব্যবহার করে না** — এটি SSH-এরই port **22** ব্যবহার করে। তাই SSH খোলা থাকলে SFTP এমনিতেই কাজ করে, আলাদা করে port খুলতে হয় না।

> ⚠️ FTP (21) data plain text-এ পাঠায়, এমনকি password-ও। Production-এ FTP-এর বদলে **SFTP** ব্যবহার করাই নিয়ম।

**অতিরিক্ত যেসব port বাস্তবে দরকার হয়:**

| Port | Service |
|------|---------|
| 3306 | MySQL / MariaDB |
| 5432 | PostgreSQL |
| 27017 | MongoDB |
| 6379 | Redis |
| 3000 / 5000 / 8000 | Node.js, Django ইত্যাদি app-এর default dev port |

✅ Database port (3306, 5432, 27017) **কখনোই public inbound rule-এ open করা যাবে না** — শুধু নিজের server বা VPC-এর ভেতর থেকে access দিতে হবে।

---

## 3. Nginx পরিচিতি

**Nginx** হলো একটি **Web Server** এবং **Reverse Proxy**।

### Nginx-এর ব্যবহারসমূহ

| ব্যবহার | ব্যাখ্যা |
|---------|----------|
| **Web Server** | HTML/CSS/JS-এর মতো static file browser-কে serve করা |
| **Reverse Proxy** | বাইরের request নিয়ে ভেতরের app (Node/Django/Laravel)-এ পাঠানো |
| **Caching** | একই response বারবার না বানিয়ে জমিয়ে রেখে দ্রুত দেওয়া |
| **Static Content Hosting** | Image, CSS, JS দ্রুত serve করা |
| **Multiple Site Hosting** | একই server-এ অনেকগুলো domain/site চালানো |
| **API Gateway** | একাধিক backend service-কে একটি entry point-এর নিচে আনা |
| **Load Balancing** | Traffic কে একাধিক server-এর মধ্যে ভাগ করে দেওয়া |

**Definition — Reverse Proxy:** যে server client-এর request নিজে গ্রহণ করে, তারপর সেটি পেছনের (backend) কোনো application-এ পাঠায় এবং উত্তর ফেরত এনে client-কে দেয়। Client কখনোই backend-কে সরাসরি দেখে না।

**Example 1 —** ব্যবহারকারী `https://devsskills.com` এ যায় → Nginx সেটি `http://127.0.0.1:5000` (Express app)-এ পাঠায়।
**Example 2 —** `https://devsskills.com/api` → Node backend, আর `https://devsskills.com/` → React build folder।

✅ Reverse proxy ব্যবহার করলে app-এর আসল port (5000, 3000) বাইরে থেকে কেউ জানতে বা ধরতে পারে না — এটি security-র জন্য বড় সুবিধা।

---

## 4. Nginx-এর প্রয়োজনীয় Command

```bash
# Nginx install করা
sudo apt update
sudo apt install nginx -y

# Nginx-এর status দেখা
systemctl status nginx

# Configuration ঠিক আছে কিনা check করা (reload-এর আগে বাধ্যতামূলক)
sudo nginx -t

# Config update করার পর reload করা
sudo systemctl reload nginx

# একদম বন্ধ করে চালু করা (প্রয়োজনে)
sudo systemctl restart nginx

# Server reboot হলেও Nginx যেন নিজে চালু হয়
sudo systemctl enable nginx
```

✅ **`reload` না `restart`?** → `reload` করলে চলমান connection নষ্ট হয় না, তাই **downtime হয় না**। `restart` করলে service পুরোপুরি বন্ধ হয়ে আবার চালু হয় — live site-এ কয়েক মুহূর্তের downtime হতে পারে।

✅ **নিয়ম:** সবসময় আগে `sudo nginx -t`, তারপর `sudo systemctl reload nginx`। Test fail করলে **কখনোই reload করা যাবে না** — reload করলে live site down হয়ে যেতে পারে।

---

## 5. Static Site Host করা

### ধাপ ১ — `conf.d` folder-এ domain-এর নামে config file বানানো

```bash
sudo nano /etc/nginx/conf.d/something.com.conf
```

### ধাপ ২ — Server block লেখা

```nginx
server {
    listen 80 default_server;          # port 80-এ শুনবে, এবং কোনো server_name না মিললে এটিই default
    server_name _;                     # _ মানে "যেকোনো নাম" (catch-all)
    root /var/www/figmaland;           # ফাইলগুলো কোন folder থেকে serve হবে
    index index.html index.htm;        # folder চাইলে কোন ফাইলটি দেখাবে

    location / {
        try_files $uri $uri/ =404;     # আগে file, তারপর folder খুঁজবে; না পেলে 404 দেবে
    }
}
```

### ধাপ ৩ — Website-এর ফাইল রাখা

```bash
cd /var/www/
sudo mkdir figmaland
# অথবা GitHub থেকে clone করে আনা
sudo git clone https://github.com/username/figmaland.git figmaland
```

✅ এখানে `git` command-এর সাথে **`sudo` লাগবে**, কারণ আমরা `/var/www/` এ কাজ করছি — এটি ubuntu user-এর home directory নয়, তাই সাধারণ user-এর write permission নেই।

### ধাপ ৪ — Test ও Reload

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### ধাপ ৫ — কাজ করেছে কিনা যাচাই

```bash
curl -I http://localhost          # HTTP/1.1 200 OK আসা উচিত
```

অথবা browser-এ server-এর IP লিখে দেখা।

✅ Permission ঠিক না থাকলে **403 Forbidden** আসে। ঠিক করার command:

```bash
sudo chown -R www-data:www-data /var/www/figmaland
sudo chmod -R 755 /var/www/figmaland
```

---

## 6. Express.js Backend Host করা (PM2 + Nginx)

লক্ষ্য: EC2-তে একটি Express.js backend চালানো, PM2 দিয়ে সবসময় চালু রাখা, আর Nginx দিয়ে reverse proxy করা।

### ধাপ ১ — Home folder-এ project folder বানানো

```bash
cd ~
mkdir application
cd application
```

### ধাপ ২ — Node.js ও Yarn install করা (project clone করার আগেই)

```bash
# Node.js (NodeSource থেকে LTS version)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Yarn (যদি development-এ yarn ব্যবহার করা হয়ে থাকে)
sudo npm install -g yarn

# Version check
node -v
npm -v
yarn -v
```

### ধাপ ৩ — Repository clone ও package install

```bash
git clone https://github.com/username/my-api.git
cd my-api

yarn install        # অথবা: npm install
```

✅ Home folder (`/home/ubuntu`)-এ কাজ করার সময় **`sudo` লাগে না**, কারণ এই folder-এর মালিক ubuntu user নিজেই।

### ধাপ ৪ — PM2 দিয়ে app চালানো

**Definition — PM2:** এটি Node.js-এর একটি process manager। এটি app-কে background-এ চালায়, crash করলে নিজে থেকে restart করে, এবং server reboot হলেও আবার চালু করতে পারে।

**Example 1 —** SSH বন্ধ করলেও app চলতে থাকে।
**Example 2 —** app crash করলে PM2 সাথে সাথে আবার চালু করে দেয়।

```bash
sudo npm install -g pm2

# App চালু করা
pm2 start index.js --name my-api

# চলছে কিনা দেখা
pm2 list
pm2 logs my-api

# Server reboot হলেও যেন আবার চালু হয়
pm2 startup
pm2 save
```

✅ `pm2 save` না দিলে reboot-এর পর app আর চালু হবে না — এটি খুব সাধারণ একটি ভুল।

### ধাপ ৫ — Local-এ কাজ করছে কিনা যাচাই

```bash
curl http://127.0.0.1:5000
```

Response এলে বুঝতে হবে backend ঠিকমতো চলছে — এরপরই Nginx-এর কাজ শুরু।

### ধাপ ৬ — Nginx install ও default site বন্ধ করা

```bash
sudo apt install nginx -y
```

Ubuntu-তে `/etc/nginx/nginx.conf` ফাইলের ভেতরে দুটি line থাকে:

```nginx
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

আমরা যেহেতু `conf.d` ব্যবহার করছি, তাই `sites-enabled` এর line টি comment করে দিতে হবে:

```nginx
include /etc/nginx/conf.d/*.conf;
# include /etc/nginx/sites-enabled/*;
```

✅ **আরও পরিষ্কার বিকল্প:** line comment না করে শুধু default site-এর symlink মুছে দেওয়া। এতে Ubuntu-র মূল structure অক্ষত থাকে:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

> ⚠️ Default site না সরালে server IP-তে গেলে Nginx-এর "Welcome to nginx!" page দেখাবে, নিজের site নয় — কারণ default site টি `default_server` হিসেবে সব request ধরে ফেলে।

### ধাপ ৭ — Reverse proxy config লেখা

```bash
sudo nano /etc/nginx/conf.d/devsskills.com.conf
```

```nginx
server {
    listen 80;
    server_name devsskills.com www.devsskills.com;

    location / {
        proxy_pass http://127.0.0.1:5000;              # app যেই port-এ চলছে
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

✅ `server_name` এ domain-এর বানান হুবহু ঠিক থাকতে হবে — `www.devsskills.com` (একটি অক্ষর ভুল হলেও `www` version কাজ করবে না)।

**এখানে যা হচ্ছে:**
- `listen 80` → port 80-এ request শুনবে
- `server_name` → কোন domain-এর জন্য এই block কাজ করবে
- `location /` এর ভেতরে `proxy_pass` → এটিই মূল **reverse proxy**; request কে backend-এ পাঠিয়ে দিচ্ছে

### ধাপ ৮ — Test ও Reload

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### ধাপ ৯ — Firewall / Inbound Rules

প্রথমে দেখতে হবে **HTTP (port 80)** inbound rules-এ আছে কিনা; না থাকলে add করতে হবে।

- **AWS EC2** → Security Group → Inbound rules → Add rule → Type: HTTP, Port: 80, Source: 0.0.0.0/0
- **Ubuntu-র নিজের firewall (UFW)** চালু থাকলে:

```bash
sudo ufw allow 'Nginx Full'     # port 80 এবং 443 দুটোই খুলে দেয়
sudo ufw status
```

✅ Cloud provider-এর Security Group এবং server-এর নিজের UFW — **দুটোই আলাদা firewall**। একটি খোলা থাকলেও অন্যটি বন্ধ থাকলে site কাজ করবে না।

---

## 7. Domain সেটআপ — A Record ও CNAME

Domain panel-এর **Manage / DNS** section থেকে record add করতে হবে।

| Record Type | Host | Value | কাজ |
|-------------|------|-------|-----|
| **A** | `@` | `13.229.45.10` (instance IP) | Domain কে server-এর IP-তে পাঠায় |
| **CNAME** | `www` | `devsskills.com` | `www` version কে মূল domain-এ পাঠায় |

**Definition — CNAME:** CNAME অর্থ **Canonical Name**। এটি একটি নামকে অন্য একটি নামের **alias** বানায়, IP-তে নয়।

**Example 1 —** `www.devsskills.com` → CNAME → `devsskills.com` → A record → IP
**Example 2 —** `blog.devsskills.com` → CNAME → `myblog.netlify.app`

✅ DNS change সাথে সাথে কাজ নাও করতে পারে — **propagation** এ কয়েক মিনিট থেকে কয়েক ঘণ্টা লাগতে পারে। যাচাই করার command:

```bash
dig +short devsskills.com
nslookup devsskills.com
```

---

## 8. HTTPS / SSL যুক্ত করা (Certbot)

### ধাপ ১ — Certbot install

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### ধাপ ২ — SSL certificate নেওয়া

```bash
sudo certbot --nginx -d devsskills.com -d www.devsskills.com
```

### ধাপ ৩ — Port 443 খোলা

Instance-এর inbound rules-এ **HTTPS (port 443)** add করতে হবে।

- **AWS** → Security Group → Add rule → Type: HTTPS, Port: 443, Source: 0.0.0.0/0
- **UFW** ব্যবহার করলে: `sudo ufw allow 'Nginx Full'`

### ধাপ ৪ — যাচাই

```bash
curl -I https://devsskills.com
sudo certbot certificates          # কোন domain-এ কবে পর্যন্ত cert আছে দেখায়
```

✅ **SSL নেওয়ার আগে অবশ্যই DNS ঠিকভাবে server-এ point করতে হবে এবং port 80 খোলা থাকতে হবে।** Certbot নিজে port 80-এ একটি validation file বসিয়ে ডোমেইনের মালিকানা প্রমাণ করে — DNS ঠিক না থাকলে বা port 80 বন্ধ থাকলে certificate issue **fail** করবে।

---

# PART 2 — অতিরিক্ত নোট (নতুন সংযোজন)

## 9. Nginx-এর File Structure

| Path | কাজ |
|------|-----|
| `/etc/nginx/nginx.conf` | প্রধান config file; এখান থেকেই বাকি সব file include হয় |
| `/etc/nginx/conf.d/` | নিজের বানানো `.conf` file রাখার folder (আমরা এটি ব্যবহার করছি) |
| `/etc/nginx/sites-available/` | Ubuntu-র নিজস্ব পদ্ধতি — সব site-এর config এখানে থাকে |
| `/etc/nginx/sites-enabled/` | যে site গুলো **চালু**, তাদের symlink এখানে থাকে |
| `/var/www/` | Website-এর ফাইল রাখার default জায়গা |
| `/var/log/nginx/access.log` | কে কখন কোন page চেয়েছে তার log |
| `/var/log/nginx/error.log` | ✅ কোনো সমস্যা হলে **সবার আগে এখানে দেখতে হবে** |

**Log দেখার command:**

```bash
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

### sites-available পদ্ধতি (Ubuntu-র নিজস্ব উপায়)

```bash
sudo nano /etc/nginx/sites-available/devsskills.com
sudo ln -s /etc/nginx/sites-available/devsskills.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

✅ `conf.d` আর `sites-available` — **দুটোর যেকোনো একটি বেছে নিতে হবে**, দুটো একসাথে মেশালে একই server_name দুবার আসতে পারে এবং Nginx conflict warning দেয়।

---

## 10. Server Block-এর প্রতিটি Directive-এর ব্যাখ্যা

```nginx
server {
    listen 80;
    server_name devsskills.com;
    root /var/www/site;
    index index.html;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

| Directive | কাজ |
|-----------|-----|
| `server { }` | একটি website/domain-এর সম্পূর্ণ নিয়মকানুন এই block-এ থাকে |
| `listen 80;` | কোন port-এ request শুনবে (80 = HTTP, 443 = HTTPS) |
| `default_server` | কোনো `server_name` না মিললে এই block টি request handle করবে |
| `server_name` | কোন domain-এর জন্য এই block; একাধিক domain space দিয়ে লেখা যায় |
| `root` | File গুলো কোন folder থেকে খুঁজবে |
| `index` | Folder চাওয়া হলে কোন file দেখাবে (বাম থেকে ডানে খুঁজবে) |
| `location / { }` | কোন URL path-এর জন্য কোন নিয়ম চলবে |
| `try_files $uri $uri/ =404;` | আগে ওই নামের file, তারপর folder খুঁজবে; না পেলে 404 |
| `proxy_pass` | ✅ Request কে backend app-এ পাঠিয়ে দেয় — এটিই reverse proxy-র মূল directive |
| `proxy_http_version 1.1;` | Backend-এর সাথে HTTP/1.1 ব্যবহার করবে (WebSocket-এর জন্য দরকার) |
| `proxy_set_header Upgrade $http_upgrade;` | WebSocket connection upgrade করার header পাঠায় |
| `proxy_set_header Connection 'upgrade';` | Connection টিকে upgrade type হিসেবে চিহ্নিত করে |
| `proxy_set_header Host $host;` | ✅ আসল domain নাম backend-কে জানায়, নাহলে backend `127.0.0.1` দেখে |
| `proxy_set_header X-Real-IP $remote_addr;` | ✅ ব্যবহারকারীর আসল IP backend-কে পাঠায় |
| `proxy_set_header X-Forwarded-For ...` | একাধিক proxy পার হলে পুরো IP chain পাঠায় |
| `proxy_set_header X-Forwarded-Proto $scheme;` | ✅ Request http না https ছিল তা জানায় (redirect loop এড়াতে দরকার) |
| `proxy_cache_bypass $http_upgrade;` | WebSocket request গুলো cache করবে না |

✅ `X-Real-IP` ও `X-Forwarded-For` না দিলে backend-এর log-এ সব request-এর IP দেখাবে `127.0.0.1` — rate limiting বা analytics পুরোপুরি ভুল হয়ে যাবে।

---

## 11. একই সার্ভারে ৩–৪টি Application চালানোর উপায়

একটিমাত্র server-এ একাধিক application চালানোর **তিনটি পদ্ধতি** আছে:

| পদ্ধতি | উদাহরণ | কখন ব্যবহার করবেন |
|--------|---------|-------------------|
| **Separate domain** | `sitea.com`, `siteb.com` | আলাদা আলাদা client বা product |
| **Subdomain** | `api.devsskills.com`, `admin.devsskills.com` | একই product-এর ভিন্ন অংশ |
| **Path-based** | `devsskills.com/api`, `devsskills.com/admin` | একটি domain-এ সবকিছু রাখতে চাইলে |

প্রথমে ধরে নিই আমাদের ৪টি application চলছে:

| # | Application | ধরন | Port |
|---|-------------|-----|------|
| 1 | Figmaland | Static HTML site | — (file থেকে serve) |
| 2 | Express API | Node.js backend | 5000 |
| 3 | Admin Panel | React SPA (build folder) | — (file থেকে serve) |
| 4 | Next.js Site | Node.js SSR app | 3000 |

---

### পদ্ধতি ১ — আলাদা Domain, আলাদা File

প্রতিটি site-এর জন্য **আলাদা `.conf` file** বানানোই সবচেয়ে পরিষ্কার উপায়।

```bash
sudo nano /etc/nginx/conf.d/figmaland.com.conf
sudo nano /etc/nginx/conf.d/devsskills.com.conf
```

**App 1 — Static HTML site** (`/etc/nginx/conf.d/figmaland.com.conf`)

```nginx
server {
    listen 80;
    server_name figmaland.com www.figmaland.com;

    root /var/www/figmaland;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Image/CSS/JS এক বছর ধরে browser-এ cache করে রাখবে — site অনেক দ্রুত হবে
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

| Directive | ব্যাখ্যা |
|-----------|----------|
| `location ~* \.(...)$` | `~*` মানে **case-insensitive regex match**; নির্দিষ্ট extension-এর file গুলোর জন্য নিয়ম |
| `expires 1y;` | Browser এই file গুলো ১ বছর নিজের কাছে জমা রাখবে |
| `add_header Cache-Control "public, immutable"` | File পরিবর্তন হবে না — বারবার check করার দরকার নেই |

---

**App 2 — Express API** (`/etc/nginx/conf.d/api.devsskills.com.conf`)

```nginx
server {
    listen 80;
    server_name api.devsskills.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    client_max_body_size 20M;    # File upload-এর সর্বোচ্চ size (default মাত্র 1M)
}
```

✅ `client_max_body_size` না বাড়ালে বড় file upload করতে গেলে **413 Request Entity Too Large** error আসে — এটি খুব সাধারণ একটি সমস্যা।

---

**App 3 — React SPA (Admin Panel)** (`/etc/nginx/conf.d/admin.devsskills.com.conf`)

```nginx
server {
    listen 80;
    server_name admin.devsskills.com;

    root /var/www/admin/build;     # React-এর build folder
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;    # ✅ SPA-র জন্য সবচেয়ে জরুরি line
    }
}
```

✅ React/Vue/Angular-এর মতো **SPA (Single Page Application)** এ `try_files $uri $uri/ /index.html;` **অবশ্যই** দিতে হবে। কারণ SPA-তে routing browser-এর ভেতরে হয়, server-এ `/dashboard` নামের কোনো ফাইল আসলে নেই। এই line না দিলে page refresh করলেই **404 Not Found** আসবে।

**Definition — SPA (Single Page Application):** এমন web app যেখানে একবারই `index.html` load হয়, তারপর সব page পরিবর্তন JavaScript দিয়ে browser-এই হয়।

**Example 1 —** React app-এ `/dashboard` এ গেলে নতুন করে page load হয় না।
**Example 2 —** Gmail — mail খুললে পুরো page reload হয় না।

---

**App 4 — Next.js SSR app** (`/etc/nginx/conf.d/blog.devsskills.com.conf`)

```nginx
server {
    listen 80;
    server_name blog.devsskills.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Next.js-এর static asset গুলো দ্রুত serve করার জন্য
    location /_next/static/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_cache_valid 60m;
        expires 1y;
        access_log off;
    }
}
```

✅ Next.js **build করার পরেই** PM2 দিয়ে চালাতে হবে:

```bash
yarn build
pm2 start "yarn start" --name blog
pm2 save
```

---

### পদ্ধতি ২ — একটিই Domain, আলাদা Path

একই domain-এর নিচে সব app রাখতে চাইলে:

```nginx
server {
    listen 80;
    server_name devsskills.com www.devsskills.com;

    # মূল site — static React build
    root /var/www/frontend/build;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # /api → Express backend (port 5000)
    location /api/ {
        proxy_pass http://127.0.0.1:5000/;      # ⚠️ শেষের / খুব গুরুত্বপূর্ণ
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # /blog → Next.js (port 3000)
    location /blog/ {
        proxy_pass http://127.0.0.1:3000;       # এখানে শেষে / নেই — ইচ্ছাকৃত
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### ✅ `proxy_pass`-এর শেষের `/` — সবচেয়ে বেশি ভুল হয় এখানে

| Config | ব্যবহারকারী চায় | Backend পায় |
|--------|------------------|--------------|
| `proxy_pass http://127.0.0.1:5000/;` (**শেষে `/` আছে**) | `/api/users` | `/users` ← prefix কেটে যায় |
| `proxy_pass http://127.0.0.1:5000;` (**শেষে `/` নেই**) | `/api/users` | `/api/users` ← পুরো path যায় |

✅ **নিয়ম:** Backend-এর route যদি `/users` হয় → শেষে `/` **দিতে হবে**। Backend-এর route যদি নিজেই `/api/users` হয় → শেষে `/` **দেওয়া যাবে না**।

---

### পদ্ধতি ৩ — Load Balancing (একই app একাধিক copy)

একই application যদি ৩টি port-এ চলে (3001, 3002, 3003), তাহলে Nginx traffic ভাগ করে দিতে পারে:

```nginx
upstream my_app {
    least_conn;                     # যে server-এ সবচেয়ে কম connection, সেখানে পাঠাবে
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;
    server_name app.devsskills.com;

    location / {
        proxy_pass http://my_app;   # upstream-এর নাম বসবে
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

| Directive | ব্যাখ্যা |
|-----------|----------|
| `upstream name { }` | একদল backend server-এর একটি group তৈরি করে |
| `least_conn;` | সবচেয়ে কম ব্যস্ত server-এ request পাঠায় |
| `ip_hash;` | একই user সবসময় একই server-এ যাবে (session ধরে রাখতে) |

PM2 দিয়ে একই app-এর একাধিক copy চালানো:

```bash
pm2 start index.js -i 3 --name my-app     # CPU core অনুযায়ী 3টি instance
```

---

### সব App যোগ করার পর যাচাই

```bash
sudo nginx -t                       # সব file একসাথে test হবে
sudo systemctl reload nginx

# প্রতিটি domain আলাদা করে check
curl -I http://figmaland.com
curl -I http://api.devsskills.com
curl -I http://admin.devsskills.com
curl -I http://blog.devsskills.com
```

> ⚠️ **সতর্কতা:** একটি config file-এ syntax error থাকলে `nginx -t` fail করবে এবং **সবগুলো site একসাথে বন্ধ হয়ে যাবে** — শুধু ওই একটি নয়। তাই live server-এ reload-এর আগে `nginx -t` বাধ্যতামূলক।

---

## 12. SSL দেওয়ার বিভিন্ন পদ্ধতি

SSL certificate পাওয়ার একাধিক উপায় আছে। পরিস্থিতি অনুযায়ী কোনটি বাছবেন:

| # | পদ্ধতি | কখন ব্যবহার করবেন |
|---|--------|-------------------|
| 1 | `certbot --nginx` | ✅ সাধারণ ক্ষেত্রে সবচেয়ে ভালো — সব automatic |
| 2 | `certbot certonly --webroot` | Nginx চলন্ত অবস্থায় রেখে শুধু cert নিতে চাইলে |
| 3 | `certbot certonly --standalone` | Nginx এখনো install হয়নি বা বন্ধ আছে |
| 4 | DNS-01 challenge | ✅ **Wildcard certificate** (`*.domain.com`) দরকার হলে |
| 5 | কেনা CA certificate (manual) | কোম্পানি যদি DigiCert/Sectigo থেকে cert কেনে |
| 6 | Self-signed certificate | শুধু local/dev testing-এর জন্য |
| 7 | Cloudflare Origin Certificate | Site যদি Cloudflare-এর পেছনে থাকে |
| 8 | `acme.sh` client | Certbot-এর হালকা বিকল্প |

---

### পদ্ধতি ১ — `certbot --nginx` (সবচেয়ে সহজ, recommended)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d devsskills.com -d www.devsskills.com
```

Certbot নিজে থেকেই:
- Certificate নিয়ে আসে
- আপনার `.conf` file **নিজে edit করে** `listen 443 ssl;` ও certificate path যোগ করে
- HTTP → HTTPS redirect যোগ করে দেয়
- Auto-renewal timer সেট করে দেয়

✅ Certbot চালানোর পর আপনার config file দেখতে এমন হবে:

```nginx
server {
    server_name devsskills.com www.devsskills.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl;                                                       # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/devsskills.com/fullchain.pem;   # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/devsskills.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf;                      # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;                        # managed by Certbot
}

server {
    if ($host = www.devsskills.com) {
        return 301 https://$host$request_uri;
    }
    if ($host = devsskills.com) {
        return 301 https://$host$request_uri;
    }
    listen 80;
    server_name devsskills.com www.devsskills.com;
    return 404;
}
```

| Directive | ব্যাখ্যা |
|-----------|----------|
| `listen 443 ssl;` | HTTPS port-এ শুনবে এবং SSL চালু থাকবে |
| `ssl_certificate` | Public certificate + intermediate chain (`fullchain.pem`) |
| `ssl_certificate_key` | ✅ Private key — এটি **কখনোই** কারও সাথে share বা git-এ commit করা যাবে না |
| `include options-ssl-nginx.conf` | Let's Encrypt-এর তৈরি নিরাপদ default settings |
| `ssl_dhparam` | Diffie-Hellman parameter, key exchange আরও শক্ত করে |
| `return 301 https://$host$request_uri;` | ✅ HTTP-তে আসা সব request স্থায়ীভাবে HTTPS-এ পাঠায় |

> ⚠️ Certbot আপনার config file **নিজে পরিবর্তন করে**। তাই চালানোর আগে backup রাখা ভালো:
> ```bash
> sudo cp /etc/nginx/conf.d/devsskills.com.conf ~/devsskills.com.conf.bak
> ```

---

### পদ্ধতি ২ — `certbot certonly --webroot`

Nginx বন্ধ না করে শুধু certificate নিয়ে আসে, config file-এ হাত দেয় না। ✅ Live site-এর জন্য সবচেয়ে নিরাপদ, কারণ **কোনো downtime হয় না**।

প্রথমে config-এ validation path যোগ করতে হবে:

```nginx
server {
    listen 80;
    server_name devsskills.com;

    # Certbot এখানে validation file রাখবে
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        proxy_pass http://127.0.0.1:5000;
    }
}
```

```bash
sudo mkdir -p /var/www/certbot
sudo nginx -t
sudo systemctl reload nginx

sudo certbot certonly --webroot -w /var/www/certbot \
  -d devsskills.com -d www.devsskills.com
```

এরপর certificate path নিজে হাতে config-এ বসাতে হবে (নিচের section 14 দেখুন)।

---

### পদ্ধতি ৩ — `certbot certonly --standalone`

Certbot নিজেই সাময়িকভাবে port 80-এ একটি ছোট web server চালায়।

```bash
sudo systemctl stop nginx           # ⚠️ Port 80 খালি করতে হবে
sudo certbot certonly --standalone -d devsskills.com
sudo systemctl start nginx
```

> ⚠️ **সতর্কতা:** এই পদ্ধতিতে Nginx বন্ধ করতে হয়, তাই **live site কিছুক্ষণের জন্য down থাকবে**। Nginx এখনো install না হয়ে থাকলে বা প্রথমবার সেটআপের সময় এটি ব্যবহার করুন।

---

### পদ্ধতি ৪ — DNS-01 Challenge (Wildcard certificate)

`*.devsskills.com` এর মতো wildcard certificate **শুধু DNS challenge দিয়েই** পাওয়া যায়।

**Definition — Wildcard Certificate:** এমন একটি certificate যা মূল domain-এর **সব subdomain**-এর জন্য কাজ করে।

**Example 1 —** একটি cert দিয়েই `api.devsskills.com`, `admin.devsskills.com`, `blog.devsskills.com` — সব কভার হয়।
**Example 2 —** প্রতিটি নতুন client-কে `client1.myapp.com`, `client2.myapp.com` দিলে বারবার cert নিতে হয় না।

**Manual পদ্ধতি:**

```bash
sudo certbot certonly --manual --preferred-challenges dns \
  -d "*.devsskills.com" -d devsskills.com
```

Certbot একটি TXT value দেবে, সেটি domain panel-এ যোগ করতে হবে:

| Type | Host | Value |
|------|------|-------|
| TXT | `_acme-challenge` | Certbot-এর দেওয়া string |

**Automatic পদ্ধতি (Cloudflare DNS ব্যবহার করলে):**

```bash
sudo apt install python3-certbot-dns-cloudflare -y

sudo mkdir -p /root/.secrets
sudo nano /root/.secrets/cloudflare.ini
```

```ini
dns_cloudflare_api_token = your_cloudflare_api_token_here
```

```bash
sudo chmod 600 /root/.secrets/cloudflare.ini      # ✅ শুধু root পড়তে পারবে

sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
  -d "*.devsskills.com" -d devsskills.com
```

✅ Manual DNS পদ্ধতিতে **auto-renewal কাজ করে না** — প্রতি ৯০ দিনে হাতে করতে হয়। তাই wildcard cert-এর জন্য DNS plugin (automatic) ব্যবহার করাই বুদ্ধিমানের কাজ।

---

### পদ্ধতি ৫ — কেনা CA Certificate (Manual Installation)

কোম্পানি যদি DigiCert / Sectigo / GoDaddy থেকে certificate কেনে, তাহলে ধাপগুলো:

**ধাপ ১ — Private key ও CSR তৈরি**

```bash
sudo mkdir -p /etc/nginx/ssl
cd /etc/nginx/ssl

sudo openssl req -new -newkey rsa:2048 -nodes \
  -keyout devsskills.key -out devsskills.csr
```

**Definition — CSR (Certificate Signing Request):** একটি file যেখানে আপনার domain ও প্রতিষ্ঠানের তথ্য থাকে। এটি CA-কে দিলে তারা certificate বানিয়ে দেয়।

**Example 1 —** `devsskills.csr` — CA-এর website-এ paste করতে হয়।
**Example 2 —** CSR তৈরির সময় "Common Name" এ অবশ্যই domain লিখতে হবে (`devsskills.com`)।

**ধাপ ২ — CA থেকে পাওয়া file গুলো একত্র করা (chain বানানো)**

```bash
# নিজের certificate আগে, intermediate/CA bundle পরে
cat devsskills.crt intermediate.crt > devsskills-bundle.crt
```

✅ **ক্রম উল্টে গেলে** browser-এ "certificate chain incomplete" error আসবে — মোবাইল browser-এ site খুলবেই না, যদিও desktop-এ ঠিক দেখাতে পারে।

**ধাপ ৩ — Nginx config-এ যোগ করা**

```nginx
server {
    listen 443 ssl;
    server_name devsskills.com;

    ssl_certificate     /etc/nginx/ssl/devsskills-bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/devsskills.key;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**ধাপ ৪ — Key file নিরাপদ করা**

```bash
sudo chmod 600 /etc/nginx/ssl/devsskills.key
sudo chown root:root /etc/nginx/ssl/devsskills.key
sudo nginx -t
sudo systemctl reload nginx
```

---

### পদ্ধতি ৬ — Self-Signed Certificate (শুধু development)

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt
```

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate     /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;

    root /var/www/test;
    index index.html;
}
```

> ⚠️ Self-signed certificate কোনো CA sign করে না, তাই browser **"Your connection is not private"** warning দেখাবে। ✅ এটি **কখনোই production-এ ব্যবহার করা যাবে না** — শুধু local testing বা internal tool-এর জন্য।

---

### পদ্ধতি ৭ — Cloudflare Origin Certificate

Site যদি Cloudflare-এর মধ্য দিয়ে যায়, তাহলে Cloudflare নিজেই ব্যবহারকারীকে SSL দেয়। কিন্তু Cloudflare থেকে আপনার server পর্যন্ত অংশটিও encrypt করা দরকার — সেজন্য **Origin Certificate**।

**Cloudflare-এর SSL mode:**

| Mode | মানে | মন্তব্য |
|------|------|---------|
| **Off** | কোনো SSL নেই | ব্যবহার করবেন না |
| **Flexible** | User→Cloudflare encrypted, Cloudflare→Server **plain HTTP** | ⚠️ অর্ধেক নিরাপদ; redirect loop-এর প্রধান কারণ |
| **Full** | দুই দিকেই encrypted, কিন্তু cert যাচাই হয় না | Self-signed হলেও চলে |
| **Full (Strict)** | ✅ দুই দিকেই encrypted এবং cert যাচাই হয় | **এটিই ব্যবহার করা উচিত** |

**ধাপ:** Cloudflare Dashboard → SSL/TLS → Origin Server → Create Certificate → পাওয়া `.pem` ও `.key` server-এ রাখুন:

```bash
sudo mkdir -p /etc/nginx/ssl
sudo nano /etc/nginx/ssl/cloudflare-origin.pem
sudo nano /etc/nginx/ssl/cloudflare-origin.key
sudo chmod 600 /etc/nginx/ssl/cloudflare-origin.key
```

```nginx
server {
    listen 443 ssl;
    server_name devsskills.com;

    ssl_certificate     /etc/nginx/ssl/cloudflare-origin.pem;
    ssl_certificate_key /etc/nginx/ssl/cloudflare-origin.key;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

✅ Cloudflare Origin Certificate **১৫ বছর পর্যন্ত** valid থাকে, তাই বারবার renew করার ঝামেলা নেই। তবে এটি **শুধু Cloudflare-এর মাধ্যমে আসা traffic-এর জন্যই** কাজ করে — সরাসরি IP-তে গেলে browser warning দেবে।

> ⚠️ **Flexible mode + Nginx-এ HTTP→HTTPS redirect** = **ERR_TOO_MANY_REDIRECTS**। সমাধান: mode কে **Full (Strict)** করুন।

---

### পদ্ধতি ৮ — `acme.sh` (Certbot-এর বিকল্প)

```bash
curl https://get.acme.sh | sh -s email=you@example.com
source ~/.bashrc

~/.acme.sh/acme.sh --issue -d devsskills.com -w /var/www/html

~/.acme.sh/acme.sh --install-cert -d devsskills.com \
  --key-file       /etc/nginx/ssl/devsskills.key \
  --fullchain-file /etc/nginx/ssl/devsskills.crt \
  --reloadcmd      "sudo systemctl reload nginx"
```

✅ `acme.sh` pure shell script — Python dependency লাগে না, এবং ZeroSSL/Let's Encrypt দুটোই সাপোর্ট করে।

---

## 13. SSL Auto-Renewal

Let's Encrypt certificate মাত্র **৯০ দিন** valid থাকে। Certbot install করলে Ubuntu-তে একটি **systemd timer** নিজে থেকেই তৈরি হয়ে যায়।

```bash
# Timer চালু আছে কিনা দেখা
systemctl status certbot.timer
systemctl list-timers | grep certbot

# Renewal সত্যিই কাজ করবে কিনা — আসল renew না করে পরীক্ষা
sudo certbot renew --dry-run

# বর্তমান certificate গুলোর মেয়াদ দেখা
sudo certbot certificates

# হাতে renew করতে হলে
sudo certbot renew
```

✅ `--dry-run` command টি **কোনো আসল certificate নেয় না**, শুধু পুরো প্রক্রিয়া পরীক্ষা করে। SSL সেটআপের পর অবশ্যই একবার চালিয়ে দেখা উচিত — নইলে ৯০ দিন পর হঠাৎ site down হয়ে যেতে পারে।

**Renew হওয়ার পর Nginx reload করার hook:**

```bash
sudo nano /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
```

```bash
#!/bin/bash
systemctl reload nginx
```

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
```

---

## 14. Production-ready SSL Config (Hardening)

```nginx
# HTTP → HTTPS redirect করার জন্য আলাদা block
server {
    listen 80;
    server_name devsskills.com www.devsskills.com;

    # Certbot-এর renewal যেন redirect-এ আটকে না যায়
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    http2 on;
    server_name devsskills.com www.devsskills.com;

    ssl_certificate     /etc/letsencrypt/live/devsskills.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/devsskills.com/privkey.pem;

    # শুধু আধুনিক ও নিরাপদ protocol
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    # Session cache — বারবার handshake না করে site দ্রুত হয়
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # OCSP stapling — certificate যাচাই দ্রুত করে
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 1.1.1.1 valid=300s;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    client_max_body_size 20M;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

| Directive | কাজ |
|-----------|-----|
| `http2 on;` | HTTP/2 চালু করে — একসাথে অনেক file দ্রুত load হয় |
| `ssl_protocols TLSv1.2 TLSv1.3;` | ✅ পুরনো ও দুর্বল TLS 1.0/1.1 বন্ধ করে দেয় |
| `ssl_session_cache shared:SSL:10m;` | ১০MB মেমোরিতে SSL session জমা রাখে (~40,000 session) |
| `ssl_session_tickets off;` | Forward secrecy আরও শক্ত করে |
| `ssl_stapling on;` | Certificate বাতিল হয়েছে কিনা তা server নিজেই জানিয়ে দেয় |
| `resolver` | Stapling-এর জন্য DNS server লাগে |
| `Strict-Transport-Security` (HSTS) | ✅ Browser-কে বলে "এই site সবসময় HTTPS দিয়েই খুলবে" |
| `X-Content-Type-Options: nosniff` | Browser যেন file type নিজে অনুমান না করে |
| `X-Frame-Options: SAMEORIGIN` | অন্য কেউ iframe-এ আপনার site বসাতে পারবে না (clickjacking রোধ) |
| `always` | Error response (404, 500)-এও header পাঠাবে |

> ⚠️ **HSTS নিয়ে সতর্কতা:** একবার HSTS পাঠালে browser নির্দিষ্ট সময় পর্যন্ত ওই domain HTTP-তে খুলতেই দেবে না। SSL ঠিকমতো কাজ করছে **নিশ্চিত হওয়ার পরেই** এটি চালু করবেন। প্রথমে `max-age=300` দিয়ে পরীক্ষা করে তারপর বাড়ানো নিরাপদ।

**যাচাই করার command:**

```bash
sudo nginx -t
sudo systemctl reload nginx

curl -I https://devsskills.com
openssl s_client -connect devsskills.com:443 -servername devsskills.com
```

Online test: **SSL Labs** (`https://www.ssllabs.com/ssltest/`) — grade **A** বা **A+** আসা উচিত।

---

## 15. সাধারণ Error ও তার সমাধান

| Error | কারণ | সমাধান |
|-------|------|--------|
| **502 Bad Gateway** | Backend app চলছে না, বা port ভুল | `pm2 list`, `curl http://127.0.0.1:5000` দিয়ে check করুন |
| **403 Forbidden** | Folder-এর permission নেই | `sudo chown -R www-data:www-data /var/www/site` |
| **404 Not Found** (SPA refresh-এ) | `try_files` ভুল | `try_files $uri $uri/ /index.html;` দিন |
| **413 Request Entity Too Large** | Upload size limit কম | `client_max_body_size 20M;` যোগ করুন |
| **504 Gateway Timeout** | Backend উত্তর দিতে দেরি করছে | `proxy_read_timeout 120s;` বাড়ান |
| **"Welcome to nginx!" দেখাচ্ছে** | Default site এখনো active | `sudo rm /etc/nginx/sites-enabled/default` |
| **ERR_TOO_MANY_REDIRECTS** | Cloudflare Flexible mode | Cloudflare-এ **Full (Strict)** করুন |
| **Certbot: Timeout during connect** | Port 80 বন্ধ বা DNS ভুল | Inbound rule ও `dig +short domain.com` check করুন |
| **`nginx: [emerg] bind() to 0.0.0.0:80 failed`** | অন্য কোনো service port 80 দখল করে আছে | `sudo lsof -i :80` দিয়ে খুঁজে বন্ধ করুন |
| **`conflicting server name`** | একই `server_name` দুই file-এ আছে | ডুপ্লিকেট `.conf` file মুছুন |
| **Site পুরনো version দেখাচ্ছে** | Browser cache | Hard refresh (`Ctrl+Shift+R`) বা `curl -I` দিয়ে check |

**সমস্যা হলে সবার আগে যা দেখবেন:**

```bash
sudo nginx -t                              # Config-এ ভুল আছে কিনা
sudo tail -f /var/log/nginx/error.log      # ✅ আসল কারণ প্রায় সবসময় এখানে থাকে
systemctl status nginx
pm2 logs
```

---

## এক নজরে মূল পয়েন্ট

| # | পয়েন্ট |
|---|--------|
| ✅ 1 | SSH (port **22**) দিয়ে server-এ নিরাপদে connect করা হয়; **SFTP-ও একই port 22** ব্যবহার করে |
| ✅ 2 | HTTP = **80**, HTTPS = **443** — দুটোই inbound rules-এ open করতে হবে |
| ✅ 3 | Nginx একই সাথে **Web Server** ও **Reverse Proxy** |
| ✅ 4 | Config বদলানোর পর সবসময় **`sudo nginx -t` → তারপর `sudo systemctl reload nginx`** |
| ✅ 5 | `reload` করলে **downtime হয় না**, `restart` করলে হয় |
| ✅ 6 | `/var/www/` এ কাজ করতে `sudo` লাগে, `/home/ubuntu` এ লাগে না |
| ✅ 7 | Node app **PM2** দিয়ে চালাতে হয়; `pm2 startup` + `pm2 save` না দিলে reboot-এ বন্ধ হয়ে যায় |
| ✅ 8 | Reverse proxy-র মূল directive **`proxy_pass`** |
| ✅ 9 | `proxy_set_header Host / X-Real-IP / X-Forwarded-Proto` অবশ্যই দিতে হবে |
| ✅ 10 | `proxy_pass`-এর **শেষে `/` আছে কি নেই** — এতে backend-এ path বদলে যায় |
| ✅ 11 | একাধিক app-এর জন্য **প্রতিটি domain-এর আলাদা `.conf` file** বানানোই সবচেয়ে পরিষ্কার |
| ✅ 12 | React/SPA-এর জন্য **`try_files $uri $uri/ /index.html;`** বাধ্যতামূলক |
| ✅ 13 | DNS: `@` → **A record** (IP), `www` → **CNAME** (Canonical Name) |
| ✅ 14 | SSL-এর আগে **DNS ঠিক থাকতে হবে + port 80 খোলা থাকতে হবে** |
| ✅ 15 | সাধারণ ক্ষেত্রে **`sudo certbot --nginx`**, live site-এ downtime এড়াতে **`--webroot`** |
| ✅ 16 | **Wildcard cert** শুধু **DNS-01 challenge** দিয়েই পাওয়া যায় |
| ✅ 17 | Let's Encrypt cert **৯০ দিন** valid; `sudo certbot renew --dry-run` দিয়ে renewal যাচাই করুন |
| ✅ 18 | **Self-signed cert production-এ নয়**; Cloudflare ব্যবহার করলে **Full (Strict)** mode দিন |
| ✅ 19 | `ssl_certificate_key` (private key) **কখনোই share বা git-এ commit করা যাবে না** |
| ✅ 20 | সমস্যা হলে সবার আগে **`/var/log/nginx/error.log`** দেখুন |

---

## দ্রুত রেফারেন্স — Command Cheat Sheet

```bash
# ---------- Nginx ----------
sudo apt install nginx -y
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
sudo systemctl enable nginx
systemctl status nginx

# ---------- Logs ----------
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log

# ---------- Firewall ----------
sudo ufw allow 'Nginx Full'
sudo ufw status

# ---------- PM2 ----------
pm2 start index.js --name my-api
pm2 list
pm2 logs my-api
pm2 restart my-api
pm2 startup && pm2 save

# ---------- SSL ----------
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d domain.com -d www.domain.com
sudo certbot certonly --webroot -w /var/www/certbot -d domain.com
sudo certbot certificates
sudo certbot renew --dry-run

# ---------- DNS check ----------
dig +short domain.com
curl -I https://domain.com
```