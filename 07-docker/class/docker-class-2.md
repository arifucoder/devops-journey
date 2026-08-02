# 🐳 Docker Class 2 — সম্পূর্ণ নোট

> **আজকের টপিকস**
> 1. Single vs Multi-Stage Build
> 2. Docker Compose
> 3. Docker Volume
> 4. Docker Network

---

## ১. লোকাল মেশিন থেকে সার্ভারে ফাইল পাঠানো (SCP)

নিজের মেশিন থেকে রিমোট সার্ভারে (যেমন AWS EC2) ফাইল আপলোড করতে `scp` ব্যবহার করি।

```bash
scp -i <pem-key> <file-name>.zip ubuntu@<server-ip>:/home/ubuntu/
```

উদাহরণ:

```bash
scp -i docker-playground-key.pem doc-single-multi-main.zip ubuntu@35.165.15.171:/home/ubuntu/
```

### কমান্ডের প্রতিটি অংশের মানে

| অংশ | পূর্ণরূপ / মানে | ব্যাখ্যা |
|---|---|---|
| `scp` | **S**ecure **C**o**p**y | SSH প্রোটোকলের উপর দিয়ে ফাইল কপি করে, তাই ডেটা encrypted থাকে |
| `-i` | `--identity_file` | কোন private key দিয়ে লগইন হবে সেটা বলে দেয় (`.pem` ফাইল) |
| `ubuntu@35.165.15.171` | `user@host` | সার্ভারের ইউজারনেম আর IP address |
| `:/home/ubuntu/` | destination path | সার্ভারের কোন ফোল্ডারে ফাইলটা রাখা হবে |

✅ **গুরুত্বপূর্ণ:** `.pem` key-এর permission ঠিক না থাকলে `scp`/`ssh` কাজ করবে না ("UNPROTECTED PRIVATE KEY FILE" error দেবে)। তাই একবার এই কমান্ডটা চালিয়ে নিতে হয়:

```bash
chmod 400 docker-playground-key.pem
```

✅ পুরো ফোল্ডার পাঠাতে চাইলে `-r` (recursive) ফ্ল্যাগ লাগে:

```bash
scp -i key.pem -r ./my-project ubuntu@35.165.15.171:/home/ubuntu/
```

### সার্ভারে zip ফাইল unzip করা

Ubuntu-তে `unzip` প্যাকেজ ডিফল্টভাবে ইনস্টল করা থাকে না, তাই আগে ইনস্টল করে নিতে হয়:

```bash
sudo apt update
sudo apt install -y unzip
unzip file.zip
```

---

## ২. `.dockerignore` ফাইল

### 📖 সংজ্ঞা

**`.dockerignore`** ✅ (নামটা সম্পূর্ণ ছোট হাতের অক্ষরে, শুরুতে একটা ডট) হলো এমন একটা ফাইল যেখানে আমরা লিখে দিই — কোন ফাইল/ফোল্ডারগুলো **build context**-এ Docker daemon-এর কাছে পাঠানো হবে না। ঠিক `.gitignore`-এর মতো কাজ করে, শুধু Git-এর বদলে Docker-এর জন্য।

**উদাহরণ ১:** `node_modules` লিখে দিলে লোকাল মেশিনের ভারী `node_modules` ফোল্ডারটা আর ইমেজে কপি হবে না — কারণ কন্টেইনারের ভেতরে `npm install` নিজেই সেটা তৈরি করে নেবে।

**উদাহরণ ২:** `.env` লিখে দিলে সিক্রেট API key বা database password ভুলেও ইমেজের ভেতরে ঢুকে যাবে না — এতে সিকিউরিটি রিস্ক কমে।

### React/Vite অ্যাপের সাধারণ `.dockerignore`

```
node_modules
dist
.git
.gitignore
Dockerfile
README.md
npm-debug.log
.env
```

### কেন দরকার?

| কারণ | ব্যাখ্যা |
|---|---|
| ⚡ Build দ্রুত হয় | কম ফাইল Docker daemon-এ পাঠাতে হয়, তাই build context হালকা হয় |
| 📦 Image ছোট হয় | অপ্রয়োজনীয় ফাইল ইমেজে ঢোকে না |
| 🔒 নিরাপদ | `.env`, `.git` ইত্যাদি সেনসিটিভ জিনিস ইমেজে leak হয় না |
| 🧊 Cache ভালো কাজ করে | অপ্রাসঙ্গিক ফাইল বদলালেও layer cache নষ্ট হয় না |

### 📖 Build Context কী?

`docker build` কমান্ডের শেষে যে `.` (ডট) থাকে — সেটাই build context। মানে হলো: **এই ফোল্ডারের সব ফাইল Docker engine-এর কাছে পাঠানো হবে**, যাতে `COPY`/`ADD` কমান্ডগুলো সেখান থেকে ফাইল নিতে পারে।

**উদাহরণ ১:** `docker build -t myapp .` → বর্তমান ফোল্ডারটাই context।
**উদাহরণ ২:** `docker build -t myapp ./frontend` → শুধু `frontend` ফোল্ডারটা context, বাইরের ফাইল `COPY` করা যাবে না।

---

## ৩. NGINX Config ফাইল বোঝা

```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### লাইন বাই লাইন

| লাইন | কাজ |
|---|---|
| `server { ... }` | একটা virtual server ব্লক — এক NGINX-এ একাধিক সাইট চালানো যায় |
| `listen 80;` | কন্টেইনারের ভেতরে ৮০ নম্বর পোর্টে HTTP request শুনবে |
| `server_name localhost;` | কোন domain name-এর request এই ব্লক handle করবে |
| `root /usr/share/nginx/html;` | ওয়েবসাইটের ফাইলগুলো কোথায় আছে (NGINX ইমেজের ডিফল্ট ফোল্ডার) |
| `index index.html;` | ফোল্ডার চাইলে ডিফল্টভাবে কোন ফাইল দেখাবে |
| `location / { ... }` | `/` দিয়ে শুরু হওয়া সব URL-এর জন্য নিয়ম |
| `try_files $uri $uri/ /index.html;` | ⭐ SPA fallback — নিচে ব্যাখ্যা |

### ✅ `try_files` লাইনটাই সবচেয়ে গুরুত্বপূর্ণ — কেন?

React/Vue/Angular হলো **SPA (Single Page Application)**। এখানে রাউটিং হয় ব্রাউজারের JavaScript দিয়ে, সার্ভারে আসলে `/about` বা `/dashboard` নামে কোনো ফাইল **থাকেই না** — শুধু `index.html` থাকে।

`try_files $uri $uri/ /index.html;` মানে NGINX ধাপে ধাপে চেষ্টা করবে:

1. `$uri` → ঠিক ওই নামের ফাইল আছে কি? (যেমন `/logo.png`)
2. `$uri/` → ওই নামের ফোল্ডার আছে কি?
3. দুটোই না পেলে → `index.html` ফেরত দাও ✅

এই লাইনটা **না দিলে** ইউজার `/dashboard` পেজে গিয়ে refresh দিলে **404 Not Found** দেখবে — SPA deploy-এর সবচেয়ে common বাগ।

---

## ৪. Single-Stage Build

```dockerfile
# Single-stage Dockerfile
# Builds and runs the React/Vite app in the same Node.js image.

FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ENV VITE_BUILD_TYPE="Single-stage Build"

RUN npm run build

# Vite's preview server runs on port 4173 by default.
EXPOSE 4173

CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0"]
```

### প্রতিটা instruction-এর ব্যাখ্যা

| Instruction | কাজ |
|---|---|
| `FROM node:20-alpine` | বেস ইমেজ। Node.js version 20, আর `alpine` হলো খুব হালকা Linux distro (~5MB), তাই ইমেজ ছোট হয় |
| `WORKDIR /app` | ইমেজের ভেতরে `/app` নামে ফোল্ডার বানাবে এবং এরপরের সব কমান্ড ওই ফোল্ডারের ভেতর থেকে চলবে (`cd`-এর মতো) |
| `COPY package*.json ./` | `package.json` এবং `package-lock.json` — এই দুটো ফাইল `/app`-এ কপি করবে |
| `RUN npm install` | `package.json` দেখে সব dependency ইনস্টল করবে |
| `COPY . .` | বাকি সব সোর্স কোড `/app`-এ কপি করবে (`.dockerignore`-এ থাকা ফাইলগুলো বাদ দিয়ে) |
| `ENV VITE_BUILD_TYPE="..."` | একটা environment variable সেট করে |
| `RUN npm run build` | Production build বানাবে → `dist/` ফোল্ডার তৈরি হবে |
| `EXPOSE 4173` | ডকুমেন্টেশন — "এই কন্টেইনার ৪১৭৩ পোর্ট ব্যবহার করে" |
| `CMD [...]` | কন্টেইনার **রান** হওয়ার সময় যে কমান্ডটা চলবে |

### ✅ কেন `package*.json` আলাদা করে আগে কপি করি? (Layer Caching)

Dockerfile-এর প্রতিটা লাইন একটা করে **layer** তৈরি করে, আর Docker সেই layer গুলো cache করে রাখে। যে লাইন থেকে কিছু বদলায়, সেই লাইন **এবং তার নিচের সব লাইন** আবার নতুন করে চলে।

- যদি প্রথমেই `COPY . .` দিতাম → কোডের একটা অক্ষর বদলালেও পুরো `npm install` আবার চলত (২-৩ মিনিট নষ্ট)।
- এখন যেহেতু আগে শুধু `package*.json` কপি করছি → dependency না বদলালে `npm install` layer টা cache থেকেই আসে, build কয়েক সেকেন্ডে শেষ ✅

### ✅ `EXPOSE` আসলে পোর্ট খুলে দেয় না

অনেকেই ভুল বোঝে। `EXPOSE 4173` শুধু **ডকুমেন্টেশন** — এটা লিখলেই বাইরে থেকে পোর্টে ঢোকা যায় না। বাইরে থেকে অ্যাক্সেস পেতে হলে `docker run`-এ `-p` ফ্ল্যাগ দিতেই হবে।

### ✅ `--host 0.0.0.0` কেন দরকার?

ডিফল্টভাবে Vite preview server শুধু `localhost` (127.0.0.1)-এ শোনে। তখন সেটা **শুধু কন্টেইনারের ভেতর থেকেই** অ্যাক্সেসযোগ্য থাকে, বাইরে থেকে না। `0.0.0.0` মানে "সব network interface-এ শোনো" — তাই এটা না দিলে `-p` করা সত্ত্বেও ব্রাউজারে কিছু আসবে না।

`CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0"]` — এখানে মাঝের `--` টা npm-কে বলে: "এরপরের আর্গুমেন্টগুলো তোমার না, Vite-কে পাঠিয়ে দাও।"

### Image build ও Container run

```bash
# Image build
docker build -f Dockerfile.single -t react-single:v1 .

# Container run
docker run -d --name react-single -p 3001:4173 react-single:v1
```

#### ফ্ল্যাগগুলোর পূর্ণরূপ

| ফ্ল্যাগ | পূর্ণরূপ | মানে |
|---|---|---|
| `-f` | `--file` | কোন Dockerfile ব্যবহার হবে (নাম `Dockerfile` না হলে লাগে) |
| `-t` | `--tag` | ইমেজের নাম ও ট্যাগ — `name:tag` ফরম্যাটে |
| `.` | build context | বর্তমান ফোল্ডার Docker-এর কাছে পাঠানো হবে |
| `-d` | `--detach` | কন্টেইনার ব্যাকগ্রাউন্ডে চলবে, টার্মিনাল আটকে থাকবে না |
| `--name` | — | কন্টেইনারের নাম (না দিলে Docker random নাম দেয়) |
| `-p 3001:4173` | `--publish` | `host_port:container_port` → সার্ভারের ৩০০১ পোর্টে হিট করলে কন্টেইনারের ৪১৭৩-এ যাবে |

> 🌐 ব্রাউজারে দেখতে: `http://<server-ip>:3001`
> ⚠️ AWS EC2 হলে Security Group-এ ওই পোর্টটা Inbound rule-এ খুলে দিতে হবে, নাহলে লোড হবে না।

---

## ৫. Multi-Stage Build

```dockerfile
# Multi-stage Dockerfile

# Stage 1 builds the React/Vite app with Node.js.
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ENV VITE_BUILD_TYPE="Multi-stage Build"

RUN npm run build


# Stage 2 serves only the final static files with NGINX.
FROM nginx:alpine AS runner

COPY --from=builder /app/dist /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### ✅ প্রথমেই একটা পরিষ্কার ধারণা: "Stage" মানে কন্টেইনার নয়

এখানে `builder` আর `runner` — এগুলো **কন্টেইনার না**। এগুলো হলো **build stage**, অর্থাৎ ইমেজ বানানোর সময়কার অস্থায়ী ধাপ। ✅

- প্রতিবার `FROM` লেখা মানেই একটা **নতুন stage** শুরু হচ্ছে।
- `AS builder` / `AS runner` হলো ওই stage-এর **নাম বা alias** — যাতে পরে নাম ধরে ডাকা যায়।
- Build শেষে **শুধু সর্বশেষ stage-টাই** ফাইনাল ইমেজ হয়ে থাকে। আগের stage গুলো (এবং তাদের ভেতরের `node_modules`, সোর্স কোড, npm cache সব) **ফেলে দেওয়া হয়**।
- এই ফাইনাল ইমেজ থেকে যখন `docker run` করব, **তখনই** সেটা কন্টেইনার হবে। মানে দুই stage থেকে দুটো কন্টেইনার হয় না — **একটাই কন্টেইনার** হয়।

### `AS builder` / `AS runner` না দিলে কী হয়?

Alias না দিলেও চলে — তখন stage-গুলোকে **নাম্বার** দিয়ে ডাকতে হয় (০ থেকে শুরু):

```dockerfile
COPY --from=0 /app/dist /usr/share/nginx/html
```

কিন্তু এটা পড়তে কষ্ট এবং stage-এর ক্রম বদলালে ভেঙে যায়। ✅ তাই সব সময় অর্থপূর্ণ নাম দেওয়াই best practice।

### `ENV VITE_BUILD_TYPE="Multi-stage Build"` — এটা আসলে কী?

`ENV` দিয়ে একটা environment variable সেট করা হচ্ছে। এখানে দুটো ব্যাপার বুঝতে হবে:

1. **Vite-এর নিয়ম:** Vite শুধু `VITE_` দিয়ে শুরু হওয়া variable-গুলোকেই ফ্রন্টএন্ড কোডে ঢুকতে দেয়। কোডে ব্যবহার হয় `import.meta.env.VITE_BUILD_TYPE` হিসেবে।
2. **কখন কাজ করছে:** `RUN npm run build`-এর **আগে** `ENV` দেওয়া হয়েছে, তাই build-এর সময় এই ভ্যালুটা `dist/` ফোল্ডারের JS ফাইলের ভেতরে **পাকাপাকিভাবে বসে যায়**।

এখানে এটার একমাত্র কাজ — ব্রাউজারে খুলে বোঝা যে "এই পেজটা Single-stage না Multi-stage build থেকে এসেছে"। ✅ **শেখার/ডেমোর জন্য দেওয়া, না দিলেও অ্যাপ চলবে।**

> ⚠️ ✅ **সিকিউরিটি নোট:** ফ্রন্টএন্ড build-এ দেওয়া `VITE_` variable ব্রাউজারে **সবাই দেখতে পায়**। তাই এখানে কখনোই secret key, DB password, private token রাখা যাবে না।

### `FROM nginx:alpine AS runner` — এখানে builder-এর জিনিস পেল কোথায়?

দ্বিতীয় stage সম্পূর্ণ **নতুন ও খালি** — এখানে Node.js নেই, সোর্স কোড নেই, `node_modules` নেই। শুধু NGINX আছে।

তাহলে বিল্ড হওয়া ফাইল আসবে কীভাবে? — `COPY --from=builder` দিয়ে ✅

```dockerfile
COPY --from=builder /app/dist /usr/share/nginx/html
```

মানে: *"builder stage-এর `/app/dist` ফোল্ডারটা নিয়ে এসে এই stage-এর `/usr/share/nginx/html`-এ রাখো।"* এটাই multi-stage build-এর মূল জাদু — শুধু দরকারি ফাইলটুকু তুলে আনা, বাকি ভারী জিনিসপত্র ফেলে দেওয়া।

`--from=builder` না দিলে Docker `/app/dist` খুঁজে পাবে না, কারণ nginx ইমেজে ওই ফোল্ডারের অস্তিত্বই নেই → **build fail** করবে।

### বাকি লাইনগুলো

| লাইন | ব্যাখ্যা |
|---|---|
| `COPY nginx.conf /etc/nginx/conf.d/default.conf` | আমাদের নিজের config দিয়ে NGINX-এর ডিফল্ট config রিপ্লেস করছি (SPA fallback পেতে) |
| `EXPOSE 80` | NGINX ডিফল্টভাবে ৮০ পোর্টে সার্ভ করে |
| `CMD ["nginx", "-g", "daemon off;"]` | NGINX চালু করবে, তবে **foreground**-এ |

### ✅ `daemon off;` কেন লাগে?

স্বাভাবিকভাবে NGINX ব্যাকগ্রাউন্ডে (daemon হিসেবে) চলে। কিন্তু **Docker কন্টেইনার তত্ক্ষণাত্ বেঁচে থাকে যতক্ষণ তার মূল process (PID 1) চলতে থাকে।** NGINX যদি ব্যাকগ্রাউন্ডে চলে যায়, মূল process শেষ হয়ে যাবে → কন্টেইনার সঙ্গে সঙ্গে বন্ধ হয়ে যাবে। তাই `daemon off;` দিয়ে NGINX-কে সামনে (foreground) ধরে রাখা হয়।

### Build ও Run

```bash
# Image build
docker build -f Dockerfile.multi -t react-multi:v1 .

