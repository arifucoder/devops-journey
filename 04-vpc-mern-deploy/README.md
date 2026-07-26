# AWS-এ VPC এবং MERN Application Deployment গাইড

> এই নোটে ধাপে ধাপে দেখানো হয়েছে কীভাবে AWS-এ নিজের একটা VPC বানিয়ে, তার ভেতরে EC2 instance চালু করে, MongoDB install করে এবং শেষে frontend ও backend চালানোর জন্য প্রয়োজনীয় port গুলো খুলে দিতে হয়।

---

## ধাপ ১: AWS Console-এ Login

1. AWS Console-এ **login** করুন।
2. উপরের ডান দিকে গিয়ে আপনার পছন্দের **Region** সিলেক্ট করুন (যেমন: `ap-south-1` — Mumbai)।
   - ⚠️ পরের সব কাজ এই একই region-এ করতে হবে, তাই শুরুতেই ঠিকভাবে বেছে নিন।

---

## ধাপ ২: VPC তৈরি করা

1. উপরের search box-এ **`VPC`** লিখে সার্চ করুন এবং VPC service-এ যান।
   > 📝 আগের নোটে ভুলে `vps` লেখা ছিল — সঠিক শব্দটা `VPC` (Virtual Private Cloud)।
2. **Create VPC** বাটনে ক্লিক করুন।
3. উপরে দুটো অপশন থাকবে — **VPC only** এবং **VPC and more**।
   - **VPC and more** সিলেক্ট করুন।
   > এটা সিলেক্ট করলে AWS নিজে থেকেই subnet, route table, internet gateway ইত্যাদি একসাথে বানিয়ে দেয়, তাই আলাদা করে বানানোর ঝামেলা থাকে না।
4. নিচের সেটিংসগুলো এভাবে দিন:

   | Setting | Value |
   |---|---|
   | **Name tag** | পছন্দমতো একটা নাম দিন (যেমন: `mern-vpc`) |
   | **IPv4 CIDR block** | default `/16` রেখে দিন |
   | **Tenancy** | default রাখুন |
   | **Number of AZs** | `2` |
   | **Number of public subnets** | `2` |
   | **Number of private subnets** | `2` |

5. সব ঠিক থাকলে নিচে **Create VPC** বাটনে ক্লিক করুন।

> 💡 **ছোট ব্যাখ্যা:**
> - **CIDR `/16`** মানে প্রায় ৬৫,০০০ IP address পাওয়া যায়।
> - **Public subnet**-এ থাকা instance-এ internet থেকে সরাসরি ঢোকা যায় (এখানেই আমরা আমাদের app-এর EC2 রাখব)।
> - **Private subnet**-এ থাকা instance internet থেকে সরাসরি অ্যাক্সেস করা যায় না (সাধারণত database ইত্যাদির জন্য ব্যবহার হয়)।

---

## ধাপ ৩: EC2 Instance Launch করা

1. Console-এ **EC2** service-এ যান।
2. **Launch Instance** বাটনে ক্লিক করুন।
3. একটা **Name** দিন (যেমন: `mern-server`)।
4. **Application and OS Images**-এ গিয়ে **Ubuntu** সিলেক্ট করুন।
5. **Key pair** তৈরি করুন বা আগের একটা সিলেক্ট করুন (SSH দিয়ে login করার জন্য এটা দরকার হবে)।
6. **Network settings** অংশে **Edit** বাটনে ক্লিক করুন।
   - **VPC** ড্রপডাউনে এসে আমরা যে VPC (`mern-vpc`) বানিয়েছিলাম সেটা সিলেক্ট করুন।
   - **Subnet** হিসেবে একটা **public subnet** বেছে নিন।
   - **Auto-assign public IP** → **Enable** করে দিন (না হলে বাইরে থেকে অ্যাক্সেস করা যাবে না)।
7. **Launch Instance**-এ ক্লিক করুন।

---

## ধাপ ৪: MongoDB Install করা

1. EC2 instance-এ **SSH** দিয়ে connect করুন।
2. MongoDB-এর official documentation অনুসরণ করে Ubuntu-তে install করুন (AWS/Ubuntu-এর জন্য যে ধাপগুলো দেওয়া থাকে সেগুলো ফলো করুন)।
3. Install শেষ হলে MongoDB service চালু আছে কিনা চেক করুন:

   ```bash
   sudo systemctl status mongod
   ```

   - সবুজ রঙে **`active (running)`** দেখালে বুঝবেন MongoDB ঠিকমতো চলছে।
   - যদি running না থাকে, তাহলে চালু করুন:

     ```bash
     sudo systemctl start mongod
     ```

   - Instance restart হলেও যেন MongoDB নিজে থেকে চালু হয়, তার জন্য:

     ```bash
     sudo systemctl enable mongod
     ```

---

## ধাপ ৫: Security Group-এ Port (Inbound Rules) যোগ করা

সব কিছু install হয়ে গেলে, frontend ও backend বাইরে থেকে অ্যাক্সেস করতে দিতে হলে সংশ্লিষ্ট **port** গুলো **Security Group**-এর **Inbound rules**-এ যোগ করতে হবে।

> 📝 আগের নোটে `enbound` লেখা ছিল — সঠিক শব্দটা **`inbound`**।

**করার নিয়ম:** EC2 → আপনার instance সিলেক্ট → **Security** ট্যাব → Security group-এ ক্লিক → **Edit inbound rules** → **Add rule**।

সাধারণভাবে যেসব port যোগ করতে হয়:

| Type | Port | কেন দরকার |
|---|---|---|
| SSH | `22` | server-এ login করার জন্য |
| HTTP | `80` | সাধারণ web traffic |
| HTTPS | `443` | secure web traffic |
| Custom TCP | `3000` | frontend (React) |
| Custom TCP | `5000` | backend (Node/Express) — *আপনার backend যে port-এ চলে* |

> ⚠️ **নিরাপত্তা টিপস:** MongoDB-এর port (`27017`) সাধারণত `0.0.0.0/0` দিয়ে সবার জন্য খুলে দেওয়া উচিত নয়, কারণ এতে database অরক্ষিত হয়ে যায়। সম্ভব হলে MongoDB শুধু server-এর ভেতর থেকেই (`localhost`) অ্যাক্সেস করুন।

- **Source** হিসেবে সাধারণত `0.0.0.0/0` (সবার জন্য) দেওয়া হয়, তবে নিরাপত্তার জন্য নির্দিষ্ট IP দেওয়া ভালো।
- শেষে **Save rules**-এ ক্লিক করুন।

---

## সংক্ষিপ্ত ধাপ (Quick Recap)

1. AWS login → Region সিলেক্ট
2. `VPC` সার্চ → **VPC and more** দিয়ে VPC তৈরি (2 AZ, 2 public + 2 private subnet)
3. EC2 → Launch → Ubuntu → Network settings-এ নিজের VPC সিলেক্ট → Launch
4. SSH → MongoDB install → `sudo systemctl status mongod` দিয়ে চেক
5. Security group-এর **inbound rules**-এ frontend ও backend-এর port যোগ

---

*✅ এই ধাপগুলো ঠিকমতো ফলো করলে আপনার MERN application AWS-এ deploy হয়ে যাবে।*