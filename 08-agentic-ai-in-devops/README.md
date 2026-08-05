# Agentic AI in DevOps, AIOps & App with 100k Loads — Class 1 ও Class 2 (নোট)

> প্রজেক্ট সোর্স কোড: [https://github.com/arifucoder/bongodev-aiops-with-load-balacing-scripts.git](https://github.com/arifucoder/bongodev-aiops-with-load-balacing-scripts.git)

---

## ১. আজকের বিষয়বস্তু (Topics)

আজকে শেখার লক্ষ্য — **কিভাবে একটা E-commerce ওয়েবসাইটকে 1000+ (এমনকি 100,000+) ইউজারের ট্রাফিক handle করার মতো বানানো যায়।**

মূল টপিকগুলো:

1. AI এবং Agentic AI
2. Agentic AI for DevOps
3. AIOps
4. E-commerce app deploy ও monitor করা, এবং 100k user traffic generate করে সেটা tackle করা

✅ **গুরুত্বপূর্ণ:** এই ৪টা টপিক আসলে একটার পর একটা layer — আগে concept (AI/AIOps), তারপর হাতে-কলমে (hands-on) deploy + monitoring + load testing।

---

## ২. Application Architecture

পুরো সিস্টেমটা তিনটা স্তরে (layer) কাজ করে:

```text
AWS Distributed Load Testing (ECS/Fargate load generator containers, k6 script)
        |
        v
EC2 target app (Docker Compose: NGINX + Node.js API + Redis + worker)
        |
        v
Amazon CloudWatch (Metrics, Logs, Dashboard, Alarms, SNS email, Anomaly detection)
        |
        v
AIOps Lab (Incident injection, anomaly detection, log investigation, remediation practice)
```

✅ **কেন এভাবে সাজানো?**
- **EC2 app** আসল প্রোডাক্ট serve করে (product storefront + `/products` API)।
- **CloudWatch** সবকিছু monitor করে — CPU, latency, errors, queue depth ইত্যাদি।
- **AIOps Lab** হলো practice environment, যেখানে ইচ্ছাকৃতভাবে সমস্যা তৈরি করে (inject) সেটা detect ও fix করা শেখানো হয়।

### Folder Structure (প্রজেক্টের ভেতরে কী কী আছে)

| ফোল্ডার | কাজ |
|---|---|
| `app/api/` | Fastify API, product home page |
| `app/worker/` | Redis queue worker (AIOps delay ইনজেক্ট করার জন্য) |
| `app/nginx/` | NGINX reverse proxy |
| `aiops-labs/` | ইন্সট্রাক্টরের AIOps scenario guide |
| `configs/` | CloudWatch Agent config |
| `dlt/` | AWS Distributed Load Testing (DLT)-এর জন্য k6 script টেমপ্লেট |
| `iam/` | EC2 IAM policy |
| `infra/terraform-ec2-minimal/` | (Optional) EC2 Terraform |
| `scripts/` | Setup, deploy, load, SNS, AIOps script গুলো |

---

## ৩. Class 2 — হাতে-কলমে (Hands-on) EC2 তে Deploy করা

> প্রতিটা ধাপে **কী করা হচ্ছে** এবং **কেন করা হচ্ছে** দুটোই লেখা হলো।

### ধাপ ১ — EC2 Instance তৈরি

✅ প্রজেক্টের অফিসিয়াল রেকমেন্ডেশন অনুযায়ী সঠিক configuration:

| Setting | Value |
|---|---|
| AMI | `Ubuntu 24.04 LTS` |
| Instance type | `c7i.xlarge` অথবা `c7i.2xlarge` |
| Disk | `40–50 GB gp3` |
| Public IP | Enabled |
| Region | Oregon (`us-west-2`) |

Security Group (Inbound rule):

| Port | Protocol | Source |
|---|---|---|
| 22 | tcp (SSH) | শুধু তোমার নিজের IP |
| 80 | tcp (HTTP) | তোমার IP বা ক্লাসের IP range |

✅ **কখনোই এগুলো পাবলিকলি expose করা যাবে না:**
- Redis `6379`
- API internal port `3000`
- Docker internal ports

**কেন?** এগুলো internal সার্ভিস — বাইরে থেকে direct access দিলে যে কেউ database/queue-তে ঢুকে ডেটা নষ্ট করতে পারবে। শুধু NGINX (port 80) publicly খোলা থাকবে, বাকি সব তার পেছনে (reverse proxy-এর ভেতরে) সুরক্ষিত থাকবে।

### ধাপ ২ — EC2-তে Connect ও Update করা

নিজের PC থেকে SSH করে EC2-তে connect করে সিস্টেম আপডেট করা হয়:

```bash
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP
sudo apt update && sudo apt upgrade -y
```

### ধাপ ৩ — IAM Role তৈরি করা

IAM থেকে একটা নতুন Role তৈরি করতে হবে যেখানে:
- **Trusted entity** হবে **EC2** (service হিসেবে)
- এর সাথে attach করতে হবে একটা AWS managed policy: **`CloudWatchAgentServerPolicy`** ✅

> ✅ চাইলে `AmazonSSMManagedInstanceCore` পলিসিও attach করা যায় — এতে AWS Systems Manager Session Manager দিয়ে বিনা SSH key-তেও ইনস্ট্যান্সে connect করা যায়, যেটা বাস্তব প্রোডাকশনেও ভালো practice।

### ধাপ ৪ — Role টা EC2 Instance-এ Attach করা

```text
EC2 console → Instance select করো → Actions → Security → Modify IAM role
→ তোমার তৈরি করা role select করো → Update IAM role
```

### ধাপ ৫ — Role-এ Inline Policy যোগ করা

Role-টার ভেতরে একটা **inline policy** যোগ করতে হয় (এই ফাইলটা প্রজেক্টেই আছে: `iam/ec2-instance-role-policy.json`)। এটা EC2 ইনস্ট্যান্সকে অনুমতি দেয়:

| Section (Sid) | কী করতে পারবে |
|---|---|
| `CloudWatchMetrics` | মেট্রিক পাঠানো, alarm/anomaly detector/dashboard তৈরি করা |
| `CloudWatchLogs` | লগ গ্রুপ/স্ট্রিম তৈরি, লগ পাঠানো, Logs Insights query চালানো |
| `Ec2DescribeForCloudWatchAgent` | CloudWatch Agent-কে instance সম্পর্কে তথ্য দেওয়া |
| `SnsEmailAlerts` | SNS টপিক তৈরি করা ও ইমেইল alert পাঠানো |

✅ **সংক্ষেপে:** এই policy না দিলে EC2 নিজে থেকে CloudWatch-এ মেট্রিক/লগ পাঠাতে পারবে না, dashboard-ও বানাতে পারবে না, এমনকি email alert-ও পাঠাতে পারবে না।

### ধাপ ৬ — প্রজেক্ট ফাইল আপলোড ও Setup Script চালানো

নিজের ল্যাপটপ থেকে zip ফাইল EC2-তে পাঠানো হয়, তারপর unzip করে setup script চালানো হয়:

```bash
# ল্যাপটপ থেকে:
scp -i your-key.pem flashscale-dlt-ec2-cloudwatch-aiops.zip ubuntu@EC2_PUBLIC_IP:/tmp/
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP

# EC2-তে:
cd /tmp
unzip flashscale-dlt-ec2-cloudwatch-aiops.zip
cd flashscale-dlt-ec2-cloudwatch
sudo ./scripts/setup-ec2.sh
```

`setup-ec2.sh` স্ক্রিপ্টটা Docker, Docker Compose ইত্যাদি dependency ইনস্টল করে দেয়। ✅ এরপর logout করে আবার login করতে হয় — কারণ Docker group permission (sudo ছাড়া `docker` কমান্ড চালানোর অনুমতি) নতুন session-এই কার্যকর হয়:

```bash
exit
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP
```

### ধাপ ৭ — প্রজেক্ট `/opt` ফোল্ডারে Move করা

✅ সম্পূর্ণ কমান্ড:

```bash
sudo mkdir -p /opt/flashscale-dlt
sudo rsync -a --delete /tmp/flashscale-dlt-ec2-cloudwatch/ /opt/flashscale-dlt/
sudo chown -R ubuntu:ubuntu /opt/flashscale-dlt
cd /opt/flashscale-dlt
```

**কেন `/opt`-এ move করা হয়?** `/tmp` টেম্পোরারি জায়গা (reboot হলে ফাঁকা হয়ে যেতে পারে), তাই প্রোডাকশন-স্টাইল অ্যাপ সাধারণত `/opt`-এ রাখা হয় — এটাই standard Linux convention।

### ধাপ ৮ — Environment Configure করা (`.env`)

```bash
cp .env.example .env
nano .env
```

ন্যূনতম যেসব ভ্যালু বসাতে হবে:

```env
PROJECT_NAME=flashscale-dlt
AWS_REGION=us-west-2
API_REPLICAS=2
WORKER_REPLICAS=1
ALERT_EMAIL=your-email@example.com
AIOPS_ALERT_EMAIL=your-email@example.com
```

✅ `AWS_REGION` অবশ্যই সেই region হতে হবে যেখানে তোমার EC2 ইনস্ট্যান্স চলছে (এখানে Oregon = `us-west-2`)।

### ধাপ ৯ — CloudWatch Setup

```bash
sudo ./scripts/setup-cloudwatch.sh
```

এটা তৈরি করে:
- CloudWatch Log Groups
- CloudWatch Metric Filters
- CloudWatch Dashboard
- CloudWatch Agent config
- AIOps dashboard widgets

দেখার জায়গা: **AWS Console → CloudWatch → Dashboards → flashscale-dlt**

### ধাপ ১০ — SNS Email Alerts Setup

```bash
sudo ./scripts/setup-sns-alerts.sh
```

এরপর নিজের ইমেইলে যাও এবং **Confirm subscription** বাটনে ক্লিক করো — এটা না করলে SNS alert email আসবে না।

এই স্ক্রিপ্টটা request-volume ভিত্তিক alarm তৈরি করে (10k, 20k, 30k ... 100k requests/minute)। ✅ মনে রাখা জরুরি: **এগুলো "request volume" alert, direct virtual-user (VU) alert না।**

✅ SNS ঠিকমতো কাজ করছে কিনা টেস্ট করার কমান্ড:

```bash
./scripts/test-sns-alert.sh
```

### ধাপ ১১ — AIOps Anomaly Alarms Setup ✅

CloudWatch setup-এর পরে এটা চালাতে হয়:

```bash
sudo ./scripts/setup-aiops.sh
```

এটা তৈরি করে এমন alarm গুলো: Traffic anomaly, Latency anomaly, Queue depth anomaly, API 5xx spike, NGINX 5xx spike, Queue depth high, Injected error observed, Worker bottleneck observed।

✅ **খুব গুরুত্বপূর্ণ পয়েন্ট:** Anomaly alarm গুলো কাজ করার জন্য আগে "normal" ট্রাফিকের baseline data দরকার। তাই anomaly detection-এর উপর ভরসা করার আগে ১৫–৩০ মিনিট normal ট্রাফিক চালিয়ে রাখতে হয়:

```bash
sudo APP_DIR=/opt/flashscale-dlt \
  SCHEDULE="*/5 * * * *" \
  VUS=25 \
  DURATION=2m \
  ./scripts/install-periodic-small-load.sh
```

### ধাপ ১২ — App Deploy করা

✅ Deploy কমান্ড:

```bash
./scripts/deploy.sh
```

Container গুলো চালু আছে কিনা যাচাই:

```bash
./scripts/status.sh
```

লোকালি টেস্ট:

```bash
curl http://localhost/health
curl http://localhost/products
curl http://localhost/stats
```

ব্রাউজারে খোলার URL:

| URL | কী দেখায় |
|---|---|
| `http://EC2_PUBLIC_IP/` | Product storefront |
| `http://EC2_PUBLIC_IP/products` | JSON API (আগের মতোই অপরিবর্তিত) |
| `http://EC2_PUBLIC_IP/aiops` | AIOps classroom lab UI |

### ধাপ ১৩ — Small Load Test দিয়ে Pipeline যাচাই করা

✅ Small load test কমান্ড (সাধারণত `sudo` ছাড়াই চলে):

```bash
VUS=25 DURATION=2m ./scripts/run-small-local-load.sh
VUS=100 DURATION=3m ./scripts/run-small-local-load.sh
```

এই সময় CloudWatch-এ যা দেখা উচিত: API request count, NGINX request count, API latency, 5xx errors, CPU, Memory, Queue depth, Orders accepted/processed, AIOps scenario signals।

---

## ৪. 100k Traffic Handle করা — বড় লোড টেস্ট

✅ **গুরুত্বপূর্ণ:** তোমার EC2 মেশিনে k6 লোকাল install করার দরকার নেই। বরং একটা k6 script-কে **প্যাকেজ (zip)** করে **AWS Distributed Load Testing (DLT)** নামের AWS Solution-এ আপলোড করা হয় — এবং সেই AWS solution নিজেই **ECS/Fargate container** চালিয়ে হাজার হাজার virtual user (VU) simulate করে। একটা মাত্র মেশিন থেকে 100k ট্রাফিক জেনারেট করা বাস্তবিকভাবে সম্ভব না — তাই distributed (একাধিক container/মেশিন মিলিয়ে) load generate করতে হয়।

Package তৈরি করার কমান্ড:

```bash
./scripts/package-dlt-k6.sh \
  --base-url http://EC2_PUBLIC_IP \
  --peak-vus 1000 \
  --order-ratio 0.05
```

আউটপুট: `dlt/dist/flashscale-k6-1000vus.zip` — এটা ল্যাপটপে কপি করে **AWS Distributed Load Testing** কনসোলে k6 test হিসেবে আপলোড করতে হয়।

✅ **রেকমেন্ডেড progression (একধাপে একদম 100k-তে না গিয়ে ধাপে ধাপে বাড়াও):**

`1k → 5k → 10k → 25k → 50k → 100k` VUs

> ⚠️ **গুরুত্বপূর্ণ সতর্কতা:** Load test শুধুমাত্র নিজের owned resource-এর ওপর চালানো উচিত — অন্য কারো সার্ভার/ওয়েবসাইটে অনুমতি ছাড়া load test চালানো illegal ও অনৈতিক।

### Traffic বাড়লে কী করতে হবে (Remediation / Scaling)

✅ হঠাৎ traffic বাড়লে remediation প্রক্রিয়া:

```bash
# Queue depth বাড়লে → worker বাড়াও
WORKER_REPLICAS=2 ./scripts/deploy.sh

# API latency ও CPU বাড়লে → API container বাড়াও
API_REPLICAS=4 ./scripts/deploy.sh
```

এরপর আবার scenario রান করে CloudWatch graph compare করলে বোঝা যায় scaling কাজ করলো কিনা।

---

## ৫. গুরুত্বপূর্ণ Concept ও Definitions

| Term | সংজ্ঞা | উদাহরণ |
|---|---|---|
| **Agentic AI** | এমন AI system যেটা শুধু প্রশ্নের উত্তর দেয় না, বরং নিজে থেকে ধাপে ধাপে সিদ্ধান্ত নিয়ে, tool/script ব্যবহার করে একটা কাজ সম্পূর্ণ করতে পারে | 1) Claude Code নিজে script চালিয়ে সার্ভার সেটআপ করে দেওয়া 2) একটা AI agent নিজে থেকে CloudWatch alarm দেখে, root cause খুঁজে, worker scale করে দেওয়া |
| **AIOps** (AI for IT Operations) | Machine learning/AI ব্যবহার করে সার্ভার/অ্যাপ্লিকেশনের anomaly (অস্বাভাবিক আচরণ) detect করা, root cause বের করা এবং incident দ্রুত resolve করার প্র্যাকটিস | 1) CloudWatch Anomaly Detector হঠাৎ latency বেড়ে যাওয়া চিহ্নিত করা 2) Log Insights দিয়ে স্বয়ংক্রিয়ভাবে error pattern খুঁজে বের করা |
| **IAM Role** | AWS-এর একটা identity, যেটা নির্দিষ্ট resource (যেমন EC2)-কে অস্থায়ীভাবে নির্দিষ্ট permission দেয়, কোনো password/key ছাড়াই | 1) EC2-কে CloudWatch-এ মেট্রিক পাঠানোর অনুমতি দেওয়া 2) Lambda function-কে S3 bucket পড়ার অনুমতি দেওয়া |
| **CloudWatch** | AWS-এর monitoring service — মেট্রিক, লগ, ড্যাশবোর্ড ও অ্যালার্মের জন্য ব্যবহৃত হয় | 1) CPU usage কত % সেটা গ্রাফে দেখা 2) 5xx error বেড়ে গেলে email alert পাঠানো |
| **SNS** (Simple Notification Service) | AWS-এর messaging/notification service, যেটা দিয়ে email/SMS alert পাঠানো যায় | 1) Traffic 50k/minute পার হলে email আসা 2) কোনো alarm trigger হলে on-call ইঞ্জিনিয়ারকে SMS যাওয়া |
| **Reverse Proxy (NGINX)** | এমন একটা সার্ভার যেটা ইউজারের request প্রথমে receive করে, তারপর সঠিক internal service-এ পাঠায় — ফলে internal service সরাসরি বাইরে expose হয় না | 1) `/products` request-কে ভেতরের Node.js API-তে পাঠানো 2) একই সময়ে একাধিক API container-এর মধ্যে load balance করা |
| **Message Queue (Redis)** | একটা সিস্টেম যেখানে টাস্ক (যেমন অর্ডার) জমা রাখা হয়, আর worker সেগুলো একটার পর একটা process করে — এতে হঠাৎ অনেক request এলেও app crash করে না | 1) অর্ডার প্লেস হলে queue-তে যোগ হওয়া, worker সেটা প্রসেস করা 2) ইমেইল পাঠানোর কাজ queue-তে রেখে asynchronously সম্পন্ন করা |
| **Worker Service** | Background-এ চলা একটা প্রসেস, যেটা queue থেকে টাস্ক নিয়ে কাজ সম্পন্ন করে | 1) Order processing worker 2) Image resizing worker |
| **VU (Virtual User)** | Load testing-এ একটা "virtual user" মানে একটা simulated ব্যবহারকারী, যে বারবার request পাঠায় — VU সংখ্যা যত বেশি, একসাথে তত বেশি ইউজার simulate হয় | 1) `VUS=100` মানে ১০০ জন ইউজার একসাথে সাইট ব্যবহার করছে এমন simulate করা 2) `VUS=1000` দিয়ে 1000 concurrent user টেস্ট করা |
| **Anomaly Detection Baseline** | CloudWatch-কে "normal" ট্রাফিক প্যাটার্ন শেখানোর জন্য দরকার হওয়া প্রাথমিক ডেটা — এটা ছাড়া anomaly alarm ভুল সিগন্যাল দিতে পারে | 1) ১৫–৩০ মিনিট normal ট্রাফিক চালিয়ে baseline তৈরি করা 2) Baseline ছাড়া হঠাৎ চালু করা alarm false alarm দেওয়া |
| **Auto-scaling / Remediation** | কোনো মেট্রিক (যেমন queue depth, CPU) নির্দিষ্ট সীমা পার হলে resource (worker/API container) সংখ্যা বাড়িয়ে দেওয়ার প্র্যাকটিস | 1) `WORKER_REPLICAS=2` করে worker বাড়ানো 2) `API_REPLICAS=4` করে API container বাড়ানো |

