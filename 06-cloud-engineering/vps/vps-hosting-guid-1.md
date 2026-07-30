# VPS + Ubuntu + Nginx + Cloudflare দিয়ে Multiple Site Hosting — সম্পূর্ণ গাইড

> এই ডকুমেন্টটা তোমার personal reference। উপর থেকে নিচে পড়লে একটা পূর্ণ ছবি পাবে,
> আবার পরে দরকার অনুযায়ী যেকোনো section আলাদা করে দেখতে পারবে।
> Command গুলো English-এ রাখা হয়েছে যাতে copy-paste করা যায়।

---

## সূচিপত্র

0. শুরুর আগে — কিছু মৌলিক ধারণা
1. পুরনো সব কিছু Clean করা (Fresh Start)
2. VPS নেওয়া (Contabo বা অন্য company)
3. SSH দিয়ে server-এ Login করা
4. Initial Server Setup ও Security
5. Nginx Install ও মূল ধারণা
6. Site Deploy করার ৪টা পদ্ধতি
7. Cloudflare দিয়ে Domain / Subdomain যুক্ত করা
8. HTTPS / SSL যোগ করা
9. দৈনন্দিন Deploy Workflow ও দরকারি Command
10. সমস্যা হলে কোথায় দেখবে (Troubleshooting)

---

## 0. শুরুর আগে — কিছু মৌলিক ধারণা

কয়েকটা শব্দ বারবার আসবে, তাই আগে পরিষ্কার করে নিই:

- **VPS (Virtual Private Server):** একটা পুরো computer যেটা কোনো datacenter-এ চলছে,
  আর তুমি internet দিয়ে সেটা ব্যবহার করো। Contabo, DigitalOcean, Linode এসব কোম্পানি এটা ভাড়া দেয়।
- **IP address:** তোমার server-এর ঠিকানা, যেমন `95.111.240.50`। এটা দিয়েই বাইরের জগৎ তোমার server খুঁজে পায়।
- **SSH:** নিরাপদভাবে দূর থেকে server-এর terminal-এ ঢুকে command চালানোর পদ্ধতি।
- **Nginx:** web server software — browser থেকে request এলে সঠিক site-এর file/response ফেরত পাঠায়।
- **Domain / DNS:** `example.com` এর মতো নাম। DNS হলো সেই system যেটা এই নামকে IP-তে অনুবাদ করে।
- **Cloudflare:** DNS + security + CDN service। তোমার domain-এর নাম আর IP-এর সংযোগ এখানে ঠিক করে দেওয়া হয়।

পুরো chain-টা এমন:

```
Browser → domain লেখে → Cloudflare (DNS) IP জানায় → সেই IP-তে থাকা Nginx → সঠিক site serve করে
```

---

## 1. পুরনো সব কিছু Clean করা (Fresh Start)

তুমি আগে না বুঝে অনেক কিছু করেছ, এখন পরিষ্কার করে শুরু করতে চাও। **দুইটা রাস্তা** আছে।

### রাস্তা A — পুরো OS নতুন করে বসানো (সবচেয়ে পরিষ্কার) ✅ সুপারিশকৃত

যেহেতু site গুলো শুধু learning-এর জন্য ছিল, সবচেয়ে সহজ আর নিশ্চিত উপায় হলো Contabo panel থেকে
পুরো Ubuntu নতুন করে reinstall করা। এতে সব কিছু মুছে একদম fresh Ubuntu পাবে।

Contabo-তে ধাপগুলো:

1. Contabo customer panel-এ login করো।
2. তোমার VPS select করো → **"Reinstall"** বা **"Reset"** option খোঁজো।
3. OS হিসেবে **Ubuntu 24.04 LTS** (বা latest LTS) বেছে নাও।
4. Confirm করলে ১০-১৫ মিনিটে নতুন Ubuntu বসে যাবে, নতুন root password দেবে।

> ⚠️ Reinstall করলে server-এর সব data মুছে যায়। কোনো দরকারি file থাকলে আগে ব্যাকআপ নাও।
> Learning site হলে চিন্তা নেই।

### রাস্তা B — OS না মুছে হাতে হাতে পরিষ্কার করা

যদি OS রাখতে চাও কিন্তু শুধু আগের site/config মুছতে চাও:

