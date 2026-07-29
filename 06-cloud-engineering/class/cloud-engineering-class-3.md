# ☁️ Cloud Engineering — Master Class 3 (Project Deploy)

> ✅ = গুরুত্বপূর্ণ পয়েন্ট / মনে রাখা জরুরি।

---

## 🎯 প্রজেক্ট Overview

এই class-এ আমরা একটা full project (**cloudmentor**) deploy করব। Architecture flow:

```
React (frontend)  →  API Gateway  →  Lambda  →  S3  →  OpenAI
                                        │
                                        ├──→ DynamoDB (data store)
                                        └──→ CloudWatch (logs)
```

> ✅ দুটো আলাদা অংশ আছে:
> - **Frontend** → EC2-তে nginx দিয়ে host হয় (static files থাকে `/var/www/cloudmentor`-এ)
> - **Backend** → serverless (API Gateway + Lambda + DynamoDB), যেটা **SAM** দিয়ে deploy হয়
>
> এই কারণেই পরে দুই ধরনের secret লাগবে — একটা set EC2-তে frontend push করার জন্য, আরেকটা set serverless backend deploy করার জন্য।

---

## 1. EC2 Machine তৈরি

`EC2 → Launch instance`

| Setting | Value |
|---------|-------|
| Name | cloudmentor |
| OS (AMI) | Ubuntu 24.04 LTS |
| Instance type | `t2.large` (2 vCPU + 8 GiB memory) |
| Key pair | নতুন একটা তৈরি করো (`.pem` download হবে) |
| Storage | 10 GiB SSD (root EBS volume) |
| Security group | HTTP (80), HTTPS (443), SSH (22) allow |

→ **Launch instance**

> ✅ **Security group নিয়ে সতর্কতা:** শুধু দরকারি port খোলো — **HTTP, HTTPS, SSH**। Database port (যেমন MySQL 3306 / MSSQL 1433) কখনো `0.0.0.0/0`-তে (পুরো ইন্টারনেটে) খুলবে না — এটা বড় security risk। এই project-এ data থাকবে **DynamoDB**-তে (managed), তাই EC2-তে কোনো DB port খোলার দরকারই নেই।

---

## 2. IAM User + Access Key

`IAM → Users → Create user`

```
Create user → name দাও → Next
→ Attach policies directly → AdministratorAccess → Next → Create user
```

তারপর user-এর ভেতরে গিয়ে:

```
Create access key → CLI → confirmation checkbox → Next → Create access key
```

> ✅ এই **Access Key ID** ও **Secret Access Key** পরে GitHub secret-এ লাগবে — সেভ করে রাখো (Secret key একবারই দেখা যায়)।
>
> ✅ শেখার জন্য `AdministratorAccess` ঠিক আছে, কিন্তু real production-এ **least privilege** (শুধু যতটুকু access দরকার ততটুকু) দেওয়াই best practice।

---

## 3. EC2-তে Connect + Nginx Install

Terminal থেকে (তোমার PC → instance) SSH করে connect করো। এরপর nginx বসাও:

```bash
sudo apt update
sudo apt install -y nginx rsync curl unzip git
sudo systemctl enable nginx      # boot-এ auto start
sudo systemctl start nginx       # এখনই চালু
sudo systemctl status nginx      # চলছে কিনা check
```

> ✅ `enable` = রিবুট হলেও nginx নিজে চালু হবে। `start` = এই মুহূর্তে চালু করে। দুটো একসাথে করতে চাইলে `sudo systemctl enable --now nginx`।

---

## 4. Application Folders তৈরি

সাধারণত application-এর file আমরা `/opt`-এ রাখি, আর nginx-এর static frontend `/var/www`-এ থাকে।

```bash
sudo mkdir -p /opt/cloudmentor
sudo mkdir -p /var/www/cloudmentor

sudo chown -R ubuntu:ubuntu /opt/cloudmentor
sudo chown -R ubuntu:ubuntu /var/www/cloudmentor
```

> ✅ `-p` flag-এর কাজ: parent folder না থাকলে সেটা আগে বানিয়ে নেবে, তারপর ভেতরের folder বানাবে। থাকলে error না দিয়ে চুপচাপ কাজ করে।
>
> ✅ `chown -R ubuntu:ubuntu` = folder-এর ownership `ubuntu` user-কে দেয়, যাতে root ছাড়াই file লেখা/মোছা যায় (`-R` = ভেতরের সব file/folder সহ)।

---

## 5. Nginx Configuration (সঠিক ধাপে)

### 5.1 আগে বুঝি: `sites-available` vs `sites-enabled`

Ubuntu-তে nginx দুটো folder ব্যবহার করে:

| Folder | কাজ |
|--------|-----|
| `/etc/nginx/sites-available/` | এখানে **আসল config file** লেখা হয় (সব site-এর) |
| `/etc/nginx/sites-enabled/` | এখানে শুধু **symlink (shortcut)** থাকে — যে site গুলো **active** |

> ✅ **নিয়ম:** আসল file থাকবে `sites-available`-এ, আর সেটাকে **enable** করতে `sites-enabled`-এ একটা symlink বানাতে হয়। এতে site বন্ধ করতে চাইলে শুধু symlink মুছলেই হয় — মূল config নিরাপদ থাকে।