# Container run
docker run -d --name react-multi -p 3002:80 react-multi:v1
```

> 🌐 `http://<server-ip>:3002`

---

## ৬. Single vs Multi-Stage — কোনটা ভালো এবং কেন?

| বিষয় | Single-Stage | Multi-Stage |
|---|---|---|
| **Image size** | বড় (~400–600 MB+) — Node.js, `node_modules`, সোর্স কোড সবই থাকে | ছোট (~25–50 MB) — শুধু NGINX + `dist` ✅ |
| **সোর্স কোড ইমেজে থাকে?** | হ্যাঁ (leak হওয়ার ঝুঁকি) | না ✅ |
| **Web server** | Vite preview server (dev-oriented) | NGINX (production-grade) ✅ |
| **Attack surface** | বড় — npm, Node runtime, বহু প্যাকেজ থাকে | ছোট ✅ |
| **Performance** | তুলনামূলক ধীর | দ্রুত (gzip, caching, static serving-এ NGINX অপ্টিমাইজড) ✅ |
| **Deploy/pull speed** | ধীর (বড় ইমেজ) | দ্রুত ✅ |
| **Dockerfile জটিলতা** | সহজ | একটু বেশি |
| **কোথায় মানানসই** | দ্রুত টেস্ট, শেখার সময়, ছোট experiment | **Production / real-world deployment** ✅ |

### ✅ উপসংহার

**Production-এর জন্য সবসময় Multi-Stage Build।** কারণ ইমেজ অনেক ছোট হয়, সোর্স কোড ও dev dependency ফাইনাল ইমেজে থাকে না (নিরাপদ), এবং NGINX দিয়ে static ফাইল serve করা Vite preview server-এর চেয়ে অনেক দ্রুত ও স্থিতিশীল।

> 💡 **কারণটা এক লাইনে:** Build করার জন্য যা যা লাগে (Node, npm, compiler) আর অ্যাপ **চালানোর** জন্য যা লাগে (শুধু কিছু HTML/CSS/JS ফাইল) — এই দুটো এক জিনিস নয়। Multi-stage সেই দুটোকে আলাদা করে দেয়।

### ✅ বাস্তবে আরেকটা ভালো অভ্যাস: `npm install`-এর বদলে `npm ci`

```dockerfile
RUN npm ci
```

| | `npm install` | `npm ci` |
|---|---|---|
| Lock file | দরকার হলে বদলে ফেলে | হুবহু `package-lock.json` মেনে চলে ✅ |
| গতি | ধীর | দ্রুত ✅ |
| ব্যবহার | লোকাল ডেভেলপমেন্ট | CI/CD ও Docker build ✅ |

এতে "আমার মেশিনে চলে, সার্ভারে চলে না" সমস্যাটা অনেকাংশে দূর হয়।

---

## ৭. কন্টেইনারের ভেতরে ঢোকা ও Cleanup

### কন্টেইনারের ভেতরে shell নিয়ে ঢোকা

```bash
docker exec -it react-single sh
```

| ফ্ল্যাগ | পূর্ণরূপ | মানে |
|---|---|---|
| `exec` | execute | চলমান কন্টেইনারে নতুন একটা কমান্ড চালায় |
| `-i` | `--interactive` | STDIN খোলা রাখে, অর্থাৎ টাইপ করতে পারবে |
| `-t` | `--tty` | একটা terminal তৈরি করে দেয় (prompt দেখা যায়) |
| `sh` | shell | Alpine-ভিত্তিক ইমেজে `bash` থাকে না, তাই `sh` ব্যবহার করি ✅ |

বের হতে:

```bash
exit
```

### Cleanup কমান্ড

```bash
# সব কন্টেইনার (চলমান + বন্ধ) জোর করে মুছে ফেলা
docker rm -f $(docker ps -a -q)

# সব ইমেজ জোর করে মুছে ফেলা
docker rmi -f $(docker images -a -q)
```

| অংশ | মানে |
|---|---|
| `docker ps -a` | `--all` → বন্ধ কন্টেইনার সহ সব দেখায় |
| `-q` | `--quiet` → শুধু ID লিস্ট দেয় |
| `$( ... )` | ভেতরের কমান্ডের আউটপুটকে বাইরের কমান্ডের argument বানায় |
| `-f` | `--force` → চলমান অবস্থাতেও মুছে দেয় |
| `rmi` | **r**e**m**ove **i**mage |

> ⚠️ **সতর্কতা:** এই দুটো কমান্ড **সব কিছু** মুছে ফেলে — অন্য প্রজেক্টের কন্টেইনার/ইমেজও। শেয়ার্ড বা প্রোডাকশন সার্ভারে চালানো যাবে না।

✅ **আরও সহজ ও পরিচিত বিকল্প:**