```bash
# ১) Nginx-এ কোন কোন site চালু আছে দেখো
ls -l /etc/nginx/sites-enabled/
ls -l /etc/nginx/sites-available/

# ২) enabled site-গুলোর link মুছে দাও (এতে মূল file মুছবে না, শুধু বন্ধ হবে)
sudo rm /etc/nginx/sites-enabled/*

# ৩) না-লাগলে sites-available থেকেও config মুছে দাও (default রেখে দিতে পারো)
sudo rm /etc/nginx/sites-available/your-old-site

# ৪) site-এর file গুলো কোথায় আছে দেখে মুছে দাও (সাধারণত /var/www তে)
ls -l /var/www/
sudo rm -rf /var/www/your-old-site

# ৫) background-এ কোনো app (Node/Python) চলছে কিনা দেখো
sudo systemctl list-units --type=service | grep -iE 'node|pm2|gunicorn|python'
# pm2 দিয়ে চালিয়ে থাকলে:
pm2 list
pm2 delete all

# ৬) config ঠিক আছে কিনা check করে reload
sudo nginx -t
sudo systemctl reload nginx
```

Cloudflare-এর দিকেও পরিষ্কার করতে হবে (নিচে Part 7-এ বিস্তারিত):
পুরনো A/CNAME record গুলো Cloudflare dashboard-এর **DNS** সেকশন থেকে মুছে দাও যেগুলো আর দরকার নেই।

> আমার সুপারিশ: learning purpose যেহেতু, **রাস্তা A (reinstall)** নাও। একদম clean slate পাবে,
> আর নিচের গাইড হুবহু step-by-step follow করতে পারবে কোনো পুরনো config-এর ঝামেলা ছাড়াই।

---

## 2. VPS নেওয়া (Contabo বা অন্য company)

মূল ধারণা সব কোম্পানিতেই এক। ধাপগুলো:

1. কোম্পানির website-এ গিয়ে একটা VPS plan order করো (RAM/CPU/storage অনুযায়ী দাম)।
2. OS হিসেবে **Ubuntu (latest LTS, যেমন 24.04)** বেছে নাও।
3. Order complete হলে email/panel-এ তিনটা জিনিস পাবে:
   - **Server IP address** (যেমন `95.111.240.50`)
   - **Username** (সাধারণত `root`)
   - **Password** (root password)

এই তিনটা জিনিস দিয়েই তুমি server-এ ঢুকবে।

> Contabo-তে server ready হতে মাঝে মাঝে কিছুটা সময় লাগে। Panel-এ status "running/active" দেখালে তৈরি।

---

## 3. SSH দিয়ে server-এ Login করা

তোমার নিজের computer-এর terminal থেকে (Linux/macOS-এ Terminal, Windows-এ PowerShell বা Git Bash):

```bash
ssh root@YOUR_SERVER_IP
```

উদাহরণ:

```bash
ssh root@95.111.240.50
```

প্রথমবার একটা fingerprint confirm করতে বলবে — `yes` লিখো। তারপর root password চাইবে
(টাইপ করার সময় কিছু দেখাবে না, এটা normal)। ঢুকে গেলে terminal-এ server-এর prompt দেখতে পাবে।

### SSH Key দিয়ে login (password ছাড়া — অনেক নিরাপদ ও সুবিধাজনক)

বারবার password টাইপ করা এড়াতে আর security বাড়াতে SSH key ব্যবহার করা ভালো।

তোমার **নিজের computer-এ** (server-এ নয়) একটা key বানাও:

```bash
ssh-keygen -t ed25519 -C "amar-laptop"
```

Enter চাপতে থাকলে default জায়গায় (`~/.ssh/id_ed25519`) key তৈরি হবে।
এবার সেই public key server-এ পাঠাও:

```bash
ssh-copy-id root@YOUR_SERVER_IP
```

এরপর থেকে `ssh root@YOUR_SERVER_IP` দিলে আর password লাগবে না।

---

## 4. Initial Server Setup ও Security

Fresh server-এ ঢুকে প্রথমেই এই কাজগুলো করা ভালো অভ্যাস।

### ৪.১ System update

```bash
sudo apt update && sudo apt upgrade -y
```

### ৪.২ root-এর বদলে একটা normal user বানানো

সবসময় `root` দিয়ে কাজ করা ঝুঁকিপূর্ণ। একটা sudo-user বানাও:

```bash
adduser deploy            # 'deploy' নামে user (নাম যা খুশি দিতে পারো)
usermod -aG sudo deploy    # তাকে sudo ক্ষমতা দাও
```