### 5.2 কেন ঐ দুটো command চালাতে হয়? (তোমার teacher যেখানে আটকেছিল)

Config লেখার পর দুটো command লাগে:

```bash
# ১) ভুল জায়গায় বসে যাওয়া link পরিষ্কার করা (থাকলে)
sudo rm /etc/nginx/sites-available/cloudmentor

# ২) সঠিকভাবে sites-enabled-এ symlink বানানো
sudo ln -s /etc/nginx/sites-available/cloudmentor /etc/nginx/sites-enabled/
```

**কেন?**
- nginx active config পড়ে **`sites-enabled`** থেকে। তাই তোমার site চালু করতে ওখানে একটা symlink থাকতেই হবে।
- সমস্যা হয় যখন `ln -s`-এর **direction উল্টো** হয়ে যায়। সঠিক syntax:

  ```bash
  sudo ln -s <SOURCE: আসল file>  <DESTINATION: যেখানে shortcut বসবে>
  ```
  অর্থাৎ **source = `sites-available/cloudmentor`**, **destination = `sites-enabled/`**।
- ভুল দিকে link বানালে `sites-available`-এই একটা আজেবাজে link তৈরি হয় → তখন প্রথম command (`rm`) দিয়ে সেটা মুছে, দ্বিতীয় command দিয়ে ঠিকভাবে link বানাতে হয়।

> ✅ যদি তুমি শুরু থেকেই `sites-available`-এ **আসল file** (vim/nano দিয়ে) বানাও, তাহলে প্রথম `rm` command লাগবেই না — সরাসরি দ্বিতীয় command (`ln -s`) চালালেই হবে।

### 5.3 সম্পূর্ণ সঠিক ক্রম (এটাই ফলো করো)

```bash
# 1. আসল config file বানাও sites-available-এ
sudo vim /etc/nginx/sites-available/cloudmentor
# (নিচের config paste করো, তারপর save)

# 2. site enable করো (sites-enabled-এ symlink)
sudo ln -s /etc/nginx/sites-available/cloudmentor /etc/nginx/sites-enabled/

# 3. config-এ ভুল আছে কিনা test করো
sudo nginx -t

# 4. এখন default site বন্ধ করো (symlink মুছবে, আসল file থেকে যাবে)
sudo rm /etc/nginx/sites-enabled/default

# 5. আবার test করে reload করো
sudo nginx -t
sudo systemctl reload nginx
```

**Config file (cloudmentor):**

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;   # IPv6 support

    server_name _;

    root /var/www/cloudmentor;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### 5.4 default কখন delete করব? আর 403 কেন?

> ✅ **default site সবার আগে delete করো না।** আগে নিজের site enable করে `nginx -t` দিয়ে ঠিক আছে নিশ্চিত করো, তারপর default মোছো। এতে কিছু ভাঙলেও একটা working default থাকবে।
>
> ✅ browser-এ public IP + `:80` হিট করলে **403 Forbidden** আসা স্বাভাবিক — কারণ `/var/www/cloudmentor`-এ এখনো `index.html` নেই। পরে GitHub Actions pipeline যখন frontend build করে ফেলবে, তখন content চলে আসবে আর page দেখা যাবে। অর্থাৎ 403 মানে "nginx ঠিক আছে, শুধু content এখনো deploy হয়নি"।

`sudo nginx -t` দিয়ে সবসময় reload-এর আগে config test করার অভ্যাস রাখো।

---

## 6. GitHub Repo + CI/CD Secrets

GitHub-এ repo তৈরি করে local `origin` বদলে ঐ repo-র URL দাও, তারপর code push করো।

Workflow-এর YAML file যেসব **secret** ব্যবহার করে, সেগুলো এখানে দিতে হবে:

```
Repo → Settings → Secrets and variables → Actions → New repository secret
→ Name + Secret দাও
```

| Secret Name | Value / কোথা থেকে |
|-------------|-------------------|
| `AWS_ACCESS_KEY_ID` | IAM থেকে |
| `AWS_SECRET_ACCESS_KEY` | IAM থেকে |
| `AWS_REGION` | যে region-এ instance ও Lambda আছে |
| `SAM_STACK_NAME` | যেকোনো নাম (deploy হওয়া stack-এর নাম) |
| `OPENAI_API_KEY` | খালি string — `""` (mock mode বলে) |
| `OPENAI_MODEL` | `gpt-4.1` (বা যেকোনো model) |
| `AI_MODE` | `mock` |
| `CORS_ORIGIN` | public IP-এর URL |
| `EC2_HOST` | instance-এর IP |
| `EC2_USER` | `ubuntu` |
| `EC2_APP_DIR` | `/opt/cloudmentor` |
| `EC2_SSH_KEY` | download করা `.pem` key-এর পুরো content |

