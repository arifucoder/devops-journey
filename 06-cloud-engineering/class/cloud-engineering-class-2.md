# ☁️ Cloud Engineering — Master Class 2

> ✅ = গুরুত্বপূর্ণ পয়েন্ট, মনে রাখা জরুরি।

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

## ✅ এক নজরে মূল পয়েন্ট

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