```bash
docker system prune -a          # অব্যবহৃত container, image, network মুছবে
docker system prune -a --volumes  # সঙ্গে volume-ও মুছবে (সাবধান!)
```

---

## ৮. Docker Compose

### 📖 সংজ্ঞা

**Docker Compose** হলো একটা টুল যা একটা YAML ফাইল (`docker-compose.yaml`) ব্যবহার করে **একাধিক কন্টেইনার একসাথে define, run এবং maintain** করতে সাহায্য করে।

**উদাহরণ ১:** একটা expense tracker অ্যাপে frontend + backend + MongoDB — তিনটা সার্ভিস একটা কমান্ডে (`docker compose up -d`) চালু হয়ে যায়।

**উদাহরণ ২:** WordPress + MySQL — দুটো কন্টেইনার, একটা compose ফাইলে network আর volume সহ পুরোটা সেটআপ।

### কেন দরকার?

ধরা যাক একটা প্রজেক্টে ৪-৫টা সার্ভিস আছে — frontend, backend, database, cache — এবং প্রত্যেকের নিজস্ব Dockerfile আছে। Compose ছাড়া প্রতিবার:

1. প্রতিটা ফোল্ডারে আলাদা করে ঢুকতে হবে
2. আলাদা করে `docker build` করতে হবে
3. আলাদা করে `docker run` করতে হবে, সঙ্গে port, network, volume, env সব ম্যানুয়ালি লিখতে হবে
4. বন্ধ করতে গেলে আবার একটা একটা করে `docker stop`

Compose দিয়ে এই পুরো ব্যাপারটা **একটা ফাইলে লিখে রাখা যায় এবং একটা কমান্ডে চালানো যায়** ✅ — এবং সেই ফাইলটা Git-এ থাকে, তাই টিমের সবাই হুবহু একই environment পায়।

### ইনস্টল ও ভার্সন চেক

```bash
sudo apt install -y docker.io docker-compose-v2
docker compose version
```

> ✅ **নোট:** পুরোনো `docker-compose` (হাইফেন সহ, Python-ভিত্তিক V1) এখন deprecated। এখন `docker compose` (স্পেস সহ, V2 plugin) ব্যবহার করা হয়।

### প্রধান কমান্ডগুলো

```bash
docker compose up -d --build    # build করে ব্যাকগ্রাউন্ডে সব সার্ভিস চালু করবে
docker compose ps               # কোন সার্ভিস চলছে দেখাবে
docker compose logs -f          # সব সার্ভিসের live log
docker compose logs -f backend  # শুধু backend-এর log
docker compose down             # সব কন্টেইনার ও network বন্ধ + মুছে দেবে
docker compose down -v          # সঙ্গে volume-ও মুছবে (ডেটা চলে যাবে ⚠️)
docker compose restart backend  # নির্দিষ্ট সার্ভিস restart
```

| ফ্ল্যাগ | মানে |
|---|---|
| `up` | সার্ভিসগুলো তৈরি করে চালু করবে |
| `-d` | `--detach` → ব্যাকগ্রাউন্ডে চলবে |
| `--build` | ক্যাশ করা ইমেজ ব্যবহার না করে নতুন করে build করবে ✅ কোড বদলালে এটা দিতেই হবে |
| `-f` (logs-এ) | `--follow` → লগ লাইভ দেখাতে থাকবে |
| `-v` (down-এ) | `--volumes` → named volume-ও মুছে দেবে |

---

## ৯. 🧪 রিয়েল প্রজেক্ট: Expense Tracker (Compose + 3 Services)

এখানে একটা পূর্ণাঙ্গ multi-container অ্যাপ — **MongoDB + Node.js backend + React frontend** — যেটা একটাই কমান্ডে চলে।

### প্রজেক্ট স্ট্রাকচার

```
expense-tracker/
├── docker-compose.yaml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│       └── server.js
└── frontend/
    ├── Dockerfile
    ├── package.json
    ├── index.html
    ├── src/
    └── nginx/
        └── default.conf
```

---

### 🗂️ ফাইল ১: `docker-compose.yaml`

```yaml
services:
  mongo:
    image: mongo:7
    container_name: expense-mongo
    restart: unless-stopped
    environment:
      MONGO_INITDB_DATABASE: expense_tracker
    volumes:
      - mongo-data:/data/db
    networks:
      - expense-net

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: expense-backend
    restart: unless-stopped
    environment:
      NODE_ENV: production
      PORT: 5000
      MONGO_URI: mongodb://mongo:27017/expense_tracker
      CORS_ORIGIN: http://localhost
    depends_on:
      - mongo
    networks:
      - expense-net

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: expense-frontend
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - expense-net

volumes:
  mongo-data:

networks:
  expense-net:
    driver: bridge
```

### ⭐ লাইন বাই লাইন ব্যাখ্যা

#### `mongo` সার্ভিস

| লাইন | ব্যাখ্যা |
|---|---|
| `image: mongo:7` | নিজের কোড নেই, তাই Docker Hub থেকে রেডিমেড MongoDB 7 ইমেজ নামাবে (`build` লাগে না) |
| `container_name: expense-mongo` | কন্টেইনারের নির্দিষ্ট নাম, নাহলে Compose নিজে `foldername-mongo-1` টাইপ নাম দেয় |
| `restart: unless-stopped` | crash বা সার্ভার reboot হলে নিজে থেকেই আবার চালু হবে ✅ |
| `MONGO_INITDB_DATABASE: expense_tracker` | কন্টেইনার **প্রথমবার** চালু হওয়ার সময় এই নামে database initialize করে |
| `volumes: - mongo-data:/data/db` | ⭐ MongoDB তার ডেটা `/data/db`-তে লেখে; সেটা `mongo-data` named volume-এ সেভ হবে → **কন্টেইনার মুছলেও ডেটা বাঁচবে** ✅ |
| `networks: - expense-net` | এই কন্টেইনার `expense-net` নেটওয়ার্কে থাকবে |

> ✅ **খেয়াল করো — mongo-তে কোনো `ports:` নেই।** এটা ইচ্ছাকৃত ও সঠিক। DB পোর্ট (27017) বাইরে খুলে দিলে ইন্টারনেট থেকে যে কেউ database-এ কানেক্ট করার চেষ্টা করতে পারে — এটা বড় সিকিউরিটি রিস্ক। Backend তো একই network-এর ভেতর থেকেই `mongo:27017`-এ পৌঁছাতে পারছে, তাই বাইরে খোলার দরকারই নেই। ✅

#### `backend` সার্ভিস

| লাইন | ব্যাখ্যা |
|---|---|
| `build: context: ./backend` | নিজের কোড থেকে ইমেজ বানাবে; `./backend` ফোল্ডারটাই build context |
| `dockerfile: Dockerfile` | ওই ফোল্ডারের কোন Dockerfile ব্যবহার হবে |
| `NODE_ENV: production` | Node/Express-কে production মোডে চালায় — দ্রুত, কম ডিবাগ লগ ✅ |
| `PORT: 5000` | অ্যাপ কোন পোর্টে শুনবে |
| `MONGO_URI: mongodb://mongo:27017/expense_tracker` | ⭐⭐ এখানে `localhost` নয়, **`mongo`** — এটাই compose সার্ভিসের নাম। নিচে বিস্তারিত ব্যাখ্যা |
| `CORS_ORIGIN: http://localhost` | কোন origin থেকে আসা request backend গ্রহণ করবে |
| `depends_on: - mongo` | mongo আগে **শুরু** হবে, তারপর backend |