> ✅ **`AI_MODE = mock`** মানে আসল OpenAI API call হবে না — একটা fake/simulated উত্তর আসবে। এতে খরচ নেই এবং আসল API key reveal করতে হয় না, তাই শেখার জন্য নিরাপদ। pipeline/flow ঠিকঠাক কাজ করে, শুধু AI-এর জায়গায় dummy response আসে। এজন্যই `OPENAI_API_KEY` খালি (`""`) রাখা যায়।
>
> ✅ **`EC2_SSH_KEY` দেওয়ার নিয়ম:** `.pem` file-এর **পুরো content** কপি করো —
> - শুরু ও শেষ হবে `-----BEGIN ... KEY-----` / `-----END ... KEY-----` (৫টা করে dash সহ)
> - সামনে/পরে কোনো **extra whitespace** থাকা যাবে না
> - terminal-এ শেষে যদি একটা `%` দেখায় (zsh-এর marker), সেটা কপি করো না

---

## 7. Deploy + কোন জিনিস কোথায় যায়

সব secret দেওয়ার পর আবার code **push** করো → workflow-এর script-গুলো চলবে → Lambda সহ সবকিছু install হয়ে app দেখা যাবে।

| জিনিস | কোথায় যায় |
|-------|-----------|
| Frontend (static files) | `/var/www/cloudmentor` (nginx serve করে) |
| App থেকে upload করা file | **S3 bucket** |
| Lambda-এর variables | AI mode, CORS origin, materials bucket, OpenAI key & model, storage mode, table name |
| Logs | **CloudWatch** |
| Data store | **DynamoDB** |

---

## 8. Hosting-এর Alternative উপায়

- **AWS Amplify** — frontend host করার সবচেয়ে সহজ উপায় (auto build + CI/CD)। teacher বলেছেন এটা try করতে।
- **S3 + CloudFront** — static site S3-তে রেখে CloudFront (CDN) দিয়ে দ্রুত global delivery।

---

## 📖 দরকারি Term-এর সংজ্ঞা

**Symlink (Symbolic Link)** — একটা shortcut/pointer যা অন্য file বা folder-এর দিকে ইশারা করে; আসল file একটাই থাকে।
- Example 1: `sites-enabled/cloudmentor` → `sites-available/cloudmentor`-এর দিকে link।
- Example 2: Windows desktop-এ কোনো app-এর "shortcut" icon।

**API Gateway** — AWS-এর managed service যা client-এর request নিয়ে সঠিক backend (যেমন Lambda)-এ পাঠায়; এটা API-এর "front door"।
- Example 1: React app `POST /chat` → API Gateway → Lambda।
- Example 2: Mobile app `GET /materials` → API Gateway → Lambda → DynamoDB।

**CORS (Cross-Origin Resource Sharing)** — browser security নিয়ম, যা ঠিক করে এক origin (domain)-এর page অন্য origin-এর API call করতে পারবে কিনা।
- Example 1: `http://<public-ip>` frontend থেকে API call allow করতে `CORS_ORIGIN`-এ ঐ URL দেওয়া।
- Example 2: `https://myapp.com` (frontend) → `https://api.myapp.com` (backend) call করতে backend-এ CORS allow করা।

**SAM (Serverless Application Model)** — AWS-এর framework, যা দিয়ে Lambda/API Gateway/DynamoDB-র মতো serverless resource code (`template.yaml`) দিয়ে define ও deploy করা যায় (CloudFormation-ভিত্তিক)।
- Example 1: `sam build && sam deploy` দিয়ে Lambda + API Gateway একসাথে deploy।
- Example 2: `SAM_STACK_NAME` = deploy হওয়া stack-এর নাম।

**DynamoDB** — AWS-এর fully managed NoSQL database (key-value / document), দ্রুত ও scalable।
- Example 1: chat history / user data রাখা।
- Example 2: session বা metadata table রাখা।

**AWS Amplify** — frontend web/mobile app দ্রুত build, deploy ও host করার managed platform (CI/CD সহ)।
- Example 1: React app GitHub-এ push করলে Amplify auto build + host করে।
- Example 2: static frontend + backend একসাথে Amplify দিয়ে deploy।

---

## ✅ এক নজরে মূল পয়েন্ট

- Architecture: **React → API Gateway → Lambda → S3 → OpenAI** (+ DynamoDB, CloudWatch)
- Frontend = EC2/nginx (`/var/www/cloudmentor`); Backend = serverless (SAM deploy)
- Security group-এ শুধু **HTTP/HTTPS/SSH** — DB port public-এ কখনো না
- Nginx নিয়ম: **file → `sites-available`**, **symlink → `sites-enabled`**; `ln -s SOURCE DESTINATION`
- **default আগে মুছো না**; নিজের site enable + test করে তারপর মোছো
- **403 Forbidden = স্বাভাবিক** যতক্ষণ `index.html` deploy হয়নি
- reload-এর আগে সবসময় **`sudo nginx -t`**
- `AI_MODE = mock` → খরচ নেই, key reveal নেই; `OPENAI_API_KEY = ""`
- `EC2_SSH_KEY` = পুরো `.pem` content, কোনো extra whitespace/`%` ছাড়া
- Hosting-এর সহজ বিকল্প: **AWS Amplify** বা **S3 + CloudFront**

---

*Master Class 3 — শেষ। 🎯*