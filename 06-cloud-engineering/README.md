# ☁️ Cloud Engineering — Master Class (Combined Notes)

> Class 1–3 একসাথে। ✅ = গুরুত্বপূর্ণ পয়েন্ট / মনে রাখা জরুরি।

---

## 📑 Table of Contents

- [Class 1 — Cloud Basics, AWS Global Infrastructure & IAM](#class-1)
  - On-Premises, Hybrid, Cloud Providers
  - Cloud Service Models (IaaS/PaaS/SaaS)
  - Region, AZ, Local Zone, Edge, CloudFront
  - AWS Access (Console/CLI/SDK) + IAM
- [Class 2 — EC2, S3, IaC, Auto Scaling & Load Balancer](#class-2)
  - Service-to-Service Access
  - EC2 তৈরির উপাদান, Launch Template
  - S3, IaC, CloudWatch, ASG + LB
- [Class 3 — Project Deploy (React → API Gateway → Lambda)](#class-3)
  - EC2 + IAM setup, Nginx configuration
  - GitHub CI/CD secrets, Deploy flow
  - Hosting alternatives + Term definitions

---
<a id="class-1"></a>

# ☁️ Cloud Engineering — Master Class 1

> AWS বিষয়ে সুন্দর করে গুছিয়ে সাজানো হয়েছে।

---

## 1. On-Premises (On-Prem) কী?

**On-premises** (সংক্ষেপে *on-prem*) মানে হলো — কোনো company তার নিজের অফিসেই একটা আলাদা **computer room / server room** রাখত। সেখানে থাকত:

- Server, Storage, RAM
- Electricity + backup power
- Fire extinguisher, cooling system, light
- একটা dedicated **IT team**

কোম্পানির সব **data** ওই room-এই রাখা হতো। প্রায় ১০ বছর আগে এটাই ছিল normal practice.

### On-prem-এর সমস্যা (Pain Points)

- কোম্পানি বড় হলে server **slow** হয়ে যেত — IT team-কে এটা সামলাতে হতো
- **Backup** নিতে অনেক সময় লাগত
- Power fail করলে খুবই **risky**
- **Hardware replace** করা কঠিন ও খরচসাপেক্ষ
- সবচেয়ে বড় কথা — **scaling করা ছিল painful ও time-consuming**। নতুন server কিনে, বসিয়ে, configure করতে সপ্তাহ/মাস লেগে যেত

### On-prem-এ কারা কাজ করে (vendors)

> ✅ On-prem জগতের সুপরিচিত vendor-গুলো হলো:
> **VMware, Dell EMC, NetApp, IBM, HPE, Cisco, Nutanix** ইত্যাদি।

---

## 2. Hybrid Model

Hybrid মানে — **কিছু জিনিস on-prem-এ থাকে, আর কিছু জিনিস cloud-এ থাকে।**
অর্থাৎ পুরোপুরি cloud-এ না গিয়ে দুটো mix করে চালানো। (যেমন sensitive data on-prem, বাকি সব cloud-এ)

---

## 3. Cloud Providers & Data Centers

বড় কোম্পানিগুলো (**Azure, AWS, GCP, Alibaba Cloud, DigitalOcean** ইত্যাদি) দূরের বিভিন্ন জায়গায় বিশাল building বানিয়ে সেখানে নিজেরাই networking, device, hardware সব setup করেছে।

- এই building-গুলোকে বলে **Data Center**
- যারা এগুলো বানিয়ে service দেয়, তাদের বলে **Cloud Provider**

সহজ কথায়: তুমি নিজে server room না বানিয়ে, ভাড়ায় ওদের data center ব্যবহার করছ — এটাই cloud.

---

## 4. Cloud Service Models (IaaS, PaaS, SaaS ...)

| Model | মানে | তুমি কতটুকু manage করো | Example |
|-------|------|------------------------|---------|
| **IaaS** | Infrastructure as a Service | সবচেয়ে বেশি (OS, app) | **Amazon EC2**, **Azure Virtual Machines** |
| **PaaS** | Platform as a Service | শুধু তোমার code | **AWS Elastic Beanstalk**, **Heroku** |
| **SaaS** | Software as a Service | প্রায় কিছুই না | **Gmail**, **Microsoft 365** |

**বিস্তারিত:**

- **IaaS (Infrastructure as a Service)** — provider তোমাকে raw infrastructure দেয় (virtual server, storage, network)। OS, software সব তুমি নিজে বসাও ও manage করো।
  *উদাহরণ:* Amazon EC2, Azure Virtual Machines (আরও: DigitalOcean Droplets, Google Compute Engine)

- **PaaS (Platform as a Service)** — infrastructure + platform (OS, runtime, database) provider সামলায়; তুমি শুধু **code deploy** করো।
  *উদাহরণ:* AWS Elastic Beanstalk, Heroku (আরও: Google App Engine, Azure App Service)

- **SaaS (Software as a Service)** — পুরো ready software, browser-এ ব্যবহার করো; কিছু manage করতে হয় না।
  *উদাহরণ:* Gmail, Microsoft 365 (আরও: Dropbox, Salesforce)

> Standard cloud model মূলত তিনটা: **IaaS, PaaS, SaaS**। চতুর্থ একটা common হলো **FaaS (Function as a Service)** — *উদাহরণ:* AWS Lambda, Azure Functions। আর disaster recovery-র জন্য থাকে **DRaaS (Disaster Recovery as a Service)**।

---

## 5. AWS Global Infrastructure

AWS আমাদের তিনটা জিনিস দেয় → **Speed, Agility, Scale.**

### Region

- **Region** = একটা geographic area। যেমন: Singapore, Mumbai, Oregon, N. Virginia।

> ✅ **গুরুত্বপূর্ণ:** *"একটা region-এ minimum ৩টা Availability Zone (AZ) থাকে"*।
>
> সঠিক সম্পর্কটা এমন:
> **Region  >  Availability Zone (AZ)  >  Data Center**
> - একটা region-এ minimum সাধারণত **৩টা AZ** থাকে
> - প্রতিটা **AZ** আবার এক বা একাধিক physical **data center** দিয়ে তৈরি

### Availability Zone কেন isolated?

AZ-গুলো **isolated / fault-tolerant**।

- কারণ: একটা AZ-তে যদি বন্যা/আগুন/সমস্যা হয়, বাকিগুলো ঠিক থাকবে এবং **data safe** থাকবে।
- তাই একই application **৩টা জায়গায়** copy করে রাখা হয় (recovery-র জন্য) → AWS guarantee দিতে পারে তোমার application secure।

### Region select করার সময়

- **N. Virginia (us-east-1)** সাধারণত default থাকে, কিন্তু এটা খুবই **crowded**।
- **Oregon (us-west-2)** তুলনামূলকভাবে একটু **free**।

---

## 6. Local Zones & Edge Locations

- **Local Zone** — Region-কে end-user-দের আরও **কাছে** নিয়ে আসার একটা extension। যেসব app-এ super low latency দরকার (gaming, live streaming, real-time), সেখানে কাজে লাগে।
  *উদাহরণ:* Los Angeles Local Zone — মূল region হয়তো Oregon, কিন্তু LA-র users অনেক কম latency-তে service পায়।

- **Edge Location** — ছোট ছোট point যেখানে content **cache** করে রাখা হয়, users-এর কাছাকাছি। এগুলোকে **PoP (Point of Presence)** ও বলে। এখানে full compute হয় না, মূলত **content delivery** হয়।

---

## 7. CloudFront / CDN / PoP

ধরো তোমার application-এর মূল server **Asia**-তে। এখন অন্য region-এর একজন user দূর থেকে access করলে **slow** লাগবে।

সমাধান → content (image, video, file) user-এর কাছের **edge location**-এ cache করে রাখা। এই পুরো network-টাকে বলে **CDN (Content Delivery Network)**, আর AWS-এর CDN service হলো **CloudFront**।

**সহজ উদাহরণ:**
তুমি Bangladesh থেকে এমন একটা website খুলছ যার server আমেরিকায়। প্রথমবার image load হতে সময় লাগবে, কিন্তু CDN সেটা ঢাকার কাছের edge location-এ cache করে রাখবে — **পরের বার অনেক দ্রুত load হবে।**

---

## 8. AWS Access করার উপায় (৩টা)

1. **AWS Management Console** — browser-based **UI**, login করে ব্যবহার।

2. **AWS CLI (Command Line Interface)** — install করে terminal থেকে command দিয়ে AWS manage করা যায়।
   > ✅ **Important:** CLI দিয়ে তুমি পুরো **AWS account/services** access করো — শুধু "VPC-তে login" নয়।

3. **AWS SDK**
   > ✅ **Important:** SDK মানে **Software Development Kit** — এটা দিয়ে তোমার code (Python / JavaScript / Java ইত্যাদি) থেকে **programmatically** AWS-এর সাথে কাজ করা যায়।

---

## 9. IAM — Identity and Access Management

**IAM** হলো AWS-এর **core** (এবং free) service। এটা দিয়ে ঠিক করা হয় — **কে (identity)** AWS-এর **কোন service/resource**-এ **কতটুকু** access পাবে।

> ✅ **("WAY / WYD" mnemonic গুছিয়ে):**
> IAM মূলত দুটো প্রশ্নের উত্তর দেয় —
> - **Authentication** → *Who are you?* (তুমি কে?)
> - **Authorization** → *What can you do?* (তুমি কী করতে পারবে?)

AWS-এর **200+ service** আছে — যেমন **EC2, VPC, S3, DynamoDB, Lambda, EKS**। দরকার হলে console-এ search করলেই পাওয়া যায়।

### কয়েকটা service মনে রাখার মতো

- **EC2** — virtual server (compute)। frontend/backend/app চালানো যায়।
- **S3** — object storage; অনেকটা external hard drive / folder-এর মতো, file store করে রাখার জন্য।
- **Lambda** — serverless compute; ছোট function/computation চালায় (server নিজে manage করতে হয় না)।

> ✅ **ছোট নোট:** "সব operation Lambda-তেই হবে" বা "EC2 দিনে ২ বার S3-তে backup রাখবেই" — এগুলো **fixed rule নয়**, এগুলো একেকটা **architecture-এর example** মাত্র। কোন service কোথায় use হবে সেটা তোমার design-এর উপর নির্ভর করে।

### IAM-এর মূল components

1. **User** — একজন person বা একটা application, যার **long-term access** দরকার। username, password / access key দিয়ে create করা হয়। (মানুষ বা application — দুটোই হতে পারে, যাতে AWS-এর সাথে interact করতে পারে।)

2. **User Group** — একই ধরনের permission-ওয়ালা user-দের group। যেমন: **Developers, Ops, Testers, Finance**। group-এ permission দিলে সব member পায় → manage করা সহজ (best practice)।

3. **Role** — **temporary permission**। এটা কোনো একজন নির্দিষ্ট user-এর সাথে bound নয়। মূলত **service-to-service** communication-এর জন্য:
   - EC2 → S3 (EC2 তার app/db-র backup রাখবে S3-তে)
   - User frontend-এ click → background-এ Lambda → API compute
   - **External:** GitHub-এ code push → auto ভাবে EC2-তে deploy (CI/CD)

4. **Policy** — একটা **JSON document**, যেটা define করে কোন user/group/role কোন service-এ **Allow** বা **Deny** পাবে। এটা অনেকটা **rule book**-এর মতো।

5. **Identity Provider (IdP)** — external identity দিয়ে AWS access করার ব্যবস্থা (federation)। যেমন **Google, Microsoft Azure AD, SAML, GitHub OIDC** দিয়ে login/access। (তুমি Azure থেকে AWS access/backup করতে চাইলে এটা লাগবে।)

---

## 10. 🛠️ Practical: Console থেকে IAM User তৈরি করা

### Step 1 — Specify user details
- **User name** দাও (যেমন `fahim-admin`)
- **"Provide user access to the AWS Management Console"** — user-কে console UI দিতে চাইলে এই box check করো।

**Console password (এই step-এর ভেতরেই):**
- **Autogenerated password** — AWS নিজেই একটা password বানিয়ে দেবে।
- **Custom password** — তুমি নিজে দেবে।
  - কমপক্ষে **8 characters**
  - uppercase (A–Z) + lowercase (a–z) + number (0–9) + symbol — এর মধ্যে অন্তত ৩ ধরনের mix থাকতে হবে
- ☑ **"Users must create a new password at next sign-in"** — *recommended*। প্রথম login-এ user নিজের password বদলে নেবে।

### Step 2 — Set permissions
তিনটা option:
- ✅ **Add user to group** ← **best practice** (group দিয়ে job function অনুযায়ী permission manage করা সহজ)
- **Copy permissions** — অন্য existing user থেকে সব permission copy
- **Attach policies directly** — সরাসরি policy attach (ছোট / বিশেষ কাজে; best practice হলো group ব্যবহার করা)

### Step 3 — Review and create
সব check করে **Create user** → user তৈরি হয়ে যাবে।

---

*Master Class 1 — শেষ। 🎯*

---
<a id="class-2"></a>

# ☁️ Cloud Engineering — Master Class 2

---

## 1. Service-to-Service Access (Sample Question)

**প্রশ্ন:** একজন user-এর **EC2** machine-এ access আছে। এখন তার **RDS**-এর কিছু data manipulate করা দরকার, কিন্তু তার RDS-এ access নেই। কী করা উচিত?

**উত্তর:** আমরা **service-to-service access** তৈরি করব।

- **IAM**-এর মাধ্যমে EC2-কে একটা **Role** দিয়ে দেব।
- এই role-এর কারণে EC2 নিজে RDS-এর সাথে কথা বলতে পারবে — user-এর সরাসরি RDS access লাগবে না।

একইভাবে **S3 bucket**-এর ক্ষেত্রেও:
- আমরা SSH করে EC2-তে connect করব
- তারপর command চালিয়ে দেখব S3-তে কী কী আছে
- এটা সম্ভব হবে EC2-কে দেওয়া **IAM Role**-এর মাধ্যমে (service-to-service communication)

> ✅ **মূল নিয়ম:** এক service আরেক service-এর সাথে কথা বলবে → **user নয়, IAM Role** ব্যবহার করো।

---

## 2. EC2 তৈরি করতে যা যা লাগে

একটা EC2 instance বানাতে ৬টা জিনিস দরকার:

1. **AMI** — Amazon Machine Image
2. **Instance Type**
3. **Security Group**
4. **Volume** (Storage / EBS)
5. **VPC Subnet**
6. **User Data**

চলো একটা একটা করে বুঝি।

### 2.1 AMI (Amazon Machine Image)

`EC2 → Launch instance → (launch without walkthrough)` → প্রথমে **name**, তারপর **OS** (যেমন Ubuntu) select করলাম।

> ✅ **AMI হলো একটা software template** — এতে থাকে OS + প্রি-কনফিগার করা software। Ubuntu select করলে AWS background-এ একটা ready image দেয় যাতে OS ও দরকারি software বসানো থাকে। (**hardware** ঠিক হয় **Instance Type** থেকে।)

### 2.2 Instance Type

এখানে ঠিক হয় — কত **CPU**, কত **memory**, আর কত **খরচ**।

- উদাহরণ: `t2.nano`, `t3.micro` (ছোট, সস্তা)
- **Production**-এ সাধারণত বড় type লাগে (যেমন ~2 CPU / 8GB RAM)
- **Practice**-এর জন্য `micro` নিলেই চলে

### 2.3 Key Pair

আমাদের computer থেকে instance-এ connect করার জন্য **private key + public key**।

> ✅ Standard key generation: **RSA** এবং **ED25519**

- **Private key** → আমাদের computer-এ download হবে
- **Public key** → AWS-এ থেকে যাবে (পাওয়া যাবে `EC2 → Key Pairs`-এ)

মনে রাখার সহজ সূত্র:
- **Source (যেখান থেকে connect করছি)** → private key (তোমার computer, remote, GitHub, cloud)
- **Destination (যেখানে connect করছি)** → public key (EC2 server)

### 2.4 Security Group

> ✅ যেসব port দরকার নেই, সেগুলো **খুলবে না** — খুললে **security vulnerability** তৈরি হয়।

- সাধারণত চেক করি: **SSH (22)**, **HTTP (80)**, **HTTPS (443)**

### 2.5 Volume / Storage (EBS)

Configure storage-এ একটা space দিতে হবে যেখানে content থাকবে। এটা অনেকটা computer-এর **SSD**-র মতো — এটাই **root volume**।

- AWS-এ এটাকে বলে **EBS (Elastic Block Store)**
- সহজ তুলনা: **S3 = external SSD**, আর **EBS = computer-এর সাথে সরাসরি লাগানো SSD**

**Root volume কেন লাগে?** OS, stateful application, logs, backup রাখতে; database (MongoDB, PostgreSQL, MySQL) install করতে।

> ✅ **EBS হলো Availability Zone (AZ) specific** — একটা EBS volume শুধু একই AZ-এর EC2-তে attach করা যায়। তুলনায় **S3 = regional**। (পাওয়া যাবে `EC2 → Elastic Block Store → Volumes`-এ)

### 2.6 User Data (Advanced Details)

`Advanced details → User data` — এটা optional, কাজ করে অনেকটা **terminal**-এর মতো। এখানে shell script লিখে দেওয়া যায়, যেটা instance boot হওয়ার সময় auto চলে।

```bash
#!/bin/bash
sudo apt update -y
sudo apt install nginx -y
```

তারপর **Launch instance**।

---

## 3. Instance চালু হয়েছে কিনা যাচাই

- Instance page-এ **Running** দেখাচ্ছে কিনা দেখব।
- Nginx চলছে কিনা দেখতে: browser-এ **`http://<public-ip>:80`** দিলে nginx-এর welcome page দেখা যাবে।

---

## 4. Launch Template

একই ধরনের instance আবার লাগলে বারবার হাতে বানানোর দরকার নেই — একটা **template** বানিয়ে রাখো।

**Template বানানো:**
`Instances → instance select → Actions → Image and templates → Create template from instance`
→ template-এর **name** + **version** দাও → **Create launch template**

**Template থেকে instance বানানো:**
`Instances → Launch instances (dropdown) → Launch instance from template`

---

## 5. S3 (Simple Storage Service)

**S3 = Object Storage**, অনেকটা **external hard drive**-এর মতো।

- এখানে **folder-এর সমতুল্য = Bucket**
- ব্যবহার: static website hosting, backup, artifacts, log files, config templates রাখা

### 5.1 Storage Classes

S3-তে একাধিক class আছে; মূল দুটো:

| Class | কেমন | কখন |
|-------|------|-----|
| **S3 Standard (regular)** | দ্রুত, খুব বেশি expensive নয় | নিয়মিত ব্যবহারের data |
| **S3 Glacier** | সস্তা, তুলনামূলক **slow** | পুরনো/legacy/archival data |

- **Glacier** ভালো — যেমন govt office-এর অনেক আগের data, বা কম-লাগা legacy জিনিস আর্কাইভ করতে।

> ✅ S3 Standard-এর **durability = 99.999999999% (11 nines)**, আর **availability ≈ 99.99%**। Glacier retrieval-এ কয়েক মিনিট–ঘণ্টা সময় লাগতে পারে বলেই সেটা তুলনামূলক slow।

### 5.2 Bucket তৈরি ও Options

`S3 → Create bucket`

> ✅ **Bucket name অবশ্যই globally unique** হতে হবে — শুধু তোমার account-এ নয়, **পুরো AWS-জুড়ে** একই নাম আর কেউ ব্যবহার করতে পারবে না।

Create করার সময় গুরুত্বপূর্ণ option-গুলো:

- **Object Ownership / ACLs** — কে object-এর owner হবে তা ঠিক করে (সাধারণত ACL disabled রাখা recommended)
- **Block Public Access** — by default **সব public access block** থাকে (security-র জন্য এটা রাখা ভালো; static site host করলে ইচ্ছে করে খুলতে হয়)
- **Bucket Versioning** — একই file-এর পুরনো version রেখে দেয় (ভুলে overwrite/delete হলে ফিরে পাওয়া যায়)
- **Default encryption** — data at-rest encrypt হয় (SSE-S3 default)
- **Tags** — cost tracking / organization-এর জন্য label
- তারপর **Create bucket**

Bucket-এর ভেতরে **folder** create করা যায়, এবং সেখানে file **upload** করা যায়।

### 5.3 Upload/Download-এর উপায়

- **Manual** — console থেকে upload/download
- **Programmatically** — Python-এর **`boto3`** library দিয়ে

---

## 6. IaC — Infrastructure as Code

Infrastructure (server, storage, networking) হাতে হাতে না বানিয়ে **code দিয়ে define** করা — যাতে সব repeatable, version-controlled ও automated হয়।

- **Terraform** — third-party (HashiCorp), multi-cloud
- **AWS CloudFormation (CF)** — AWS-এর নিজস্ব IaC (Terraform-এর AWS alternative)

> ✅ IaC-এর মূল কাজ: পুরো infrastructure-টা code file-এ লিখে রাখা, তারপর এক command-এ সব provision করা (EC2, S3/Blob, networking সব একসাথে define + create + manage)।

---

## 7. CloudWatch

AWS-এর **monitoring** service — resources (EC2, ইত্যাদি)-এর metrics, logs, alarm দেখা যায় এখানে।

---

## 8. Auto Scaling Group (ASG) & Load Balancer (LB)

হঠাৎ অনেক বেশি traffic এলে আমরা আগে থেকেই **Auto Scaling Group** ও **Load Balancer** design করে রাখি।

### 8.1 Load Balancer

- User আর machine-এর মাঝে একটা **bridge** = **Load Balancer**।
- Traffic সরাসরি machine-এ না গিয়ে **আগে LB-তে** আসে।
- LB তার **algorithm** দেখে ঠিক করে user কোন machine-এ যাবে।

**Target Group:**
> ✅ LB traffic পাঠায় **Target Group**-এর কাছে। Target Group-এর কাজ traffic **distribute** করা (ধরো ৩টা machine-এর মধ্যে), সাধারণত **Round Robin** পদ্ধতিতে।

### 8.2 Auto Scaling Group তৈরি (Step by Step)

`EC2 → (নিচে) Auto Scaling → Auto Scaling Groups → Create Auto Scaling group`

1. **Name** দাও + আগে বানানো **launch template** select করো (বা নতুন বানাও) + **version: Latest** → Next
2. **Network:** VPC = default + Availability Zone থেকে **৩টা zone** select + **Zone distribution: Balanced best effort** → Next
3. **Load Balancer:** Attach a new load balancer + **Application Load Balancer** (কারণ এখানে application চলবে) + name দাও + **Scheme: Internet-facing** + Listeners & routing (আগে বানানো টাই) + VPC Lattice: No
   - **Health checks:** Elastic Load Balancing health check **on** করে দিলে ভালো → Next
4. **Group size:** Desired capacity **1**, Min **1**, Max **4** (প্রয়োজন অনুযায়ী)
   - **Scaling policy:** Target tracking scaling policy → name → **Metric: Average CPU Utilization** → **Target value: 60–70** → **Instance warmup: 10 sec** → Next
5. **Review** → **Create Auto Scaling group**

> ✅ **Login/Access:** এখন সরাসরি instance-এর IP নয়, **Load Balancer-এর URL (DNS name)** ব্যবহার করব।

ASG চালু হওয়ার পর হাতে বানানো extra instance-গুলো থেকে **একটা রেখে বাকিগুলো delete** করা যায় — ASG নিজেই দরকারমতো instance বাড়াবে/কমাবে।

---

## ✅ এক নজরে মূল পয়েন্ট (Class 2)

- Service ↔ Service access = **IAM Role** (user নয়)
- **AMI = software** (OS+apps), **Instance Type = hardware**
- Key pair: **RSA / ED25519**; private key তোমার কাছে, public key server-এ
- **EBS = AZ-specific**, **S3 = regional**
- **Bucket name = globally unique**
- S3 durability **11 nines**, availability **99.99%**
- LB → Target Group → machines (Round Robin)
- ASG: Desired/Min/Max + Target tracking (CPU %) + warmup

---

*Master Class 2 — শেষ। 🎯*

---
<a id="class-3"></a>

# ☁️ Cloud Engineering — Master Class 3 (Project Deploy)

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

### 5.2 কেন ঐ দুটো command চালাতে হয়? (teacher যেখানে আটকেছিল)

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

## ✅ এক নজরে মূল পয়েন্ট (Class 3)

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