> ✅ **backend-এও `ports:` নেই কেন?** কারণ ব্রাউজার সরাসরি backend-এ কল করে না। ব্রাউজার শুধু frontend (port 80)-এ যায়, আর frontend-এর NGINX ভেতরে ভেতরে `/api/` request-গুলো backend-এ পাঠিয়ে দেয় (proxy)। এটাকে বলে **reverse proxy pattern** — production-এ এটাই standard ✅
> ⚠️ ডিবাগ করার সময় সরাসরি API টেস্ট করতে চাইলে সাময়িকভাবে `ports: - "5000:5000"` যোগ করা যায়।

#### `frontend` সার্ভিস

| লাইন | ব্যাখ্যা |
|---|---|
| `ports: - "80:80"` | হোস্টের ৮০ (ডিফল্ট HTTP পোর্ট) → কন্টেইনারের ৮০ (NGINX)। তাই ব্রাউজারে শুধু `http://server-ip` লিখলেই হয়, পোর্ট নম্বর লাগে না ✅ |
| `depends_on: - backend` | backend আগে শুরু হবে |

#### নিচের দুটো টপ-লেভেল ব্লক

```yaml
volumes:
  mongo-data:          # named volume ডিক্লেয়ার করা হচ্ছে

networks:
  expense-net:
    driver: bridge     # user-defined bridge network
```

সার্ভিসের ভেতরে `mongo-data` বা `expense-net` ব্যবহার করার আগে এখানে **ডিক্লেয়ার করা বাধ্যতামূলক**, নাহলে Compose error দেবে। ✅

### 🔑 সবচেয়ে গুরুত্বপূর্ণ কনসেপ্ট: সার্ভিসের নামই hostname

```
mongodb://mongo:27017/expense_tracker
          ▲
          └── এটা কোনো domain বা IP নয় — compose সার্ভিসের নাম
```

Docker একই user-defined network-এ থাকা কন্টেইনারদের জন্য **built-in DNS** চালায়। তাই backend যখন `mongo` নামে কানেক্ট করতে চায়, Docker নিজে থেকেই সেটাকে mongo কন্টেইনারের internal IP-তে অনুবাদ করে দেয়। ✅

| ভুল ধারণা | সঠিক |
|---|---|
| `mongodb://localhost:27017` | `mongodb://mongo:27017` ✅ |
| `proxy_pass http://localhost:5000` | `proxy_pass http://backend:5000` ✅ |

কারণ প্রতিটা কন্টেইনারের নিজের আলাদা `localhost` আছে। backend কন্টেইনারের ভেতরে `localhost` মানে **সে নিজে** — mongo নয়।

> ⚠️ **`depends_on`-এর সীমাবদ্ধতা:** এটা শুধু নিশ্চিত করে mongo কন্টেইনার **start** হয়েছে, কিন্তু MongoDB ভেতরে ভেতরে connection নেওয়ার জন্য **ready** কিনা তা নিশ্চিত করে না। তাই বড় প্রজেক্টে `healthcheck` + `condition: service_healthy` ব্যবহার করা হয়, অথবা backend কোডে retry logic রাখা হয় ✅

---

### 🗂️ ফাইল ২: `backend/Dockerfile` (Single-stage)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY src ./src
EXPOSE 5000
CMD ["node", "src/server.js"]
```

| লাইন | ব্যাখ্যা |
|---|---|
| `FROM node:20-alpine` | Node.js 20-এর হালকা Alpine ভার্সন |
| `RUN npm install --omit=dev` | ⭐ শুধু **production dependency** ইনস্টল করবে; `devDependencies` (nodemon, eslint, jest ইত্যাদি) বাদ যাবে ✅ |
| `COPY src ./src` | পুরো ফোল্ডার নয়, শুধু `src` কপি করছে — ইমেজ ছোট ও পরিচ্ছন্ন থাকে ✅ |
| `CMD ["node", "src/server.js"]` | সরাসরি node দিয়ে সার্ভার চালু |

> ✅ **backend-এ multi-stage লাগে না কেন?** কারণ Node.js backend চালাতে Node runtime **রানটাইমেও দরকার** — বাদ দেওয়ার মতো ভারী build tool নেই। কিন্তু frontend-এ Node শুধু build-এর জন্য দরকার, চালানোর জন্য নয় — তাই সেখানে multi-stage কাজে লাগে। ✅
>
> ✅ `--omit=dev` আগে `--production` নামে পরিচিত ছিল; npm 8+ থেকে `--omit=dev` ব্যবহার করা হয়। Production-এ `npm ci --omit=dev` আরও ভালো।
>
> ✅ `CMD ["node", "src/server.js"]` — `npm start`-এর বদলে সরাসরি `node` ব্যবহার করা ভালো, কারণ npm একটা বাড়তি process তৈরি করে যা Docker-এর stop signal (SIGTERM) ঠিকমতো অ্যাপে পৌঁছাতে দেয় না।

---

### 🗂️ ফাইল ৩: `frontend/Dockerfile` (Multi-stage)

```dockerfile
# Stage 1: Build React/Vite app
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY index.html ./index.html
COPY src ./src
RUN npm run build

# Stage 2: Serve static app with NGINX
FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

| লাইন | ব্যাখ্যা |
|---|---|
| `AS builder` | Stage 1-এর নাম — এখানে Node দিয়ে React/Vite build হবে |
| `COPY index.html ./index.html` | Vite-এ `index.html` হলো **entry point** (CRA-র মতো `public/` ফোল্ডারে নয়, root-এ থাকে), তাই আলাদা করে কপি করতে হয় ✅ |
| `RUN npm run build` | `dist/` ফোল্ডারে static HTML/CSS/JS তৈরি হবে |
| `FROM nginx:alpine AS runner` | Stage 2 — সম্পূর্ণ নতুন ও খালি ইমেজ, এখানে Node নেই |
| `COPY --from=builder /app/dist ...` | ⭐ builder stage থেকে শুধু `dist` তুলে এনে NGINX-এর web root-এ রাখছে ✅ |
| `COPY nginx/default.conf ...` | নিজের NGINX config দিয়ে ডিফল্ট config রিপ্লেস |
| `CMD ["nginx","-g","daemon off;"]` | NGINX foreground-এ চলবে, নাহলে কন্টেইনার সাথে সাথে বন্ধ হয়ে যাবে |

> ⚠️ এখানে `npm install` (dev সহ) দরকার, কারণ Vite নিজেই একটা `devDependency` — build করতে সেটা লাগবেই। তাই এখানে `--omit=dev` দেওয়া যাবে না ✅
> কিন্তু চিন্তা নেই — builder stage ফাইনাল ইমেজে থাকেই না, তাই এই ভারী `node_modules` ফাইনাল ইমেজে যাচ্ছে না। **এটাই multi-stage-এর সৌন্দর্য** ✅

---

