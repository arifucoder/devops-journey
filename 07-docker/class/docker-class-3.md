# 🐳 Docker — Class 3 (ক্লাস নোটস)

> বিষয়: MySQL container, image rebuild, image delete, Docker Hub-এ push, এবং DevSecOps + Docker Scout দিয়ে security scanning।

---

## ১. MySQL Container চালানো

```bash
docker run -d \
  --name mysql-db \
  --network=my-network \
  -v mysql_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=nure123 \
  -e MYSQL_DATABASE=class-db \
  -p 3306:3306 \
  mysql:latest
```

### কমান্ডের প্রতিটি অংশের ব্যাখ্যা

| অংশ | কাজ |
|---|---|
| `-d` | Detached mode — container background-এ চলবে, terminal আটকে থাকবে না |
| `--name mysql-db` | Container-এর একটা পড়ার মতো নাম, পরে id-র বদলে এই নাম ব্যবহার করা যাবে |
| `--network=my-network` | Custom bridge network-এ যুক্ত করা, যাতে অন্য container নাম দিয়ে connect করতে পারে |
| `-v mysql_data:/var/lib/mysql` | Named volume — database-এর data container-এর বাইরে persist করে |
| `-e MYSQL_ROOT_PASSWORD=nure123` | root user-এর password (MySQL image-এর জন্য বাধ্যতামূলক) |
| `-e MYSQL_DATABASE=class-db` | Container প্রথমবার চালু হওয়ার সময় স্বয়ংক্রিয়ভাবে এই নামে একটি database তৈরি হবে |
| `-p 3306:3306` | Host-এর 3306 port → container-এর 3306 port-এ map করা |
| `mysql:latest` | কোন image থেকে container বানানো হবে |

### ✅ যে বিষয়গুলো আগে থেকে নিশ্চিত করতে হবে

✅ **`--network=my-network` ব্যবহার করার আগে network টি তৈরি করা থাকতে হবে**, না থাকলে Docker error দেবে:

```bash
docker network create my-network
docker network ls
```

✅ **Named volume আলাদা করে তৈরি করার দরকার নেই** — `-v mysql_data:/var/lib/mysql` লিখলে Docker নিজেই `mysql_data` volume তৈরি করে নেয়। যাচাই করতে:

```bash
docker volume ls
docker volume inspect mysql_data
```

✅ **Container ডিলিট করলেও volume-এর data থাকে**। তাই container মুছে আবার নতুন করে run করলেও আগের database ফেরত পাওয়া যায়। Volume সহ মুছতে চাইলে আলাদা করে `docker volume rm mysql_data` দিতে হয়।

✅ **Host-এ যদি আগে থেকে MySQL চলে**, তাহলে 3306 port দখল হয়ে থাকবে এবং container start হবে না। তখন host port বদলে নিতে হবে, যেমন `-p 3307:3306`।

### Container-এর ভেতরে ঢুকে MySQL ব্যবহার

```bash
docker exec -it mysql-db mysql -u root -p
```

```sql
SHOW DATABASES;
USE class-db;
```

---

## ২. Container-এ সমস্যা হলে: Debug ও Inspection

```bash
# container-এর log দেখা
docker logs <container_id_or_name>

# live log (নতুন log আসতে থাকলে দেখাবে)
docker logs -f mysql-db

# শেষ ৫০ লাইন log
docker logs --tail 50 mysql-db
```

✅ `docker logs` **container-এর ভেতরের application-এর output/error** দেখায় (যেমন MySQL কেন crash করল)।
✅ কিন্তু **configuration** (IP, volume, network, env) দেখতে হলে `docker inspect` লাগে — দুটো আলাদা কাজ:

```bash
docker inspect mysql-db
docker ps -a        # বন্ধ হয়ে যাওয়া container-ও দেখা যায়
docker stats        # CPU / RAM ব্যবহার
```

> 💡 Container বারবার restart নিচ্ছে বা exit করছে? — প্রথমেই `docker ps -a` দিয়ে exit code দেখুন, তারপর `docker logs` দিয়ে কারণ খুঁজুন।

---

## ৩. Code/Content আপডেট করলে কী করতে হবে

HTML, CSS, JS বা যেকোনো source code পরিবর্তন করলে **শুধু container restart করলে হবে না** — কারণ image তৈরি হওয়ার সময়ই code image-এর ভেতরে copy হয়ে যায়।

### ✅ সঠিক ধাপ (rebuild → remove old → run new)

