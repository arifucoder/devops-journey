# Cloudflare দিয়ে Domain + VPS Setup (SSL সহ)

## ধাপ ১: Cloudflare-এ Domain যোগ করা

1. Namecheap থেকে domain name নিন।
2. Cloudflare-এ যান: **Domains → Overview → Add domain → Connect domain**
3. Domain name লিখুন → **Free plan / Pro plan** সিলেক্ট করুন → **Continue to activation** ক্লিক করুন।
4. এখন Nameservers পাওয়া যাবে, যেমন:
   ```
   coby.ns.cloudflare.com
   galilea.ns.cloudflare.com
   ```
5. Namecheap-এ গিয়ে এই nameservers আপডেট করুন → **Done, check nameservers** ক্লিক করুন।
6. Cloudflare-এর সাথে propagate হতে কিছুটা সময় লাগবে।

## ধাপ ২: DNS Records সেট করা

Cloudflare → **DNS → Records**-এ যান:

- **A record**: আগে থেকে থাকলে edit করুন, না থাকলে নতুন add করুন।
  - Name = domain
  - IPv4 address = VPS-এর IP
  - Proxy status = **DNS only** (troubleshooting সহজ করার জন্য শুরুতে এটাই রাখা ভালো)
  - Save করুন

- **CNAME record**:
  - Name = `www`
  - Target = domain
  - TTL = Auto
  - Save করুন

  *(এটা চাইলে A record হিসেবেও `www` + IP দিয়ে add করা যায়)*

- MX ও TXT records আগের মতোই থাকবে (কোনো পরিবর্তন লাগবে না)।

এরপর ping করে চেক করুন সঠিক server IP আসছে কিনা:
```bash
ping domain.ext
```

### Proxied (কমলা cloud) মোডের সুবিধা

- Visitor প্রথমে Cloudflare-এর সার্ভারে যায়, তারপর Cloudflare সেটা VPS-এ ফরওয়ার্ড করে — visitor আসল VPS IP দেখতে পায় না।
- **সুবিধা:**
  - DDoS protection
  - CDN caching (static content দ্রুত লোড হয়, সার্ভার লোড কমে)
  - Free SSL (visitor-browser পর্যন্ত, তবে Full/Strict mode-এর জন্য origin-এ আলাদা certificate লাগে)
  - আসল সার্ভার IP লুকানো থাকায় সরাসরি আক্রমণ করা কঠিন হয়

**শুরুতে "DNS only" রাখার কারণ:** নতুন সেটআপে Nginx কনফিগার ও Certbot দিয়ে SSL ইস্যু করার সময় Cloudflare প্রক্সি মাঝে থাকলে —
- Certbot-এর HTTP verification ফেইল করতে পারে (verification request Cloudflare-এর IP-তে আটকে যায়)
- Redirect loop বা "too many redirects" এরর হতে পারে (Cloudflare ও VPS-এর SSL mode mismatch হলে)

## ধাপ ৩: SSL/TLS Setup

1. Cloudflare → **SSL/TLS** ট্যাবে যান → **Configure** → **Full (strict)** সিলেক্ট করুন → Save করুন।
2. SSL/TLS → **Origin Server** → **Create Certificate**
   - Private key type: **RSA**
   - Hostnames: `domain.ext` এবং `*.domain.ext` (wildcard SSL চাইলে)
   - Certificate validity: সর্বোচ্চ ১৫ বছর
   - **Create** ক্লিক করুন
3. এখন দুটি key পাওয়া যাবে: **certificate** ও **private key**

## ধাপ ৪: VPS-এ Certificate সেটআপ

SSH দিয়ে VPS-এ কানেক্ট করুন, তারপর `/etc/ssl/cloudflare` ডিরেক্টরিতে যান।

**Certificate ফাইল বানান:**
```bash
vim theextraschool.com.pem
```
Cloudflare থেকে পাওয়া certificate paste করে save করুন। Save হওয়ার পর `cat` দিয়ে চেক করুন।

**Key ফাইল বানান:**
```bash
vim theextraschool.com.key
```
Cloudflare থেকে পাওয়া private key এখানে paste করুন।

**Permission ঠিক করুন:**
```bash
sudo chmod 644 /etc/ssl/cloudflare/theextraschool.com.pem
sudo chmod 600 /etc/ssl/cloudflare/theextraschool.com.key
```

**Snippet ফাইল বানান:**
```bash
sudo vim /etc/nginx/snippets/cloudflare-ssl-theextraschool.conf
```
এতে এই দুই লাইন লিখুন:
```bash
ssl_certificate     /etc/ssl/cloudflare/theextraschool.com.pem;
ssl_certificate_key /etc/ssl/cloudflare/theextraschool.com.key;
```

Nginx server block-এ এই snippet include করতে হবে:
```bash
include snippets/cloudflare-ssl-theextraschool.conf;
```

## ধাপ ৫: Nginx Configuration

```bash
vim /etc/nginx/sites-available/theextraschool.com
```

এতে এই কোড দিন:
```bash
# HTTP -> HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name theextraschool.com www.theextraschool.com;

    return 301 https://$host$request_uri;
}

# HTTPS server block
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name theextraschool.com www.theextraschool.com;

    root /var/www/theextraschool.com;
    index index.html index.htm;

    include snippets/cloudflare-ssl-theextraschool.conf;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Symlink বানান:**
```bash
sudo ln -s /etc/nginx/sites-available/theextraschool.com /etc/nginx/sites-enabled/
```

Static site-এর ফাইল `/var/www/theextraschool.com`-এ রাখুন।

**Test ও reload করুন:**
```bash
sudo nginx -t
```
Syntax okay হলে:
```bash
sudo systemctl reload nginx
```

## ধাপ ৬: Subdomain যোগ করা (উদাহরণ: `app.theextraschool.com`)

**১. Cloudflare-এ DNS record যোগ করুন**

DNS → Add record:
```bash
Type: A
Name: app (শুধু subdomain অংশ)
Content: 81.17.101.34 (VPS IP)
Proxy status: Proxied (কমলা cloud)
```

**২. VPS-এ static ফাইলের জন্য ফোল্ডার বানান**
```bash
sudo mkdir -p /var/www/app.theextraschool.com
```
এখানে subdomain-এর কন্টেন্ট (index.html ইত্যাদি) রাখুন।

**৩. নতুন Nginx config ফাইল বানান**
```bash
sudo vim /etc/nginx/sites-available/app.theextraschool.com
```

কন্টেন্ট:
```bash
server {
    listen 80;
    listen [::]:80;
    server_name app.theextraschool.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name app.theextraschool.com;

    root /var/www/app.theextraschool.com;
    index index.html index.htm;

    include snippets/cloudflare-ssl-theextraschool.conf;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
> লক্ষ্য করুন — `cloudflare-ssl-theextraschool.conf` snippet-টাই আবার reuse করা হচ্ছে, কারণ এটা wildcard cert (নতুন cert বানানো লাগছে না)।

**৪. সাইট enable করুন**
```bash
sudo ln -s /etc/nginx/sites-available/app.theextraschool.com /etc/nginx/sites-enabled/
```

**৫. টেস্ট ও reload করুন**
```bash
sudo nginx -t
sudo systemctl reload nginx
```

**৬. ব্রাউজারে চেক করুন**
```
https://app.theextraschool.com
```