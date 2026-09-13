# PERN Stack — Docker + CI/CD সম্পূর্ণ গাইড

এই গাইডে ধাপে ধাপে দেখানো হয়েছে কীভাবে একটা PERN (PostgreSQL, Express, React, Node) প্রজেক্ট থেকে শুরু করে সম্পূর্ণ Dockerized, production-এ live, এবং GitHub Actions দিয়ে CI/CD automate করা একটা অ্যাপ পর্যন্ত নিয়ে যাওয়া যায়।

---

## ধাপ ১: প্রজেক্ট স্ট্রাকচার তৈরি

Root ফোল্ডারের ভেতরে `client` আর `server` নামে দুইটা আলাদা ফোল্ডার থাকবে:

```
project-root/
├── client/     # React (Vite) frontend
└── server/     # Express + Prisma backend
```

---

## ধাপ ২: Git init এবং .gitignore

Root ফোল্ডারে গিয়ে:

```bash
git init
```

তিন জায়গাতেই আলাদা `.gitignore` রাখা ভালো — **root**, **client**, এবং **server** — যাতে `node_modules`, `.env`, `dist` ইত্যাদি কখনোই GitHub এ push না হয়।

```gitignore
# root/.gitignore, client/.gitignore, server/.gitignore — এ common entries
node_modules
dist
.env
.env.local
npm-debug.log
```

> **⚠️ গুরুত্বপূর্ণ:** `.env` ফাইল কখনোই git এ push হবে না (এতে পাসওয়ার্ড/সিক্রেট থাকে)। VPS এ এই ফাইলগুলো আলাদাভাবে `scp` দিয়ে পাঠাতে হবে।

---

## ধাপ ৩: server আগে, তারপর client

প্রথমে `server/` এর কাজ (Express routes, Prisma schema, models) সম্পূর্ণ করে লোকালি টেস্ট করে নাও। server ঠিকমতো কাজ করলে তারপর `client/` (React UI, API calls) এর কাজ শুরু করো।

দুইটাই আলাদাভাবে (`npm run dev`) লোকালি ভালোভাবে চললে — তখনই Docker এ হাত দাও।

---

## ধাপ ৪: Dockerfile ও docker-compose.yml লেখা

### server/Dockerfile (multi-stage build)

```dockerfile
# ---------- Stage 1: Build ----------
FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
COPY prisma ./prisma
RUN npm ci
RUN npx prisma generate

COPY . .
RUN npm run build

# ---------- Stage 2: Runtime ----------
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/prisma.config.ts ./

EXPOSE 5000
CMD ["node", "dist/server.js"]
```

> **⚠️ সাধারণ ভুল:** runtime stage এ যদি `prisma.config.ts` কপি করা ভুলে যাও, তাহলে `prisma migrate deploy` চালানোর সময় "Config file not found" এরর আসবে — কারণ builder stage এ ফাইলটা থাকলেও, শেষ image এ সেটা না থাকলে কাজ করবে না।

### client/Dockerfile (multi-stage: build + nginx serve)

```dockerfile
# ---------- Stage 1: Build ----------
FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci
COPY . .

ARG VITE_API_BASE=http://localhost:5000/api
ENV VITE_API_BASE=$VITE_API_BASE
RUN npm run build

# ---------- Stage 2: Serve with nginx ----------
FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### client/nginx.conf (container এর ভেতরের — SPA fallback এর জন্য)

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### docker-compose.yml (root এ)

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - nudodata:/var/lib/postgresql/data
    networks:
      - nudo-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5

  server:
    build:
      context: ./server
      dockerfile: Dockerfile
    environment:
      PORT: 5000
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
    ports:
      - "127.0.0.1:5000:5000"
    depends_on:
      db:
        condition: service_healthy
    networks:
      - nudo-net

  client:
    build:
      context: ./client
      dockerfile: Dockerfile
      args:
        VITE_API_BASE: ${VITE_API_BASE:-http://localhost:5000/api}
    ports:
      - "127.0.0.1:8080:80"
    depends_on:
      - server
    networks:
      - nudo-net

volumes:
  nudodata:

networks:
  nudo-net:
    driver: bridge
```

> **নোট:** `127.0.0.1:5000:5000` এভাবে explicit IP দিয়ে port bind করা হয়েছে (শুধু `5000:5000` না) — যাতে reverse proxy ব্যবহার করলে container এর port গুলো বাইরের ইন্টারনেট থেকে সরাসরি access করা না যায়, শুধু host machine এর ভেতর থেকেই (nginx দিয়ে) পৌঁছানো যায়।