```bash
# ১. নতুন করে image build করা (নতুন tag দিয়ে)
docker build -t simple-docker-app:v2 .

# ২. পুরনো container বন্ধ করে মুছে ফেলা
docker stop simple-nure
docker rm simple-nure

# ৩. নতুন image দিয়ে container চালানো
docker run -d --name simple-nure -p 8080:80 simple-docker-app:v2
```

✅ প্রতিবার নতুন version-এ **নতুন tag** (`v1`, `v2`, `v3`) দেওয়াই ভালো অভ্যাস — এতে সমস্যা হলে আগের version-এ সহজে ফিরে যাওয়া যায় (rollback)।

> 💡 development-এর সময় বারবার rebuild এড়াতে bind mount ব্যবহার করা যায়:
> `docker run -d -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx:latest`
> এতে file পরিবর্তন করলেই সরাসরি container-এ দেখা যায়। তবে **production-এ সবসময় rebuild-ই করতে হবে।**

---

## ৪. Docker Image ডিলিট করা

```bash
# একাধিক image একসাথে ডিলিট
docker rmi <image-id-1> <image-id-2> -f

# নাম:tag দিয়েও ডিলিট করা যায়
docker rmi simple-docker-app:v1 -f
```

| কমান্ড | কাজ |
|---|---|
| `docker images` | সব image-এর তালিকা |
| `docker rmi <id>` | Image ডিলিট |
| `-f` | Force — container ব্যবহার করছে এমন image-ও জোর করে মুছে দেয় |
| `docker image prune -a` | ব্যবহার না হওয়া সব image একসাথে মুছে ফেলা |
| `docker system prune -a` | Unused image + container + network একসাথে পরিষ্কার (disk খালি করার জন্য) |

✅ `-f` ছাড়া image মুছতে গেলে যদি সেটি কোনো container ব্যবহার করে, Docker error দেবে। নিরাপদ উপায় হলো আগে container মুছে তারপর image মোছা।

---

## ৫. Docker Hub-এ Image আপলোড (Push)

Docker Hub-এ image push করার জন্য **আপনার Docker Hub username অত্যন্ত গুরুত্বপূর্ণ**, কারণ image-এর নাম অবশ্যই এই format-এ হতে হবে:

```
<dockerhub-username>/<image-name>:<tag>
```

### ধাপ ১ — Image build

```bash
docker build -t simple-docker-app:v1 .
```

### ধাপ ২ — Username দিয়ে tag করা

```bash
docker image tag simple-docker-app:v1 arifucoder/simple-docker-app:v1
docker images
```

✅ `docker tag` কোনো নতুন image তৈরি করে না — একই image-এর জন্য আরেকটি **নাম (alias)** বানায়। তাই `docker images` দিলে দুটি নাম দেখা গেলেও IMAGE ID একই থাকবে এবং extra disk খরচ হয় না।

### ধাপ ৩ — Docker Hub-এ login

```bash
docker login -u arifucoder
```

✅ Password-এর জায়গায় Docker Hub থেকে **Access Token (PAT)** generate করে সেটি paste করাই নিরাপদ ও recommended পদ্ধতি (Account Settings → Personal access tokens)।

আগে অন্য account login করা থাকলে:

```bash
docker logout
```

### ধাপ ৪ — Push

```bash
docker push arifucoder/simple-docker-app:v1
```

### ধাপ ৫ — আপলোড করা image দিয়ে container চালানো

```bash
docker run -d --name simple-nure -p 8080:80 arifucoder/simple-docker-app:v1
```

✅ Image local-এ না থাকলে Docker নিজে থেকেই Docker Hub থেকে pull করে নেয়। আলাদাভাবে আগে নামাতে চাইলে:

```bash
docker pull arifucoder/simple-docker-app:v1
```

> ⚠️ যে username দিয়ে `docker login` করা, tag-এও ঠিক সেই username থাকতে হবে। অন্য কারও username (যেমন `bongodev/...`) দিয়ে tag করা image আপনার account থেকে push করা যাবে না — `denied: requested access to the resource is denied` error আসবে।

### Push প্রক্রিয়া এক নজরে

| ধাপ | কমান্ড |
|---|---|
| ১ | `docker build -t simple-docker-app:v1 .` |
| ২ | `docker image tag simple-docker-app:v1 arifucoder/simple-docker-app:v1` |
| ৩ | `docker login -u arifucoder` |
| ৪ | `docker push arifucoder/simple-docker-app:v1` |
| ৫ | `docker run -d --name simple-nure -p 8080:80 arifucoder/simple-docker-app:v1` |

