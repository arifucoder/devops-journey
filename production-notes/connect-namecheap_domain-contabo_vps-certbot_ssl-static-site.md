# Namecheap Domain → Contabo VPS (Static Site + Auto-renew SSL, No Cloudflare)

এই গাইডে Cloudflare ছাড়াই সরাসরি Namecheap domain-কে VPS-এর সাথে কানেক্ট করে **Let's Encrypt (Certbot)** দিয়ে SSL ইনস্টল করা হবে, যা নিজে থেকেই auto-renew হয়।

## ধাপ ১: Namecheap-এ DNS Record সেট করা

1. Namecheap থেকে পছন্দমতো domain কিনুন।
2. Namecheap dashboard → **Domain List → Manage → Advanced DNS**-এ যান।
3. **A record** add করুন:
   - Type = A Record
   - Host = `@`
   - Value = Contabo VPS-এর IP
   - TTL = Automatic
4. `www`-এর জন্য আরেকটি record add করুন:
   - Type = A Record (অথবা CNAME)
   - Host = `www`
   - Value = VPS IP (A record হলে) / domain.ext (CNAME হলে)
   - TTL = Automatic
5. Propagate হতে কিছু সময় লাগবে।

ping করে চেক করুন সঠিক IP আসছে কিনা:
```bash
ping domain.ext
```

## ধাপ ২: VPS-এ Nginx ও Static Site সেটআপ

SSH দিয়ে Contabo VPS-এ কানেক্ট করুন।

**Nginx ইনস্টল করুন (না থাকলে):**
```bash
sudo apt update
sudo apt install nginx -y
```

**Static site-এর জন্য ফোল্ডার বানান:**
```bash
sudo mkdir -p /var/www/domain.ext
```
এখানে site-এর ফাইল (index.html ইত্যাদি) রাখুন।

**Nginx config ফাইল বানান:**
```bash
sudo vim /etc/nginx/sites-available/domain.ext
```

কন্টেন্ট (প্রথমে শুধু HTTP, SSL Certbot নিজেই পরে যোগ করে দেবে):
```bash
server {
    listen 80;
    listen [::]:80;
    server_name domain.ext www.domain.ext;

    root /var/www/domain.ext;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Symlink বানান:**
```bash
sudo ln -s /etc/nginx/sites-available/domain.ext /etc/nginx/sites-enabled/
```

**Test ও reload করুন:**
```bash
sudo nginx -t
sudo systemctl reload nginx
```

## ধাপ ৩: Certbot দিয়ে SSL ইনস্টল করা

**Certbot ইনস্টল করুন:**
```bash
sudo apt install certbot python3-certbot-nginx -y
```

**SSL সার্টিফিকেট ইস্যু করুন (Nginx config automatic আপডেট হবে):**
```bash
sudo certbot --nginx -d domain.ext -d www.domain.ext
```
- Email দিন, terms accept করুন।
- **HTTP → HTTPS redirect** করতে চাইলে সেই অপশন সিলেক্ট করুন।

Certbot নিজে থেকেই `/etc/nginx/sites-available/domain.ext`-এ SSL config (certificate path, port 443 block) যোগ করে দেবে।

## ধাপ ৪: Auto-renew Verify করা

Certbot ইনস্টলের সাথে সাথে auto-renew systemd timer বা cron job নিজে থেকেই সেট হয়ে যায়।

**Timer চেক করুন:**
```bash
sudo systemctl status certbot.timer
```

**Renewal dry-run টেস্ট করুন (আসলে renew না করেই টেস্ট):**
```bash
sudo certbot renew --dry-run
```

## ধাপ ৫: ব্রাউজারে চেক করুন

```
https://domain.ext
https://www.domain.ext
```