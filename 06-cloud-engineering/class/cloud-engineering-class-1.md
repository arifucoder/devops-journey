# ☁️ Cloud Engineering — Master Class 1

> এই নোটটা তোমার নিজের লেখা থেকে গুছিয়ে সাজানো হয়েছে।
> যেখানে ভুল ছিল, সেখানে **✅ সংশোধন** মার্ক দিয়ে ঠিক করে দেওয়া হয়েছে।

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

> ✅ **সংশোধন:** তুমি লিখেছিলে *"netforch, eu, obms, p9, hpe"* — নামগুলো একটু অস্পষ্ট ছিল, তাই exact decode করা যায়নি। On-prem জগতের সুপরিচিত vendor-গুলো হলো:
> **VMware, Dell EMC, NetApp, IBM, HPE, Cisco, Nutanix** ইত্যাদি। (এদের মধ্যে HPE তোমার লেখায় ঠিক ছিল ✅)

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

> ✅ **সংশোধন — RaaS নিয়ে:**
> তুমি IaaS/PaaS/SaaS-এর সাথে **RaaS** লিখেছিলে। কিন্তু **RaaS**-এর সবচেয়ে প্রচলিত মানে হলো **Ransomware as a Service** — এটা একটা **malicious / cybercrime** model, legitimate cloud service model নয়।
>
> Standard cloud model মূলত তিনটা: **IaaS, PaaS, SaaS**। চতুর্থ একটা common হলো **FaaS (Function as a Service)** — *উদাহরণ:* AWS Lambda, Azure Functions। আর disaster recovery-র জন্য থাকে **DRaaS (Disaster Recovery as a Service)**।

---

## 5. AWS Global Infrastructure

AWS আমাদের তিনটা জিনিস দেয় → **Speed, Agility, Scale.**

### Region

- **Region** = একটা geographic area। যেমন: Singapore, Mumbai, Oregon, N. Virginia।

> ✅ **গুরুত্বপূর্ণ সংশোধন:**
> তুমি লিখেছিলে *"একটা region-এ minimum ৩টা discrete data center থাকে"*। আসল term-টা হলো **Availability Zone (AZ)**, data center না।
>
> সঠিক সম্পর্কটা এমন:
> **Region  >  Availability Zone (AZ)  >  Data Center**
> - একটা region-এ minimum সাধারণত **৩টা AZ** থাকে
> - প্রতিটা **AZ** আবার এক বা একাধিক physical **data center** দিয়ে তৈরি

### Availability Zone কেন isolated?

AZ-গুলো **isolated / fault-tolerant** (তুমি "ifartolarance" = *fault tolerance* বলতে চেয়েছিলে ✅)।

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
   > ✅ **সংশোধন:** CLI দিয়ে তুমি পুরো **AWS account/services** access করো — শুধু "VPC-তে login" নয়।

3. **AWS SDK**
   > ✅ **সংশোধন:** SDK মানে **Software Development Kit** — "key" নয়! এটা দিয়ে তোমার code (Python / JavaScript / Java ইত্যাদি) থেকে **programmatically** AWS-এর সাথে কাজ করা যায়।

---

## 9. IAM — Identity and Access Management

**IAM** হলো AWS-এর **core** (এবং free) service। এটা দিয়ে ঠিক করা হয় — **কে (identity)** AWS-এর **কোন service/resource**-এ **কতটুকু** access পাবে।

> ✅ **সংশোধন (তোমার "WAY / WYD" mnemonic গুছিয়ে):**
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

তোমার screenshot অনুযায়ী step-গুলো:

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

## ✅ মূল সংশোধনগুলো এক নজরে

1. **SDK** = Software Development **Kit** (key নয়)
2. Region-এর ভেতরে থাকে **Availability Zone (AZ)**, minimum ৩টা; আর AZ = এক/একাধিক data center। সঠিক ক্রম: **Region > AZ > Data Center**
3. **CLI** দিয়ে পুরো **AWS account** access হয় — শুধু VPC নয়
4. **RaaS** সাধারণত *Ransomware-as-a-Service* (malicious) বোঝায়। Standard cloud model হলো **IaaS / PaaS / SaaS** (+ FaaS)
5. On-prem vendor নামগুলো অস্পষ্ট ছিল — সম্ভাব্য নাম: **VMware, Dell EMC, NetApp, IBM, HPE, Cisco**
6. **EC2 / S3 / Lambda**-এর সম্পর্ক কোনো fixed rule নয়, example architecture মাত্র
7. IAM mnemonic: **Authentication** (Who are you) + **Authorization** (What can you do)

---

*Master Class 1 — শেষ। 🎯*