---

## ৬. DevSecOps ও Docker Scout

### 📘 DevSecOps কী?

**DevSecOps = Development + Security + Operations** ✅
অর্থাৎ security-কে শেষে আলাদা ধাপ হিসেবে না রেখে, development ও deployment-এর **প্রতিটি ধাপেই** যুক্ত করা।

**উদাহরণ ১:** Developer code push করার সাথে সাথেই CI pipeline-এ স্বয়ংক্রিয়ভাবে image scan হয়; vulnerability পাওয়া গেলে build fail করে দেয়।
**উদাহরণ ২:** Docker image build হওয়ার আগেই যাচাই করা হয় যে password/API key কোনো `.env` বা hardcoded string আকারে image-এ ঢুকে যাচ্ছে কি না।

---

### 📘 CVE কী?

**CVE = Common Vulnerabilities and Exposures** ✅
পাবলিকলি জানা security দুর্বলতাগুলোর একটি আন্তর্জাতিক তালিকা, যেখানে প্রতিটি দুর্বলতার আলাদা ID থাকে (যেমন `CVE-2023-44487`)। ফলে সারা পৃথিবীর সবাই একই ভাষায় একই সমস্যার কথা বলতে পারে।

**উদাহরণ ১:** পুরোনো version-এর `nginx` ব্যবহার করলে সেই version-এর পরিচিত CVE-গুলোর মাধ্যমে attacker সার্ভারে আক্রমণ করতে পারে।
**উদাহরণ ২:** পুরোনো `mysql` image-এ authentication bypass-সংক্রান্ত CVE থাকতে পারে, যেটি নতুন version-এ ঠিক করা হয়েছে।

---

### 📘 SBOM কী?

**SBOM = Software Bill of Materials** ✅
একটি image বা application-এর ভেতরে ঠিক কোন কোন package, library ও তাদের কোন version আছে — তার সম্পূর্ণ তালিকা। একে বলা যায় software-এর "উপকরণ তালিকা"।

**উদাহরণ ১:** SBOM দেখে বোঝা যায় আপনার image-এ `openssl 1.1.1` আছে, তাই নতুন কোনো OpenSSL vulnerability এলে সাথে সাথেই বোঝা যায় আপনি ঝুঁকিতে আছেন কি না।
**উদাহরণ ২:** Log4j-এর মতো বড় vulnerability এলে SBOM থাকলে কয়েক মিনিটেই খুঁজে বের করা যায় প্রতিষ্ঠানের কোন কোন image-এ সেই library আছে।

---

### 🔍 Docker Scout — Image security scanning

Docker Scout দিয়ে জানা যায় আমাদের application বা image-এ কোনো security vulnerability আছে কি না — যেমন পুরোনো version-এর `mysql`/`nginx` ব্যবহার করা, অথবা image-এর ভেতরে sensitive environment variable রেখে দেওয়া।

```bash
# Docker Scout ইনস্টল আছে কি না দেখা
docker scout version

# image-এর দ্রুত overview
docker scout quickview nginx:latest

# সম্পূর্ণ vulnerability তালিকা
docker scout cves nginx:latest

# সমস্যা সমাধানের পরামর্শ
docker scout recommendations nginx:latest

# image-এর SBOM দেখা
docker scout sbom nginx:latest
```

| কমান্ড | কাজ |
|---|---|
| `docker scout version` | Scout ইনস্টল ও version যাচাই |
| `docker scout quickview <image>` | কতগুলো Critical/High/Medium/Low সমস্যা আছে, সংক্ষেপে |
| `docker scout cves <image>` | প্রতিটি CVE-এর বিস্তারিত তালিকা |
| `docker scout recommendations <image>` | কোন base image-এ গেলে সমস্যা কমবে, তার পরামর্শ |
| `docker scout sbom <image>` | Image-এর ভেতরের সব package ও version-এর তালিকা |

✅ `docker scout` ব্যবহারের জন্য সাধারণত **Docker Hub-এ login থাকতে হয়** (`docker login`)।
✅ Vulnerability-র severity level: **Critical > High > Medium > Low**। প্রথমেই Critical ও High ঠিক করা উচিত।

---

### 🛡️ Vulnerability কমানোর বাস্তব উপায় (market practice)