এবার নতুন user দিয়ে SSH key setup করতে চাইলে:

```bash
# root হিসেবে থাকা অবস্থায়
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy
```

এরপর থেকে `ssh deploy@YOUR_SERVER_IP` দিয়ে ঢুকবে, আর দরকারে `sudo` ব্যবহার করবে।

### ৪.৩ Firewall চালু করা (UFW)

```bash
sudo ufw allow OpenSSH      # SSH বন্ধ হয়ে গেলে ঢুকতে পারবে না, তাই আগে এটা
sudo ufw allow 'Nginx Full' # port 80 (HTTP) আর 443 (HTTPS) খুলে দেয়
sudo ufw enable
sudo ufw status             # কী কী খোলা আছে দেখো
```

> ⚠️ `ufw enable` করার আগে অবশ্যই `allow OpenSSH` দিয়ে রাখো, নইলে নিজেই নিজের server থেকে বেরিয়ে গিয়ে আটকে যাবে।

---

## 5. Nginx Install ও মূল ধারণা

### Install

```bash
sudo apt install nginx -y
sudo systemctl status nginx      # active (running) দেখলে ঠিক আছে
```

Browser-এ `http://YOUR_SERVER_IP` লিখলে Nginx-এর default "Welcome" page দেখবে।

### দরকারি Command (মুখস্থ রাখো)

```bash
sudo systemctl start nginx      # চালু
sudo systemctl stop nginx       # বন্ধ
sudo systemctl restart nginx    # বন্ধ করে আবার চালু
sudo systemctl reload nginx     # config নতুন করে পড়ে, downtime ছাড়া
sudo nginx -t                   # config-এ ভুল আছে কিনা check
```

> **সোনালি নিয়ম:** config বদলানোর পর সবসময় আগে `sudo nginx -t`, তারপর `sudo systemctl reload nginx`।

### Config কোথায় থাকে?

```
/etc/nginx/
├── nginx.conf                 # মূল config (সাধারণত হাত দিতে হয় না)
├── sites-available/           # এখানে প্রতিটা site-এর config লেখো
│   └── default                # default site
└── sites-enabled/             # চালু করা site-এর symlink এখানে থাকে
```

**Workflow:** `sites-available/`-এ config বানাও → `sites-enabled/`-এ link করে "enable" করো → reload।

