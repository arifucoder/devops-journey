# Express App Deploy: VPS + PM2 + Nginx Reverse Proxy (Cloudflare SSL)

## ধাপ ১: App VPS-এ আনা

Express app-টি clone অথবা copy করে VPS-এর `/opt` ফোল্ডারে নিতে হবে।

## ধাপ ২: PM2 দিয়ে App Run করা

`/opt`-এর যেখানে app copy করা হয়েছে, সেখানে গিয়ে PM2-এর মাধ্যমে নির্দিষ্ট একটি port-এ app run করাতে হবে।

## ধাপ ৩: Nginx Config ফাইল বানানো

`/etc/nginx/sites-available`-এ নির্দিষ্ট নামে একটি ফাইল বানাতে হবে:

```bash
sudo vim /etc/nginx/sites-available/foodika.theextraschool.com
```

কন্টেন্ট:

```bash
server {
    listen 443 ssl;
    server_name foodika.theextraschool.com;
    include /etc/nginx/snippets/cloudflare-ssl-theextraschool.conf;
    location / {
        proxy_pass http://localhost:5000;
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
server {
    listen 80;
    server_name foodika.theextraschool.com;
    return 301 https://$host$request_uri;
}
```

## ধাপ ৪: সাইট Enable করা

```bash
sudo ln -s /etc/nginx/sites-available/foodika.theextraschool.com /etc/nginx/sites-enabled/
```

## ধাপ ৫: Test করা

```bash
sudo nginx -t
```

## ধাপ ৬: Reload করা

```bash
sudo systemctl reload nginx
```