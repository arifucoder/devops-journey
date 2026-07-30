# এক Domain-এর ৪ Subdomain-এ ২টা Next.js + ২টা Express Deploy — সম্পূর্ণ গাইড

> এই গাইডটা ধরে নিচ্ছে: তোমার একটা VPS আছে (Ubuntu), SSH দিয়ে ঢুকতে পারো, আর তোমার domain
> Cloudflare-এ আছে যেখানে `*` (wildcard) A record দিয়ে সব subdomain server-এর IP-তে point করানো।
> নিচে example হিসেবে domain ধরা হয়েছে `example.com` — তোমার নিজের domain দিয়ে বদলে নিও।

---

## আমরা কী বানাচ্ছি (Architecture)

একটা মাত্র server, একটা মাত্র IP। সামনে দাঁড়িয়ে **Nginx** সব request receive করে, আর subdomain
অনুযায়ী সঠিক app-এর কাছে পাঠিয়ে দেয় (reverse proxy)। প্রতিটা app আলাদা একটা port-এ চলে,
আর **PM2** সেগুলোকে সবসময় চালু রাখে।

```
                                  ┌─────────────────────────────┐
 web1.example.com  ───┐           │  Next.js App 1   (port 3000)│
 web2.example.com  ───┤           │  Next.js App 2   (port 3001)│
 api1.example.com  ───┤─► Nginx ─►│  Express API 1   (port 4000)│
 api2.example.com  ───┘  (reverse │  Express API 2   (port 4001)│
                          proxy)  └─────────────────────────────┘
```

| Subdomain          | ধরন        | Port | Folder          |
|--------------------|------------|------|-----------------|
| web1.example.com   | Next.js    | 3000 | /opt/web1   |
| web2.example.com   | Next.js    | 3001 | /opt/web2   |
| api1.example.com   | Express    | 4000 | /opt/api1   |
| api2.example.com   | Express    | 4001 | /opt/api2   |

> Port গুলো তুমি যা খুশি রাখতে পারো, শুধু প্রতিটা আলাদা হতে হবে আর কোনোটা 80/443 না হয় (ওগুলো Nginx-এর)।

---

## ধাপ ০ — Cloudflare DNS যাচাই

তোমার `*` wildcard record থাকায় `web1`, `web2`, `api1`, `api2` — সব automatically server-এর
IP-তে চলে আসবে। তবু নিশ্চিত হও:

Cloudflare dashboard → তোমার domain → **DNS** → এই record টা আছে কিনা দেখো:

| Type | Name | Content        | Proxy status |
|------|------|----------------|--------------|
| A    | `*`  | YOUR_SERVER_IP | Proxied 🟠   |

- চাইলে root domain-এর জন্যও একটা `A @ → YOUR_SERVER_IP` রাখতে পারো।
- Proxy 🟠 (কমলা) থাকলে SSL-এর জন্য আমরা নিচে Cloudflare Origin Certificate ব্যবহার করব।

> ভালো অভ্যাস: যদিও `*` সব কভার করে, চাইলে নির্দিষ্ট subdomain-এর জন্য আলাদা A record-ও
> বানাতে পারো (তখন সেটা wildcard-কে override করে)। শুরুতে `*` দিয়েই চলবে।

---

## ধাপ ১ — Server তৈরি করা (Node, PM2, Nginx)

SSH দিয়ে server-এ ঢোকো, তারপর:

```bash
# সিস্টেম update
sudo apt update && sudo apt upgrade -y

# Node.js LTS ইনস্টল (NodeSource থেকে, উদাহরণে Node 22)
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# ঠিকঠাক বসেছে কিনা check
node -v
npm -v

# PM2 (process manager) — app গুলো সবসময় চালু রাখবে
sudo npm install -g pm2

# Nginx (আগে ইনস্টল না থাকলে)
sudo apt install -y nginx

# Firewall — SSH আর Nginx-এর port খুলে দাও
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

> ⚠️ **কম RAM-এর VPS হলে (১-২ GB):** Next.js build করতে গিয়ে memory শেষ হয়ে যেতে পারে।
> তখন একটা swap file বানিয়ে নাও:
> ```bash
> sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
> sudo mkswap /swapfile && sudo swapon /swapfile
> echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
> ```

---

## ধাপ ২ — App-এর কোড server-এ আনা

প্রতিটা app-কে `/opt/`-এর ভেতরে আলাদা folder-এ রাখব। GitHub থেকে clone করাই সবচেয়ে সহজ।

> **কেন `/opt`, `/var/www` নয়?** Linux convention অনুযায়ী `/opt` হলো বাইরে থেকে আনা, নিজে
> চালানো application-এর জায়গা — আর তোমার চারটা app-ই (Next.js `next start` + Express) চলমান
> Node process। `/var/www` শুধু নিছক **static file**-এর জন্য, যেগুলো Nginx সরাসরি পড়ে পাঠায়
> (কোনো process চলে না)। নিয়মটা এক লাইনে: **চলমান process → `/opt`, static file → `/var/www`।**
> ভবিষ্যতে কোনো বিশুদ্ধ static site (`next export` বা শুধু HTML/CSS) যোগ করলে সেটা `/var/www`-এ রেখো।

```bash
# প্রতিটা app-এর folder বানিয়ে মালিকানা নিজের user-কে দাও (/opt সাধারণত root-এর, তাই দরকার)
sudo mkdir -p /opt/web1 /opt/web2 /opt/api1 /opt/api2
sudo chown -R $USER:$USER /opt/web1 /opt/web2 /opt/api1 /opt/api2
cd /opt

# প্রতিটা repo clone করো (তোমার আসল repo URL বসাও)
git clone https://github.com/you/nextjs-app-1.git web1
git clone https://github.com/you/nextjs-app-2.git web2
git clone https://github.com/you/express-api-1.git api1
git clone https://github.com/you/express-api-2.git api2
```

> Git না ব্যবহার করলে নিজের computer থেকে `scp -r ./my-app user@SERVER_IP:/opt/web1`
> দিয়ে file পাঠাতে পারো।

---

## ধাপ ৩ — প্রতিটা App প্রস্তুত করা

### Next.js app দুটো (web1, web2)

Next.js production-এ চালাতে আগে **build** করতে হয়, তারপর `next start` চলে।

```bash
# web1
cd /opt/web1
npm install
npm run build        # .next folder তৈরি করে

# web2
cd /opt/web2
npm install
npm run build
```

> `next start` স্বয়ংক্রিয়ভাবে `PORT` environment variable পড়ে। তাই আলাদা করে port কোডে
> লেখার দরকার নেই — PM2 থেকে PORT সেট করে দিলেই হবে (পরের ধাপে)।

### Express app দুটো (api1, api2)

Express-এ শুধু dependency install করলেই হয়:

```bash
cd /opt/api1 && npm install
cd /opt/api2 && npm install
```

> ⚠️ **গুরুত্বপূর্ণ:** তোমার Express কোডে port যেন hard-code করা না থাকে। এটা এমন হওয়া উচিত:
> ```js
> const PORT = process.env.PORT || 4000;
> app.listen(PORT, () => console.log(`Server running on ${PORT}`));
> ```
> তাহলে PM2 থেকে PORT সেট করলে সেটাই কাজ করবে।

### Environment variables (.env)

প্রতিটা app-এর নিজের `.env` file দরকার হলে সেটা ওই folder-এ বানাও, যেমন:

```bash
cd /opt/api1
nano .env
# ভেতরে: DATABASE_URL=..., JWT_SECRET=... ইত্যাদি
```

Next.js-এ browser-এ লাগবে এমন variable-এর নাম `NEXT_PUBLIC_` দিয়ে শুরু করতে হয়, আর সেগুলো
**build-এর আগে** সেট থাকতে হবে (তাই `.env` বানিয়ে তারপর `npm run build`)।

---

## ধাপ ৪ — PM2 দিয়ে চারটা App একসাথে চালানো

সব app একটা জায়গা থেকে manage করতে একটা **ecosystem file** বানাই।

```bash
nano /opt/ecosystem.config.js
```

ভেতরে লেখো:

```js
module.exports = {
  apps: [
    {
      name: "web1",
      cwd: "/opt/web1",
      script: "npm",
      args: "start",                 // package.json-এ "start": "next start"
      env: { NODE_ENV: "production", PORT: 3000 },
    },
    {
      name: "web2",
      cwd: "/opt/web2",
      script: "npm",
      args: "start",
      env: { NODE_ENV: "production", PORT: 3001 },
    },
    {
      name: "api1",
      cwd: "/opt/api1",
      script: "server.js",           // তোমার entry file (index.js/app.js হতে পারে)
      env: { NODE_ENV: "production", PORT: 4000 },
    },
    {
      name: "api2",
      cwd: "/opt/api2",
      script: "server.js",
      env: { NODE_ENV: "production", PORT: 4001 },
    },
  ],
};
```

এবার চালু করো:

```bash
cd /opt
pm2 start ecosystem.config.js