Enable করার command:

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
```

### Server Block কী?

প্রতিটা site একটা `server { ... }` block দিয়ে define হয়। এটাই Nginx-এর মূল building block।
কয়েকটা গুরুত্বপূর্ণ অংশ:

```nginx
server {
    listen 80;                      # কোন port-এ শুনবে (80 = HTTP)
    server_name example.com;        # কোন domain-এর জন্য এই block
    root /var/www/example;          # site-এর file কোথায়
    index index.html;               # default কোন file দেখাবে

    location / {                    # request-এর path অনুযায়ী নিয়ম
        try_files $uri $uri/ =404;
    }
}
```

Nginx আসা প্রতিটা request-এর `server_name` মিলিয়ে ঠিক করে কোন `server` block ব্যবহার করবে।
এভাবেই একই server-এ একই IP দিয়ে অনেকগুলো আলাদা site চালানো যায় — এটাকে বলে **name-based virtual hosting**।

---

## 6. Site Deploy করার ৪টা পদ্ধতি

তুমি নানাভাবে শিখতে চেয়েছ, তাই চারটা scenario আলাদা করে দিলাম।

### পদ্ধতি ১ — আলাদা Domain-এ আলাদা Site

ধরো দুইটা site: `site-one.com` আর `site-two.com`।

file রাখো আলাদা folder-এ:

```bash
sudo mkdir -p /var/www/site-one
sudo mkdir -p /var/www/site-two
echo "<h1>Site One</h1>" | sudo tee /var/www/site-one/index.html
echo "<h1>Site Two</h1>" | sudo tee /var/www/site-two/index.html
```

**Site One-এর config** (`/etc/nginx/sites-available/site-one`):

```nginx
server {
    listen 80;
    server_name site-one.com www.site-one.com;
    root /var/www/site-one;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Site Two-এর config** (`/etc/nginx/sites-available/site-two`):

```nginx
server {
    listen 80;
    server_name site-two.com www.site-two.com;
    root /var/www/site-two;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

দুইটাই enable করে reload:

```bash
sudo ln -s /etc/nginx/sites-available/site-one /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/site-two /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

দুইটা domain-ই Cloudflare-এ একই IP-তে point করা থাকবে (Part 7)। Nginx `server_name` দেখে
সঠিক site দেখাবে।

---

### পদ্ধতি ২ — Subdomain দিয়ে Site

Subdomain মানে `blog.example.com`, `app.example.com` এরকম। Nginx-এর দিক থেকে এটা প্রায় হুবহু
পদ্ধতি ১-এর মতো — শুধু `server_name`-এ subdomain লিখবে।

```bash
sudo mkdir -p /var/www/blog
echo "<h1>My Blog</h1>" | sudo tee /var/www/blog/index.html
```

`/etc/nginx/sites-available/blog`:

```nginx
server {
    listen 80;
    server_name blog.example.com;   # <-- subdomain
    root /var/www/blog;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/blog /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

Cloudflare-এ subdomain-এর জন্য আলাদা একটা DNS record বানাতে হবে (Part 7 দেখো)।

---

### পদ্ধতি ৩ — শুধু IP দিয়ে সরাসরি একটা Site দেখানো

কোনো domain ছাড়াই `http://YOUR_SERVER_IP` লিখলে যে site দেখাবে, সেটা ঠিক করা হয় `default_server`
দিয়ে। যে `server` block-এ `default_server` লেখা থাকে, domain না মিললে Nginx সেটাই দেখায়।

```bash
sudo mkdir -p /var/www/default-site
echo "<h1>IP দিয়ে আসা Default Site</h1>" | sudo tee /var/www/default-site/index.html
```

`/etc/nginx/sites-available/default-site`:

```nginx
server {
    listen 80 default_server;       # <-- এই block-টাই default
    server_name _;                  # _ মানে "যেকোনো নাম / নাম নেই"
    root /var/www/default-site;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

> ⚠️ পুরো Nginx-এ **শুধু একটা** block-এ `default_server` থাকতে পারে port 80-এর জন্য।
> তাই এটা enable করার আগে পুরনো `default` config disable করে দাও:
> `sudo rm /etc/nginx/sites-enabled/default`

```bash
sudo ln -s /etc/nginx/sites-available/default-site /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

### পদ্ধতি ৪ — একই IP-তে Path/Directory দিয়ে আলাদা Project

মানে `http://YOUR_SERVER_IP/project-a` আর `http://YOUR_SERVER_IP/project-b` — একই IP, কিন্তু path
আলাদা। এটা করা হয় একাধিক `location` block দিয়ে, অথবা `alias` ব্যবহার করে।

```bash
sudo mkdir -p /var/www/projects/project-a
sudo mkdir -p /var/www/projects/project-b
echo "<h1>Project A</h1>" | sudo tee /var/www/projects/project-a/index.html
echo "<h1>Project B</h1>" | sudo tee /var/www/projects/project-b/index.html
```

Config — এটা **পদ্ধতি ৩-এর সেই একই `default_server` block**, শুধু ভেতরে `location` যোগ করা হয়েছে।
আলাদা করে নতুন `default_server` block বানিও না (তাতে conflict হবে, নিচের নোট দেখো):

```nginx
server {
    listen 80 default_server;
    server_name _;
    root /var/www/projects;          # <-- parent folder-কে root বানানো হলো

    location /project-a/ {
        try_files $uri $uri/ =404;
    }

    location /project-b/ {
        try_files $uri $uri/ =404;
    }
}
```

এখানে `root` হলো `/var/www/projects`, তাই `/project-a/` request এলে Nginx নিজে থেকেই
`/var/www/projects/project-a/` folder থেকে file দেয়। এবার `http://YOUR_SERVER_IP/project-a/`
আর `.../project-b/` আলাদা site দেখাবে।

> ⚠️ এই block-এও `default_server` আছে, আর পদ্ধতি ৩-এও ছিল। **দুটো একসাথে enable করলে
> Nginx `duplicate default_server` error দেবে।** তাই path-based site চাইলে পদ্ধতি ৩-এর block-এর
> ভেতরেই এই `location` গুলো যোগ করো — নতুন block বানিও না।
>
> **কেন `alias` নয়, `root`?** `alias`-এর সাথে `try_files $uri` মেশালে Nginx-এ পুরনো একটা bug
> আছে যা path ভুল resolve করে অপ্রত্যাশিত 404 দেয়। তাই এখানে `root` ব্যবহার করাই নিরাপদ ও
> সঠিক পদ্ধতি — folder structure যদি path-এর সাথে মিলে যায় (যেমন উপরে), `root` সবচেয়ে পরিষ্কার সমাধান।

---

### Dynamic App হলে (Node.js / Python) — Reverse Proxy

উপরের সব উদাহরণ static file-এর। কিন্তু Node.js বা Python app যদি background-এ কোনো port-এ চলে
(ধরো port 3000), তাহলে Nginx-কে "reverse proxy" বানাতে হয়:

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://localhost:3000;      # <-- app-এর কাছে পাঠাও
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache_bypass $http_upgrade;
    }
}
```

App-টা সবসময় চালু রাখতে **PM2** (Node) বা **systemd/gunicorn** (Python) ব্যবহার করো, যাতে server
restart হলেও app আবার চালু হয়। Node-এর জন্য দ্রুত উদাহরণ:

```bash
sudo apt install -y nodejs npm
sudo npm install -g pm2
pm2 start app.js         # তোমার app চালু করো
pm2 startup              # boot-এ auto-start setup
pm2 save
```

---

## 7. Cloudflare দিয়ে Domain / Subdomain যুক্ত করা

Cloudflare-এর কাজ হলো: তোমার domain লিখলে browser যেন তোমার server-এর IP খুঁজে পায়।

### একবারের setup

1. Cloudflare account-এ domain add করা থাকলে, **Registrar** (যেখান থেকে domain কিনেছ) এর
   nameserver Cloudflare-এর দেওয়া nameserver-এ বদলাতে হয় (একবারই)।
2. Cloudflare dashboard → তোমার domain → **DNS** সেকশনে যাও।

### DNS Record যোগ করা

**মূল domain-এর জন্য** (example.com):

| Type | Name | Content (IPv4 address) | Proxy status |
|------|------|------------------------|--------------|
| A    | @    | YOUR_SERVER_IP         | Proxied 🟠   |

**www-এর জন্য:**

| Type  | Name | Content        | Proxy   |
|-------|------|----------------|---------|
| CNAME | www  | example.com    | Proxied |

**Subdomain-এর জন্য** (blog.example.com):

| Type | Name | Content        | Proxy   |
|------|------|----------------|---------|
| A    | blog | YOUR_SERVER_IP | Proxied |

- `@` মানে মূল domain নিজে।
- **Proxy status (কমলা মেঘ 🟠):** on থাকলে traffic Cloudflare হয়ে যায় (DDoS protection, caching,
  আসল IP লুকানো)। শুরুতে on রাখলেই ভালো। শুধু SSL debug করার সময় সাময়িকভাবে off (grey) করা লাগতে পারে।

### পুরনো record মোছা

আগে না বুঝে বানানো অপ্রয়োজনীয় record গুলো এই DNS সেকশন থেকেই **Delete** করে দাও।

> DNS পরিবর্তন কার্যকর হতে কয়েক মিনিট থেকে কিছু ঘণ্টা লাগতে পারে (propagation)।
> `https://dnschecker.org`-এ গিয়ে দেখতে পারো record ছড়িয়েছে কিনা।