---

## ৬. এক নজরে মূল পয়েন্ট ✅

- ✅ **Agentic AI** মানে AI নিজে থেকে ধাপে ধাপে কাজ সম্পন্ন করা, শুধু উত্তর দেওয়া না।
- ✅ **AIOps** মানে AI/ML দিয়ে সার্ভারের অস্বাভাবিক আচরণ (anomaly) স্বয়ংক্রিয়ভাবে ধরা ও দ্রুত resolve করা।
- ✅ Architecture flow: **AWS Distributed Load Testing → EC2 (NGINX + API + Redis + Worker) → CloudWatch → AIOps Lab**।
- ✅ EC2 সেটআপে সঠিক spec: **Ubuntu 24.04 LTS, c7i.xlarge/c7i.2xlarge, 40–50GB gp3, Oregon (us-west-2)**।
- ✅ Redis (6379), API internal port (3000), এবং Docker internal port — এগুলো **কখনো পাবলিকলি expose করা যাবে না**।
- ✅ IAM Role-এ managed policy `CloudWatchAgentServerPolicy` + custom inline policy (`iam/ec2-instance-role-policy.json`) attach করতে হয়।
- ✅ Setup ক্রম: `setup-ec2.sh` → `/opt`-এ move → `.env` configure → `setup-cloudwatch.sh` → `setup-sns-alerts.sh` (+ email confirm) → `setup-aiops.sh` → `deploy.sh` → small load test।
- ✅ Anomaly alarm ঠিকমতো কাজ করার জন্য আগে **১৫–৩০ মিনিট baseline traffic** দরকার।
- ✅ 100k ট্রাফিক একটা মেশিন থেকে জেনারেট করা যায় না — **AWS Distributed Load Testing (ECS/Fargate)**-এ k6 script প্যাকেজ করে আপলোড করতে হয়, এবং ধাপে ধাপে (1k → 5k → 10k → 25k → 50k → 100k) লোড বাড়াতে হয়।
- ✅ Traffic বাড়লে remediation: Queue backlog হলে **worker বাড়াও** (`WORKER_REPLICAS`), latency/CPU বাড়লে **API container বাড়াও** (`API_REPLICAS`)।
- ⚠️ Load test **শুধু নিজের owned resource-এ** চালাতে হবে — অনুমতি ছাড়া অন্য কারো সিস্টেমে না।