pm2 list            # চারটা app 'online' দেখাচ্ছে কিনা দেখো
pm2 logs            # কোনো error আছে কিনা live দেখো (বেরোতে Ctrl+C)

# server reboot হলেও app যেন auto-start হয়
pm2 startup         # একটা command print করবে — সেটা copy করে চালাও
pm2 save            # বর্তমান app-list মনে রাখে
```

### App ঠিকঠাক চলছে কিনা server থেকেই যাচাই

Nginx setup করার আগেই server-এর ভেতর থেকে test করা যায়:

```bash
curl http://localhost:3000     # web1
curl http://localhost:4000     # api1
```

HTML বা JSON response এলে বুঝবে app চলছে। এবার এদের সামনে Nginx বসাব।

---

## ধাপ ৫ — Nginx Reverse Proxy (subdomain → app)

প্রতিটা subdomain-এর request সঠিক port-এ পাঠাতে Nginx config বানাই। প্রথমে HTTP (port 80) দিয়ে
কাজ করিয়ে নিই, তারপর ধাপ ৬-এ SSL যোগ করব।

```bash
sudo nano /etc/nginx/sites-available/apps
```

ভেতরে চারটা `server` block লেখো:

```nginx
# ---------- Next.js App 1 ----------
server {
    listen 80;
    server_name web1.example.com;

    location / {
        proxy_pass http://localhost:3000;
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

# ---------- Next.js App 2 ----------
server {
    listen 80;
    server_name web2.example.com;

    location / {
        proxy_pass http://localhost:3001;
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

# ---------- Express API 1 ----------
server {
    listen 80;
    server_name api1.example.com;

    location / {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# ---------- Express API 2 ----------
server {
    listen 80;
    server_name api2.example.com;

    location / {
        proxy_pass http://localhost:4001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> Next.js-এর block-এ `Upgrade`/`Connection` header গুলো আছে যাতে hot-reload/WebSocket ঠিকমতো চলে।
> Express-এও WebSocket (socket.io ইত্যাদি) ব্যবহার করলে ওই দুটো header যোগ করে নিও।

Config enable করে reload:

```bash
sudo ln -s /etc/nginx/sites-available/apps /etc/nginx/sites-enabled/
sudo nginx -t                       # ভুল আছে কিনা check
sudo systemctl reload nginx
```

এই পর্যায়ে **এখনো domain দিয়ে browser-এ test কোরো না** — কারণ তোমার Cloudflare proxy 🟠 on,
আর origin-এ এখনো HTTPS (SSL) বসানো হয়নি। এখন domain খুললে 525/522 error আসতে পারে।

তার বদলে server-এর ভেতর থেকেই সরাসরি test করো — এটা Cloudflare-কে একেবারে ছোঁয় না:

```bash
# app সরাসরি চলছে কিনা
curl http://localhost:3000        # web1
curl http://localhost:4000        # api1

# Nginx সঠিক subdomain-কে সঠিক app-এ পাঠাচ্ছে কিনা (Host header দিয়ে নকল করে)
curl -H "Host: web1.example.com" http://localhost
curl -H "Host: api1.example.com" http://localhost
```

সঠিক HTML/JSON response এলে বুঝবে app → PM2 → Nginx চেইনটা ঠিক আছে। এবার SSL বসিয়ে
তারপর domain দিয়ে browser-এ খুলব।

> **কেন এখানে Flexible mode ব্যবহার করছি না?** Cloudflare-কে সাময়িকভাবে "Flexible" করে HTTP দিয়ে
> test করা যেত, কিন্তু পরে Full strict-এ ফেরত যেতে হয় — এই flip-flop থেকেই redirect loop-এর মতো
> ঝামেলা হয়। তাই আমরা এখানে Cloudflare-কে ছুঁইই না; local `curl` দিয়ে test করে সরাসরি ধাপ ৬-এ
> SSL + Full strict-এ যাই। এতে কোনো টগল লাগে না।

---

## ধাপ ৬ — HTTPS / SSL (Cloudflare Origin Certificate দিয়ে, wildcard)

যেহেতু Cloudflare ব্যবহার করছ আর proxy on, সবচেয়ে সহজ ও নির্ভরযোগ্য উপায় হলো একটা
**Origin Certificate** বানানো যা `*.example.com` সব subdomain কভার করে।

> **ক্রম নিয়ে একটা কথা:** নিচের ৬.১-এ Cloudflare-এ cert **বানানোটা** যেকোনো সময় করা যায় —
> চাইলে একদম শুরুতেই (ধাপ ০-এর সাথে) বানিয়ে রাখতে পারো, এর কোনো dependency নেই। শুধু server-এ
> **install করা** (৬.৩–৬.৪) Nginx-এর `server` block তৈরির পরেই সম্ভব, তাই সেটা এখানে রাখা হয়েছে।

### ৬.১ Cloudflare-এ Origin Certificate বানাও

1. Cloudflare dashboard → **SSL/TLS** → **Origin Server** → **Create Certificate**।
2. Hostnames দাও: `*.example.com` এবং `example.com`।
3. তৈরি হলে দুটো জিনিস দেখাবে — **Origin Certificate** আর **Private Key**। দুটোই copy করে রাখো।

### ৬.২ Server-এ certificate save করো

```bash
sudo mkdir -p /etc/ssl/cloudflare

# certificate (উপরের "Origin Certificate" অংশ পেস্ট করো)
sudo nano /etc/ssl/cloudflare/example.com.pem

# private key (উপরের "Private Key" অংশ পেস্ট করো)
sudo nano /etc/ssl/cloudflare/example.com.key

# key-এর permission নিরাপদ করো
sudo chmod 600 /etc/ssl/cloudflare/example.com.key
```

### ৬.৩ SSL setting একটা snippet-এ রাখো (বারবার লেখা এড়াতে)

```bash
sudo nano /etc/nginx/snippets/cloudflare-ssl.conf
```

```nginx
ssl_certificate     /etc/ssl/cloudflare/example.com.pem;
ssl_certificate_key /etc/ssl/cloudflare/example.com.key;
ssl_protocols       TLSv1.2 TLSv1.3;
```

### ৬.৪ Nginx config-টা HTTPS-এ আপডেট করো

`/etc/nginx/sites-available/apps` খুলে প্রতিটা block-কে এই রূপে বদলাও (উদাহরণ web1-এর জন্য):

```nginx
# HTTP → HTTPS redirect
server {
    listen 80;
    server_name web1.example.com;
    return 301 https://$host$request_uri;
}

# আসল HTTPS block
server {
    listen 443 ssl;
    server_name web1.example.com;
    include snippets/cloudflare-ssl.conf;

    location / {
        proxy_pass http://localhost:3000;
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

বাকি তিনটা (web2 → 3001, api1 → 4000, api2 → 4001) একইভাবে বদলাও — শুধু `server_name` আর
`proxy_pass`-এর port পাল্টাবে।

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### ৬.৫ Cloudflare-এ SSL mode ঠিক করো

Cloudflare dashboard → **SSL/TLS** → **Overview** → mode সেট করো **Full (strict)**।
(এতে Cloudflare ↔ তোমার server পর্যন্ত পুরো path এনক্রিপ্টেড থাকবে।)

চাইলে **SSL/TLS → Edge Certificates → Always Use HTTPS** চালু করে দাও।

এবার `https://web1.example.com` তালার আইকনসহ খুলবে। 🔒

---

## ধাপ ৭ — CORS (Frontend ↔ Backend কথা বলা)

তোমার Next.js app যদি Express API-তে request পাঠায় (আলাদা subdomain, তাই "cross-origin"),
তাহলে Express-এ CORS allow করতে হবে। Express কোডে:

```bash
cd /opt/api1 && npm install cors
```

```js
const cors = require("cors");

app.use(cors({
  origin: ["https://web1.example.com", "https://web2.example.com"],
  credentials: true,   // cookie/auth পাঠালে দরকার
}));
```

কোড বদলানোর পর ওই app-টা restart করো: `pm2 restart api1`।

---

## ধাপ ৮ — কোড আপডেট হলে নতুন করে Deploy করা

পরে যখন GitHub-এ নতুন কোড push করবে, server-এ এসে:

**Next.js app আপডেট:**

```bash
cd /opt/web1
git pull
npm install          # নতুন dependency এলে
npm run build        # নতুন build জরুরি
pm2 restart web1
```

**Express app আপডেট:**

```bash
cd /opt/api1
git pull
npm install
pm2 restart api1
```

---

## দরকারি Command চিটশিট

```bash
# ---- PM2 ----
pm2 list                 # সব app-এর অবস্থা
pm2 logs web1            # নির্দিষ্ট app-এর log
pm2 restart web1         # একটা app restart
pm2 restart all          # সব restart
pm2 stop api2            # একটা বন্ধ
pm2 delete web2          # list থেকে সরানো
pm2 save                 # বর্তমান অবস্থা মনে রাখা

# ---- Nginx ----
sudo nginx -t                          # config check
sudo systemctl reload nginx            # reload (downtime ছাড়া)
sudo tail -f /var/log/nginx/error.log  # live error

# ---- কোন port-এ কী চলছে ----
sudo ss -ltnp | grep -E '3000|3001|4000|4001'
```

---

## Troubleshooting (কিছু ভুল হলে)

| লক্ষণ | সম্ভাব্য কারণ ও সমাধান |
|-------|------------------------|
| **502 Bad Gateway** | পেছনের app বন্ধ বা ভুল port। `pm2 list` দেখো, `curl http://localhost:PORT` দিয়ে test করো। |
| **সব subdomain একই site দেখাচ্ছে** | `server_name` ভুল, বা পুরনো `default` config চালু আছে। `sudo rm /etc/nginx/sites-enabled/default` করে reload দাও। |
| **SSL error / "not secure"** | Cloudflare SSL mode ভুল (Full strict হওয়া উচিত), বা Origin cert ঠিকমতো paste হয়নি। |
| **Redirect loop (too many redirects)** | Cloudflare mode "Flexible" + Nginx-এ 80→443 redirect একসাথে থাকলে হয়। mode **Full (strict)** করো। |
| **CORS error (browser console-এ)** | Express-এ ঠিক origin allow করা হয়নি (ধাপ ৭)। |
| **Next.js build fail (memory)** | RAM কম — ধাপ ১-এর swap file বানাও। |
| **reboot-এর পর app বন্ধ** | `pm2 startup` চালিয়ে print হওয়া command রান করোনি, বা `pm2 save` করোনি। |

---

## এক নজরে পুরো ছবি

```
Browser (https://web1.example.com)
        │
        ▼
Cloudflare (DNS + SSL edge, Full strict)
        │  (Origin cert দিয়ে এনক্রিপ্টেড)
        ▼
তোমার VPS-এর Nginx (443) ──► server_name মিলিয়ে ──► সঠিক localhost:PORT
        │
        ▼
PM2-তে চলা Next.js / Express app
```

মূল যে চারটা জিনিস মনে রাখবে:

- প্রতিটা app আলাদা **port**-এ চলে, **PM2** সেগুলো চালু রাখে।
- **Nginx** subdomain (`server_name`) দেখে সঠিক port-এ `proxy_pass` করে।
- **Cloudflare wildcard `*`** সব subdomain-কে server-এর IP-তে আনে।
- **Origin Certificate** (`*.example.com`) দিয়ে HTTPS, Cloudflare mode **Full (strict)**।

এই প্যাটার্ন একবার বুঝে গেলে ৫ম, ৬ষ্ঠ subdomain যোগ করা মিনিটের কাজ — শুধু নতুন port, নতুন
PM2 entry, আর নতুন Nginx server block। 🚀