---

## 8. HTTPS / SSL যোগ করা (site-কে `https://` করা)

দুইটা প্রচলিত রাস্তা।

### রাস্তা A — Let's Encrypt + Certbot (free, স্ট্যান্ডার্ড)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot নিজেই certificate এনে Nginx config বদলে HTTPS চালু করে দেবে, আর auto-renew setup করে দেবে।
Subdomain-এর জন্যও একইভাবে `-d blog.example.com` দিয়ে চালাও।

> ⚠️ Cloudflare proxy (🟠) on থাকলে Certbot-এর HTTP validation মাঝে মাঝে আটকায়।
> তখন হয় সাময়িকভাবে proxy grey (off) করো, নয়তো নিচের রাস্তা B ব্যবহার করো।

### রাস্তা B — Cloudflare Origin Certificate (Cloudflare ব্যবহারকারীদের জন্য সহজ)

যেহেতু তুমি Cloudflare ব্যবহার করছ:

1. Cloudflare dashboard → **SSL/TLS** → **Origin Server** → **Create Certificate**।
2. তৈরি হওয়া certificate আর private key দুইটা server-এ file হিসেবে save করো
   (যেমন `/etc/ssl/cloudflare/`)।
3. Nginx config-এ HTTPS block যোগ করো:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/ssl/cloudflare/cert.pem;
    ssl_certificate_key /etc/ssl/cloudflare/key.pem;

    root /var/www/example;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