| সমস্যা | সমাধান |
|---|---|
| `latest` tag ব্যবহার | নির্দিষ্ট version pin করা — `nginx:1.27-alpine` |
| ভারী base image | ছোট image ব্যবহার — `alpine` বা `slim` variant |
| root user দিয়ে app চালানো | Dockerfile-এ `USER appuser` দিয়ে non-root user সেট করা |
| Password/API key `-e` বা Dockerfile-এ রাখা | Docker secrets বা runtime env ব্যবহার করা, image-এ কখনো নয় |
| অপ্রয়োজনীয় file image-এ ঢুকে যাওয়া | `.dockerignore` ফাইল ব্যবহার করা |
| পুরোনো base image | নিয়মিত `docker pull` করে base image আপডেট রাখা ও rebuild করা |

> ⚠️ `MYSQL_ROOT_PASSWORD=nure123`-এর মতো password শেখার সময় ঠিক আছে, কিন্তু production-এ কখনোই command বা Dockerfile-এ plain text password রাখা যাবে না।

---

## ৭. দ্রুত রেফারেন্স — সব কমান্ড এক জায়গায়

| কাজ | কমান্ড |
|---|---|
| Network তৈরি | `docker network create my-network` |
| MySQL container চালানো | `docker run -d --name mysql-db --network=my-network -v mysql_data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=nure123 -e MYSQL_DATABASE=class-db -p 3306:3306 mysql:latest` |
| Log দেখা | `docker logs -f mysql-db` |
| Container-এ ঢোকা | `docker exec -it mysql-db bash` |
| Image build | `docker build -t simple-docker-app:v2 .` |
| Image tag | `docker image tag simple-docker-app:v1 arifucoder/simple-docker-app:v1` |
| Login / Logout | `docker login -u arifucoder` / `docker logout` |
| Image push | `docker push arifucoder/simple-docker-app:v1` |
| একাধিক image ডিলিট | `docker rmi <id1> <id2> -f` |
| Security overview | `docker scout quickview nginx:latest` |
| CVE তালিকা | `docker scout cves nginx:latest` |
| সমাধানের পরামর্শ | `docker scout recommendations nginx:latest` |

---

## ✅ এক নজরে মূল পয়েন্ট

1. ✅ **Named volume (`-v mysql_data:/var/lib/mysql`) ছাড়া MySQL container মুছলে সব data হারিয়ে যায়** — volume থাকলে data নিরাপদে থেকে যায়।
2. ✅ **`--network=my-network` ব্যবহারের আগে `docker network create my-network` দিয়ে network তৈরি করে নিতে হবে।**
3. ✅ **Code/HTML/JS আপডেট করলে image আবার build করতে হবে**, তারপর পুরনো container মুছে নতুন image দিয়ে container চালাতে হবে — শুধু restart-এ কাজ হবে না।
4. ✅ **`docker logs` = application-এর error/output; `docker inspect` = container-এর configuration** — দুটো ভিন্ন কাজে ব্যবহৃত হয়।
5. ✅ **একাধিক image একসাথে মুছতে:** `docker rmi <id1> <id2> -f`
6. ✅ **Docker Hub-এ push করতে image-এর নাম অবশ্যই `username/image-name:tag` format-এ হতে হবে**, এবং login করা username আর tag-এর username এক হতে হবে।
7. ✅ **`docker tag` নতুন image বানায় না** — একই image ID-র জন্য আরেকটি নাম তৈরি করে।
8. ✅ **Login-এ password-এর বদলে Access Token (PAT) ব্যবহার করাই নিরাপদ**; অন্য account login থাকলে আগে `docker logout`।
9. ✅ **DevSecOps = Development + Security + Operations** — security প্রতিটি ধাপে যুক্ত থাকে।
10. ✅ **CVE = Common Vulnerabilities and Exposures** (পরিচিত security দুর্বলতার আন্তর্জাতিক তালিকা)।
11. ✅ **SBOM = Software Bill of Materials** (image-এর ভেতরের সব package ও version-এর তালিকা)।
12. ✅ **Docker Scout-এর ৩টি প্রধান কমান্ড:** `quickview` (সারসংক্ষেপ) → `cves` (বিস্তারিত) → `recommendations` (সমাধান)।
13. ✅ **পুরোনো version-এর image (যেমন পুরোনো `mysql` বা `nginx`) নিজেই একটি security vulnerability** — নিয়মিত base image আপডেট রাখতে হবে।
14. ✅ **Production-এ `latest` tag নয়, নির্দিষ্ট version ব্যবহার করুন এবং password কখনো image-এর ভেতরে রাখবেন না।**