---

## ধাপ ৫: লোকালি টেস্ট এবং GitHub এ push

```bash
docker compose up --build
```

সব container ঠিকমতো উঠলে ও কাজ করলে:

```bash
git add .
git commit -m "Add Docker setup"
git push origin main
```

**প্রতিটা বড় ধাপের পর push করাই ভালো অভ্যাস** — যাতে সমস্যা হলে আগের workingversion এ ফিরে যাওয়া যায়।

---

## ধাপ ৬: VPS এ Docker + nginx ইনস্টল

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2

# ইউজারকে docker group এ যোগ করো (root না হলে দরকার)
sudo usermod -aG docker $USER
newgrp docker

# ভেরিফাই করো
docker --version
docker compose version

# host-level reverse proxy এর জন্য nginx
sudo apt install nginx -y
```

> সরকারিভাবে Docker এর official install script (`curl -fsSL https://get.docker.com | sh`) ব্যবহার করলে সবসময় latest ভার্সন পাওয়া যায়, তবে `apt install docker.io docker-compose-v2` দিয়েও কাজ চলে যদি Ubuntu ভার্সন যথেষ্ট নতুন হয় (22.04+)।

---

## ধাপ ৭: রিপো ক্লোন এবং .env ফাইল আনা

```bash
cd ~
git clone https://github.com/<username>/<repo>.git nudo
cd nudo
```

লোকাল PC থেকে `.env` ফাইলগুলো `scp` দিয়ে পাঠাও:

```bash
scp .env root@<vps-ip>:~/nudo/
scp client/.env root@<vps-ip>:~/nudo/client/
scp server/.env root@<vps-ip>:~/nudo/server/
```

> **root `.env`** ফাইলেই `docker-compose.yml` এর সব variable (`POSTGRES_*`, `VITE_API_BASE`) থাকতে হবে — `client/.env` বা `server/.env` এ থাকলেও `docker compose` কমান্ড শুধু root এর `.env` পড়ে।

---

## ধাপ ৮: Nginx reverse-proxy কনফিগ লেখা

### deploy/app.theextraschool.com.conf (client এর জন্য)