4. Cloudflare-এ **SSL/TLS mode** সেট করো **Full (strict)**।

---

## 9. দৈনন্দিন Deploy Workflow ও দরকারি Command

একটা নতুন static site তোলার পুরো চক্র (মুখস্থ হয়ে গেলে ২ মিনিটের কাজ):

```bash
# ১) folder বানাও ও file রাখো
sudo mkdir -p /var/www/newsite
# (এখানে তোমার HTML/CSS file গুলো copy করো — scp বা git clone দিয়ে)

# ২) config বানাও
sudo nano /etc/nginx/sites-available/newsite

# ৩) enable করো
sudo ln -s /etc/nginx/sites-available/newsite /etc/nginx/sites-enabled/

# ৪) check ও reload
sudo nginx -t
sudo systemctl reload nginx

# ৫) Cloudflare-এ DNS record যোগ করো (যদি domain থাকে)
# ৬) SSL চালাও: sudo certbot --nginx -d newsite.com
```

### নিজের computer থেকে server-এ file পাঠানো

```bash
# scp দিয়ে একটা folder পুরো পাঠানো
scp -r ./my-site-folder deploy@YOUR_SERVER_IP:/tmp/my-site-folder

# server-এ ঢুকে সঠিক জায়গায় সরানো
ssh deploy@YOUR_SERVER_IP
sudo mv /tmp/my-site-folder /var/www/newsite
```

অথবা GitHub থেকে সরাসরি:

```bash
cd /var/www
sudo git clone https://github.com/username/my-site.git newsite
```

### দরকারি চটজলদি command

```bash
sudo nginx -t                            # config check
sudo systemctl reload nginx              # reload
sudo systemctl status nginx              # চলছে কিনা
ls /etc/nginx/sites-enabled/             # কোন কোন site চালু
sudo tail -f /var/log/nginx/error.log    # সমস্যা হলে live error দেখা
sudo tail -f /var/log/nginx/access.log   # কে কে request করছে
df -h                                    # disk কতটা ভরেছে
```

---

## 10. সমস্যা হলে কোথায় দেখবে (Troubleshooting)

| সমস্যা | সম্ভাব্য কারণ / কী দেখবে |
|--------|--------------------------|
| `nginx -t` fail করছে | config-এ typo বা ভুল path। error message পড়ো, line number বলে দেয়। |
| Site খুলছে না, timeout | Firewall-এ port খোলা নেই → `sudo ufw allow 'Nginx Full'`। |
| Domain কাজ করছে না, IP কাজ করছে | DNS record ভুল বা এখনো propagate হয়নি → Cloudflare DNS check করো। |
| 502 Bad Gateway | reverse proxy-তে backend app বন্ধ → `pm2 list` / app চালু আছে কিনা দেখো। |
| 403 Forbidden | file permission সমস্যা বা `index.html` নেই → folder ও permission দেখো। |
| ভুল site দেখাচ্ছে | `server_name` মিলছে না, তাই `default_server` দেখাচ্ছে → server_name ঠিক করো। |
| SSL error | Certbot fail বা Cloudflare SSL mode ভুল → Part 8 আবার দেখো। |

সবচেয়ে বড় troubleshooting অস্ত্র:

```bash
sudo tail -f /var/log/nginx/error.log
```

এটা চালু রেখে browser-এ site refresh দাও — কী ভুল হচ্ছে সাধারণত এখানেই লেখা থাকে।

---

## শেষ কথা

মনে রাখার মতো মূল ছবিটা:

```
Domain (Cloudflare DNS) → IP → Nginx (server_name মিলিয়ে) → সঠিক folder/app → response
```

- আলাদা site = আলাদা `server` block + আলাদা `server_name` + আলাদা `root`।
- Subdomain = শুধু `server_name`-এ subdomain + Cloudflare-এ নতুন record।
- শুধু IP = `default_server` block।
- IP + path = একটা `default_server` block-এর ভেতরে একাধিক `location` block + parent folder-কে `root`।

এই চারটা প্যাটার্ন বুঝে গেলে তুমি প্রায় যেকোনো hosting scenario নিজে সাজাতে পারবে। 🚀