### 🗂️ ফাইল ৪: `frontend/nginx/default.conf` (Reverse Proxy সহ)

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location /api/ {
        proxy_pass http://backend:5000/api/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### 📖 Reverse Proxy কী?

**সংজ্ঞা:** Reverse Proxy হলো এমন একটা সার্ভার যা ক্লায়েন্টের request নিজে গ্রহণ করে, তারপর পেছনে থাকা আসল সার্ভারে পাঠিয়ে দিয়ে উত্তরটা ক্লায়েন্টকে ফেরত দেয়। ক্লায়েন্ট জানেই না পেছনে কে আছে।

**উদাহরণ ১:** এখানে NGINX `/api/expenses` request পেয়ে সেটা `backend:5000`-এ পাঠায়, ফলে ব্রাউজার মনে করে সবকিছু একই সার্ভার থেকেই আসছে।

**উদাহরণ ২:** একটা সাইটে `/blog` গেলে WordPress সার্ভারে আর `/shop` গেলে অন্য সার্ভারে পাঠানো — ইউজার একটাই domain দেখে।

### ⭐ দুটো `location` ব্লকের কাজ

| Request | কোন ব্লকে যাবে | ফলাফল |
|---|---|---|
| `http://localhost/api/expenses` | `location /api/` | NGINX সেটা `http://backend:5000/api/expenses`-এ ফরওয়ার্ড করবে ✅ |
| `http://localhost/dashboard` | `location /` | ফাইল না পেলে `index.html` ফেরত দেবে → React router বাকিটা সামলাবে ✅ |
| `http://localhost/logo.png` | `location /` | আসল ফাইলটাই সার্ভ হবে |

> ✅ NGINX বেশি **নির্দিষ্ট (specific)** prefix-কে অগ্রাধিকার দেয়। তাই `/api/` দিয়ে শুরু হওয়া request কখনোই `location /`-এ যাবে না।

### `proxy_*` লাইনগুলোর মানে

| লাইন | কাজ |
|---|---|
| `proxy_pass http://backend:5000/api/;` | ⭐ request কোথায় পাঠাবে। `backend` = compose সার্ভিসের নাম ✅ |
| `proxy_http_version 1.1;` | HTTP/1.1 ব্যবহার করবে — keep-alive connection ও WebSocket-এর জন্য দরকার |
| `proxy_set_header Host $host;` | ব্রাউজার যে domain চেয়েছিল সেটা backend-কে জানিয়ে দেয় |
| `proxy_set_header X-Real-IP $remote_addr;` | ইউজারের আসল IP পাঠায় (নাহলে backend শুধু NGINX-এর IP দেখত) |
| `proxy_set_header X-Forwarded-For ...;` | কতগুলো proxy পেরিয়ে এসেছে তার পুরো চেইন |
| `proxy_set_header X-Forwarded-Proto $scheme;` | মূল request `http` না `https` ছিল সেটা জানায় |

✅ এই header গুলো না দিলে backend-এ logging, rate-limiting, বা IP-ভিত্তিক নিরাপত্তা কাজ করবে না — সবার IP একই দেখাবে।

### ✅ এই সেটআপের সবচেয়ে বড় সুবিধা: CORS সমস্যা দূর হয়ে যায়

ব্রাউজারের চোখে **সব কিছু একই origin (`http://localhost`) থেকে আসছে** — frontend-ও, API-ও। ব্রাউজার আলাদা origin দেখে না, তাই CORS error-ই আসে না ✅

আর তাই compose-এ `CORS_ORIGIN: http://localhost` — মানে backend শুধু এই origin-এর request গ্রহণ করবে।

> ⚠️ **প্রোডাকশনে মনে রাখতে হবে:** আসল domain-এ deploy করলে `CORS_ORIGIN` বদলে `https://yourdomain.com` করতে হবে, নাহলে API call block হবে ✅

---

### ▶️ পুরো অ্যাপ চালানো

```bash
# প্রজেক্ট ফোল্ডারে গিয়ে
cd expense-tracker

# build করে ব্যাকগ্রাউন্ডে সব সার্ভিস চালু
docker compose up -d --build

# সব ঠিকঠাক চলছে কিনা দেখা
docker compose ps

# লগ দেখা
docker compose logs -f backend

# ব্রাউজারে: http://<server-ip>

# বন্ধ করা (ডেটা থাকবে)
docker compose down

# বন্ধ + ডেটাও মুছে ফেলা ⚠️
docker compose down -v
```

### 🧭 এই অ্যাপের Working Flow

```
                       ব্রাউজার (ইউজার)
                             │
                             │  http://server-ip        (port 80)
                             ▼
      ┌──────────────────────────────────────────────────────┐
      │      Docker Network: expense-net  (bridge)           │
      │                                                      │
      │   ┌───────────────────────┐                          │
      │   │   expense-frontend    │                          │
      │   │   NGINX : 80          │  ports: 80:80 ✅         │
      │   │                       │                          │
      │   │  /        → dist/     │                          │
      │   │  /api/    → proxy ────┼──┐                       │
      │   └───────────────────────┘  │                       │
      │                              │ http://backend:5000   │
      │                              ▼                       │
      │                   ┌───────────────────────┐          │
      │                   │   expense-backend     │          │
      │                   │   Node.js : 5000      │ (no port)│
      │                   └───────────┬───────────┘          │
      │                               │                      │
      │            mongodb://mongo:27017/expense_tracker      │
      │                               ▼                      │
      │                   ┌───────────────────────┐          │
      │                   │    expense-mongo      │          │
      │                   │    MongoDB : 27017    │ (no port)│
      │                   └───────────┬───────────┘          │
      └───────────────────────────────┼──────────────────────┘
                                      │  /data/db
                                      ▼
                        ┌──────────────────────────┐
                        │  Volume: mongo-data      │
                        │  host machine-এ persist  │
                        └──────────────────────────┘
```

**ধাপে ধাপে:**

1. ইউজার ব্রাউজারে `http://server-ip` হিট করে → হোস্টের ৮০ পোর্ট → frontend কন্টেইনারের NGINX
2. NGINX `dist/`-এর React ফাইল সার্ভ করে (`location /`)
3. React অ্যাপ থেকে `/api/expenses` কল যায় → NGINX সেটা ধরে (`location /api/`) → `http://backend:5000/api/expenses`-এ পাঠায়
4. Backend `mongodb://mongo:27017/expense_tracker` দিয়ে DB-তে কানেক্ট করে
5. Mongo ডেটা লেখে `/data/db`-তে → যা আসলে `mongo-data` **volume**-এ সেভ হয় ✅
6. শুধু frontend-এর পোর্ট বাইরে খোলা; backend ও mongo নেটওয়ার্কের ভেতরে সুরক্ষিত ✅

### 🔒 এই আর্কিটেকচারের নিরাপত্তা-সুবিধা

| বিষয় | সুবিধা |
|---|---|
| শুধু একটা পোর্ট খোলা (80) | Attack surface সবচেয়ে ছোট ✅ |
| DB বাইরে exposed নয় | ইন্টারনেট থেকে database-এ কেউ পৌঁছাতে পারবে না ✅ |
| Backend proxy-র পেছনে | সরাসরি API hit করা যায় না |
| CORS সমস্যা নেই | সব same-origin |
| Volume-এ ডেটা | কন্টেইনার redeploy করলেও ডেটা অক্ষত ✅ |

> ✅ **নোট:** Compose V2-তে ফাইলের শুরুতে `version: "3.8"` লেখাটা আর দরকার নেই — এটা এখন obsolete এবং warning দেয়।

---

## ১০. Docker Network

### 📖 সংজ্ঞা

**Docker Network** হলো একটা virtual network যার মাধ্যমে কন্টেইনাররা একে অপরের সাথে এবং বাইরের জগতের সাথে যোগাযোগ করে। এটা ঠিক করে দেয় — **কোন কন্টেইনার কার সাথে কথা বলতে পারবে**।

**উদাহরণ ১:** `expense-net` নামে একটা network-এ frontend, backend আর mongo রাখলাম — এরা নিজেদের মধ্যে নাম ধরে কথা বলতে পারবে।

**উদাহরণ ২:** আরেকটা প্রজেক্টের কন্টেইনার `blog-net`-এ রাখলাম — সে `expense-net`-এর mongo-তে কোনোভাবেই পৌঁছাতে পারবে না। এটাই **isolation**।

### কেন দরকার?

ধরা যাক সার্ভারে ৩টা আলাদা অ্যাপ চলছে। প্রতিটার জন্য আলাদা network দিলে একটার কন্টেইনার আরেকটার কন্টেইনারের (বিশেষ করে database-এর) সাথে যোগাযোগ করতে পারবে না। এতে —

- 🔒 **নিরাপত্তা:** একটা অ্যাপ hack হলেও অন্য অ্যাপের DB নিরাপদ থাকে
- 🧭 **সহজ যোগাযোগ:** একই network-এ থাকলে IP মনে রাখতে হয় না, **সার্ভিসের নামই hostname** ✅
- 🧹 **পরিচ্ছন্নতা:** প্রতিটা প্রজেক্ট নিজের গণ্ডিতে থাকে

> ✅ **গুরুত্বপূর্ণ পার্থক্য:** ডিফল্ট `bridge` network-এ থাকা কন্টেইনাররা **নাম দিয়ে** একে অপরকে খুঁজে পায় না (শুধু IP দিয়ে পায়)। কিন্তু আমরা নিজে বানানো (user-defined) bridge network-এ automatic DNS resolution পাওয়া যায়। এজন্যই সব সময় নিজের network বানানো best practice। Docker Compose ব্যবহার করলে সে নিজেই একটা user-defined network বানিয়ে দেয়।

### Network-এর ধরন

| Driver | কাজ | কখন ব্যবহার |
|---|---|---|
| `bridge` | ডিফল্ট। একই হোস্টের কন্টেইনারদের জন্য প্রাইভেট নেটওয়ার্ক | সাধারণ সিঙ্গেল-সার্ভার অ্যাপ ✅ |
| `host` | কন্টেইনার সরাসরি হোস্টের নেটওয়ার্ক ব্যবহার করে, isolation থাকে না | সর্বোচ্চ নেটওয়ার্ক পারফরম্যান্স দরকার হলে |
| `none` | কোনো নেটওয়ার্ক নেই, সম্পূর্ণ বিচ্ছিন্ন | নিরাপত্তা-সংবেদনশীল অফলাইন কাজ |
| `overlay` | একাধিক হোস্টের কন্টেইনারদের একসাথে যুক্ত করে | Docker Swarm / multi-server cluster |

### কমান্ডসমূহ

```bash
# Network তৈরি
docker network create nure-net

# সব network দেখা
docker network ls

# কোন কন্টেইনার কোন network-এ আছে, IP কত — বিস্তারিত
docker network inspect nure-net

# কন্টেইনার run করার সময় network-এ যুক্ত করা
docker run -d --name mysql-db --network=nure-net -e MYSQL_ROOT_PASSWORD=secret mysql:8

# চলমান কন্টেইনারকে network-এ যোগ / বিচ্ছিন্ন করা
docker network connect nure-net mysql-db
docker network disconnect nure-net mysql-db

# Network মুছে ফেলা (আগে সব কন্টেইনার disconnect থাকতে হবে)
docker network rm nure-net

# অব্যবহৃত সব network মুছে ফেলা
docker network prune
```

> ✅ `docker run`-এ ইমেজের নাম শেষে দিতেই হয় (উপরে `mysql:8`), নাহলে Docker বুঝবে না কোন ইমেজ থেকে কন্টেইনার বানাতে হবে। আর `mysql` ইমেজ চালাতে `MYSQL_ROOT_PASSWORD` env variable বাধ্যতামূলক, নাহলে কন্টেইনার চালু হয়েই বন্ধ হয়ে যাবে।

## ১১. Docker Volume

### 📖 সংজ্ঞা

**Docker Volume** হলো Docker-এর ম্যানেজ করা একটা স্টোরেজ, যা **হোস্ট মেশিনে** থাকে এবং কন্টেইনারের জীবনকালের সাথে যুক্ত নয়। কন্টেইনার মুছে গেলেও volume-এর ডেটা থেকে যায়।

**উদাহরণ ১:** `mongo-data` volume — MongoDB কন্টেইনার delete করে নতুন করে বানালেও পুরোনো সব ডেটা আগের মতোই পাওয়া যায়।

**উদাহরণ ২:** একটা ফাইল-আপলোড অ্যাপের `uploads` ফোল্ডার volume-এ রাখলে অ্যাপ আপডেট/redeploy করলেও ইউজারের আপলোড করা ছবি হারায় না।

### কেন দরকার?

কন্টেইনারের নিজস্ব filesystem **ephemeral** ✅ (অস্থায়ী) — কন্টেইনার মুছে গেলে ভেতরের সব ডেটাও মুছে যায়। Database-এর ক্ষেত্রে এটা ভয়াবহ: কন্টেইনার একবার remove হলেই সব ইউজার ডেটা শেষ।

তাই ডেটা কন্টেইনারের ভেতরে না রেখে হোস্ট মেশিনের একটা নির্দিষ্ট জায়গায় (volume-এ) রাখি।

### Volume ব্যবহারের কারণ

| # | কারণ | ব্যাখ্যা |
|---|---|---|
| ১ | **Data persistence** | কন্টেইনার stop/delete/update হলেও ডেটা টিকে থাকে ✅ |
| ২ | **Host isolation & performance** | Docker নিজে ম্যানেজ করে; container writable layer-এর তুলনায় I/O দ্রুত |
| ৩ | **Safe data sharing** | একই volume একাধিক কন্টেইনারে mount করে ডেটা শেয়ার করা যায় |
| ৪ | **সহজ Backup ও Restore** ✅ | পুরো volume এক কমান্ডে tar করে ব্যাকআপ/রিস্টোর করা যায় |
| ৫ | **Portability** ✅ | Linux, Windows, macOS — সব জায়গায় একইভাবে কাজ করে, path নিয়ে ঝামেলা নেই |

### Volume vs Bind Mount vs tmpfs

| | **Named Volume** | **Bind Mount** | **tmpfs** |
|---|---|---|---|
| কোথায় থাকে | `/var/lib/docker/volumes/` (Docker ম্যানেজ করে) | হোস্টের যেকোনো ফোল্ডার (তুমি ঠিক করো) | শুধু RAM-এ |
| সিনট্যাক্স | `-v mydata:/data/db` | `-v /home/ubuntu/app:/app` | `--tmpfs /tmp` |
| ব্যবহার | **Production database** ✅ | Development-এ live code reload | সাময়িক/সংবেদনশীল ডেটা |
| সুবিধা | Portable, নিরাপদ, backup সহজ | কোড বদলালে সাথে সাথে দেখা যায় | খুব দ্রুত, ডিস্কে কিছু লেখে না |

### কমান্ডসমূহ

```bash
# Volume তৈরি
docker volume create mydata

# সব volume দেখা
docker volume ls

# বিস্তারিত (কোথায় সেভ হচ্ছে সেটাসহ)
docker volume inspect mydata

# Volume মুছে ফেলা
docker volume rm mydata

# অব্যবহৃত সব volume মুছে ফেলা
docker volume prune

# কন্টেইনারে volume যুক্ত করে run করা
docker run -d --name mongo-db -v mydata:/data/db mongo:7
```

### Volume-এর ভেতরের ডেটা সরাসরি দেখা

```bash
sudo su
cd /var/lib/docker/volumes/<volume_name>/_data
ls -la
exit
```

| কমান্ড | মানে |
|---|---|
| `sudo su` | **s**uper **u**ser হিসেবে শেল চালু করা (এই ফোল্ডার root ছাড়া পড়া যায় না) |
| `/var/lib/docker/volumes/<name>/_data` | ওই volume-এর আসল ডেটা এখানেই থাকে |
| `exit` | root শেল থেকে বেরিয়ে আসা |

> ⚠️ ✅ **সতর্কতা:** এই ফোল্ডারে ঢুকে ফাইল দেখা শেখার জন্য ঠিক আছে, কিন্তু এখানে **সরাসরি ফাইল এডিট/ডিলিট করা উচিত নয়** — database-এর ইন্টারনাল ফাইল নষ্ট হয়ে যেতে পারে। ডেটা দেখতে/বদলাতে হলে কন্টেইনারের ভেতরে ঢুকে DB client দিয়ে করাই সঠিক উপায়।
> (Docker Desktop / macOS / Windows-এ এই path সরাসরি পাওয়া যাবে না, কারণ সেখানে Docker একটা VM-এর ভেতরে চলে।)

### ✅ Volume Backup ও Restore (বাস্তব কাজে খুব দরকারি)

```bash
# Backup: mydata volume-কে backup.tar.gz বানানো
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar czf /backup/backup.tar.gz -C /data .

# Restore: backup থেকে ফেরত আনা
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar xzf /backup/backup.tar.gz -C /data
```

`--rm` মানে কাজ শেষে কন্টেইনারটা নিজে থেকেই মুছে যাবে।

---

## ১২. কন্টেইনারের ভেতরে ঢুকে Database দেখা

```bash
# সাধারণ কন্টেইনারে shell নিয়ে ঢোকা
docker exec -it <container-name> sh

# MongoDB কন্টেইনারে সরাসরি Mongo shell নিয়ে ঢোকা
docker exec -it <container-name> mongosh
```

Mongo shell-এর ভেতরে:

```javascript
show dbs                    // সব database-এর লিস্ট
use expense                 // expense database-এ ঢোকা
show collections            // ওই DB-র সব collection দেখা
db.expenses.find()          // expenses collection-এর সব ডেটা
db.expenses.find().pretty() // সুন্দর ফরম্যাটে দেখা
db.expenses.countDocuments() // কতগুলো ডকুমেন্ট আছে
exit                        // বেরিয়ে আসা
```

> ✅ **খেয়াল রাখো:** `db.<collection_name>.find()` — এখানে ডটের পরে **collection**-এর নাম বসবে, database-এর নাম নয়। Database সিলেক্ট করা হয়ে গেছে `use` কমান্ড দিয়ে, তাই `db` মানেই এখন সেই database।

> ✅ পুরোনো `mongo` shell এখন deprecated; নতুন MongoDB ইমেজে (৬+) `mongosh` ব্যবহার করতে হয়। MySQL হলে `mysql -u root -p`, PostgreSQL হলে `psql -U postgres` — DB অনুযায়ী client আলাদা।

---

## 🎯 এক নজরে মূল পয়েন্ট

| # | পয়েন্ট |
|---|---|
| ১ | `scp -i key.pem file ubuntu@ip:/path/` দিয়ে সার্ভারে ফাইল পাঠানো হয়; key-তে `chmod 400` লাগে ✅ |
| ২ | `.dockerignore` (ছোট হাতের অক্ষরে) build দ্রুত করে, ইমেজ ছোট রাখে ও secret leak ঠেকায় ✅ |
| ৩ | `COPY package*.json ./` আলাদা করে আগে দিলে **layer caching** কাজে লাগে, build অনেক দ্রুত হয় ✅ |
| ৪ | `EXPOSE` শুধু ডকুমেন্টেশন — বাইরে থেকে ঢুকতে `docker run -p host:container` লাগবেই ✅ |
| ৫ | **Stage ≠ Container।** `AS builder` / `AS runner` হলো build stage-এর নাম; শেষ stage-টাই ফাইনাল ইমেজ হয়, একটাই কন্টেইনার চলে ✅ |
| ৬ | `COPY --from=builder /app/dist ...` — এটাই multi-stage-এর মূল কৌশল: শুধু দরকারি ফাইল তুলে আনা ✅ |
| ৭ | **Production-এ সবসময় Multi-stage** — ইমেজ ~১০x ছোট, সোর্স কোড থাকে না, NGINX দ্রুত ও নিরাপদ ✅ |
| ৮ | `nginx.conf`-এর `try_files $uri $uri/ /index.html;` না দিলে SPA-তে page refresh করলে 404 আসে ✅ |
| ৯ | `CMD ["nginx","-g","daemon off;"]` — foreground-এ না চললে কন্টেইনার সাথে সাথে বন্ধ হয়ে যায় ✅ |
| ১০ | `VITE_` env variable ব্রাউজারে দৃশ্যমান — কখনো secret রাখা যাবে না ✅ |
| ১১ | **Docker Compose** = এক YAML ফাইল + এক কমান্ডে multi-container অ্যাপ: `docker compose up -d --build` ✅ |
| ১২ | Compose-এ একই network-এ থাকা সার্ভিসরা **নাম দিয়ে** যোগাযোগ করে (`mongodb://mongo:27017`) — `localhost` নয় ✅ |
| ১৩ | **Network** কন্টেইনারদের যোগাযোগ ও isolation নিয়ন্ত্রণ করে; user-defined bridge-এ automatic DNS পাওয়া যায় ✅ |
| ১৪ | **Volume** ডেটা persist করে — কন্টেইনার মুছলেও DB-র ডেটা থাকে; database মানেই volume ✅ |
| ১৫ | `docker compose down -v` এবং `docker volume prune` ডেটা মুছে ফেলে — সাবধানে ব্যবহার ⚠️ |
| ১৬ | `docker exec -it <name> sh` দিয়ে কন্টেইনারে ঢোকা যায়; Alpine ইমেজে `bash` নেই, `sh` ব্যবহার করতে হয় ✅ |
| ১৭ | Expense Tracker-এ **শুধু frontend-এর `ports: "80:80"`** আছে; backend ও mongo-তে `ports` নেই — তারা network-এর ভেতরে সুরক্ষিত ✅ |
| ১৮ | NGINX-এর `location /api/ { proxy_pass http://backend:5000/api/; }` = **reverse proxy** → CORS সমস্যা দূর হয় এবং API সরাসরি exposed থাকে না ✅ |
| ১৯ | Backend Dockerfile-এ `npm install --omit=dev` → devDependencies বাদ, ইমেজ ছোট। কিন্তু frontend build-এ Vite নিজেই devDependency, তাই সেখানে দেওয়া যাবে না ✅ |
| ২০ | Backend-এ multi-stage লাগে না (Node runtime-এও দরকার), frontend-এ লাগে (Node শুধু build-এ দরকার) ✅ |
| ২১ | `CMD ["node", "src/server.js"]` — `npm start`-এর চেয়ে ভালো, কারণ stop signal সরাসরি অ্যাপে পৌঁছায় ✅ |
| ২২ | Compose-এ `volumes:` ও `networks:` টপ-লেভেলে **ডিক্লেয়ার করা বাধ্যতামূলক**, নাহলে error ✅ |

---

## 📌 দ্রুত রেফারেন্স: কমান্ড চিটশিট

```bash
# Image
docker build -f Dockerfile.multi -t react-multi:v1 .
docker images
docker rmi -f <image-id>

# Container
docker run -d --name react-multi -p 3002:80 react-multi:v1
docker ps                 # চলমান
docker ps -a              # সব
docker logs -f <name>     # লগ দেখা
docker stop <name>
docker rm -f <name>
docker exec -it <name> sh

# Compose
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose down

# Network
docker network create nure-net
docker network ls
docker network inspect nure-net
docker network connect nure-net <container>

# Volume
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker volume rm mydata

# Cleanup
docker system prune -a
```