```nginx
server {
    listen 80;
    server_name app.theextraschool.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name app.theextraschool.com;

    include snippets/cloudflare-ssl-theextraschool.conf;

    location / {
        proxy_pass http://127.0.0.1:8080;
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

### deploy/api.theextraschool.com.conf (server এর জন্য)

```nginx
server {
    listen 80;
    server_name api.theextraschool.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name api.theextraschool.com;

    include snippets/cloudflare-ssl-theextraschool.conf;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Enable এবং reload

```bash
sudo cp app.theextraschool.com.conf /etc/nginx/sites-available/
sudo cp api.theextraschool.com.conf /etc/nginx/sites-available/

sudo ln -s /etc/nginx/sites-available/app.theextraschool.com.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api.theextraschool.com.conf /etc/nginx/sites-enabled/

sudo nginx -t          # syntax check
sudo systemctl reload nginx
```

> Cloudflare DNS এ `app` ও `api` সাবডোমেইনের জন্য A record VPS এর IP দিয়ে বসাতে হবে, আর Cloudflare SSL/TLS মোড **Full (strict)** এ রাখতে হবে।

---

## ধাপ ৯: Prisma config সমস্যা ঠিক করা

যদি `prisma7.config.ts` নামের ফাইল থাকে, সেটা রিনেম করে দাও — কারণ Prisma CLI ডিফল্টভাবে শুধু `prisma.config.ts` নামটাই auto-detect করে:

```bash
mv server/prisma7.config.ts server/prisma.config.ts
```

তারপর কমিট করে push করো, VPS এ pull করে rebuild করো:

```bash
docker compose up --build -d
docker compose exec server npx prisma migrate deploy
```

---

## ধাপ ১০: সাধারণ সমস্যা ও সমাধান (Troubleshooting)

| সমস্যা | কারণ | সমাধান |
|---|---|---|
| `port already in use` | Host এর ঐ পোর্টে অন্য প্রোগ্রাম চলছে | `lsof -i :5000 -t` দিয়ে PID বের করে `kill -9 <PID>` |
| `relation "Todo" does not exist` | Migration চালানো হয়নি | `docker compose exec server npx prisma migrate deploy` |
| `datasource.url required` | Prisma config ফাইলের নাম ভুল/কপি হয়নি | ফাইল রিনেম + Dockerfile এ কপি করা নিশ্চিত করো |
| Browser এ `localhost:5000` এ request যাচ্ছে | root `.env` এ `VITE_API_BASE` নেই, তাই ডিফল্ট মান বসে গেছে | root `.env` এ সঠিক মান যোগ করে `--no-cache` দিয়ে client rebuild করো |
| SSH handshake fail: passphrase protected | Key জেনারেট করার সময় passphrase বসে গেছে | `-N ""` দিয়ে passphrase ছাড়া নতুন key বানাও |

---

## ধাপ ১১: CI/CD এর জন্য SSH key তৈরি

```bash
ssh-keygen -t ed25519 -f ~/.ssh/theextraschool_deploy_key -N ""
cat ~/.ssh/theextraschool_deploy_key.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/theextraschool_deploy_key   # এই প্রাইভেট key টা কপি করো
```

> **Passphrase অবশ্যই খালি রাখতে হবে** (`-N ""`), নাহলে GitHub Actions স্বয়ংক্রিয়ভাবে login করতে পারবে না।

**কীভাবে কাজ করে:**
- Public key (`.pub`) → VPS এর `authorized_keys` এ থাকে ("এই key বিশ্বাসযোগ্য")
- Private key → GitHub Secrets এ থাকে
- GitHub Actions প্রাইভেট key দিয়ে SSH করে, VPS মিলিয়ে দেখে জোড়া key আছে কিনা

### GitHub Secrets এ যোগ করো (repo → Settings → Secrets and variables → Actions)

| Secret নাম | মান |
|---|---|
| `DEPLOY_HOST` | VPS এর IP |
| `DEPLOY_USER` | `root` (বা যে ইউজার দিয়ে SSH করো) |
| `DEPLOY_SSH_KEY` | উপরের প্রাইভেট key (পুরোটা, `-----BEGIN...` থেকে `-----END...` পর্যন্ত) |

---

## ধাপ ১২: CI/CD ওয়ার্কফ্লো (.github/workflows/ci-cd.yml)

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # ---------- CI: প্রতিটা push/PR এ চলে ----------
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install & build server
        working-directory: server
        run: |
          npm ci
          npx prisma generate
          npm run build

      - name: Install, lint & build client
        working-directory: client
        run: |
          npm ci
          npm run lint
          npm run build

  # ---------- CD: শুধু main এ push হলে, CI পাশ করার পর ----------
  deploy:
    needs: build-and-test
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to VPS via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_SSH_KEY }}
          script: |
            cd ~/nudo-pern-basic
            git pull origin main
            docker compose up --build -d
            docker compose exec -T server npx prisma migrate deploy
```

**এই ওয়ার্কফ্লো যা করে:**
1. `main` এ push হলে CI অংশ চলে (server build + client lint/build) — কোনো একটা fail করলে deploy হবে না
2. CI পাশ করলে CD অংশ SSH দিয়ে VPS এ ঢুকে ঠিক ম্যানুয়াল deploy কমান্ডগুলোই চালায়
3. `git pull` → `docker compose up --build -d` → `prisma migrate deploy` — এই তিনটা ধাপই automate হয়ে যায়

কোড push করে GitHub repo এর **Actions** ট্যাবে গিয়ে workflow run লাইভ দেখা যায়।

---

## সারসংক্ষেপ — পুরো ফ্লো একনজরে

1. Project structure বানাও (client + server)
2. `git init` + `.gitignore` (তিন জায়গায়)
3. server আগে বানাও, তারপর client
4. দুইটা লোকালি আলাদাভাবে ভালোভাবে চললে Dockerfile + docker-compose লেখো
5. লোকালি `docker compose up --build` দিয়ে টেস্ট করো, GitHub এ push করো
6. VPS এ Docker + nginx ইনস্টল করো
7. Repo clone করো, `.env` ফাইল scp করো
8. Reverse-proxy nginx config লেখো, enable করো, SSL কনফার্ম করো
9. Docker compose port গুলো `127.0.0.1` তে bind করো
10. Prisma migration চালাও, ট্রাবলশুট করো যা লাগে
11. SSH deploy key বানাও, GitHub Secrets এ বসাও
12. CI/CD yaml লেখো, push করো — automatic deploy চালু!