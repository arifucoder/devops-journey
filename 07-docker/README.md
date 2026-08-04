# 🐳 Docker — সম্পূর্ণ নোট (Class 1 → Class 4)

> চারটি ক্লাসের সব নোট এক জায়গায়, ধারাবাহিকভাবে সাজানো — একদম শূন্য থেকে শুরু করে Production-level debugging পর্যন্ত।

## 📚 সূচিপত্র

| পর্ব | বিষয় | সেকশন |
|---|---|---|
| **পর্ব ১** | Docker-এর ভিত্তি — VM, Container, Architecture, প্রথম হাতে-কলমে | ১–১১ |
| **পর্ব ২** | Image Build কৌশল — Single vs Multi-stage, `.dockerignore`, NGINX | ১২–১৮ |
| **পর্ব ৩** | Multi-container — Compose, Network, Volume, রিয়েল প্রজেক্ট | ১৯–২৩ |
| **পর্ব ৪** | Image lifecycle, Docker Hub ও Security (DevSecOps, Scout) | ২৪–২৯ |
| **পর্ব ৫** | Debugging — ৭ ধাপ, Exit code, ১৫টি বাস্তব সিনারিও | ৩০–৩৬ |
| **পরিশিষ্ট** | শব্দকোষ, মাস্টার চিটশিট, এক নজরে মূল পয়েন্ট | — |

---

# পর্ব ১ — Docker-এর ভিত্তি

**এই পর্বের প্রবাহ:** Evolution of VMs → What is VM → Containers → Docker | containerd → Docker Architecture → Docker Recipe → Run & Inspect Containers

---

## ১. Physical Server — VM-এর আগের যুগ

Container বা VM আসার আগে সব application একটাই physical machine-এর উপর একসাথে চলত।

![Physical Servers](/07-docker/class/images/class-1/01-physical-server.svg)

একটা **যৌথ পরিবারের বাড়ির** সাথে তুলনা করলে বোঝা সহজ — সবাই এক ছাদের নিচে, একই hardware, একই OS, একই resource শেয়ার করে।

| দিক | Physical Server-এ যা হতো |
|-----|--------------------------|
| Hardware | সবার জন্য একটাই (shared) |
| Operating System | একটাই OS সবাই শেয়ার করে |
| Isolation | খুবই কম — একজনের সমস্যা সবাইকে affect করে |
| Flexibility | কম (rigid), একটা machine-এর উপর পুরো নির্ভরশীল |

✅ **মূল সমস্যা:** একটা app crash করলে বা একটা bug থাকলে **পুরো machine সহ বাকি সব app** সমস্যায় পড়ত। এখান থেকেই **isolation**-এর দরকার তৈরি হয় — আর সেটাই VM ও Container-এর জন্ম দেয়।

---

## ২. Virtual Machine (VM) কী?

> **📖 সংজ্ঞা — Virtual Machine (VM):** একটা physical computer-এর ভেতরে software দিয়ে বানানো একটা "নকল/virtual কম্পিউটার", যার নিজের আলাদা full Operating System থাকে। একই hardware-কে অনেক ভাগে ভাগ করে প্রতিটা ভাগকে একটা স্বতন্ত্র কম্পিউটারের মতো ব্যবহার করা যায়।
>
> **উদাহরণ ১:** একটা শক্তিশালী server-এর উপর VMware/VirtualBox দিয়ে ৩টা VM বানানো — একটায় Windows, একটায় Linux, একটায় macOS।
> **উদাহরণ ২:** AWS-এর EC2 instance আসলে cloud-এ চলা একটা VM।

VM-কে বলা যায় **আলাদা আলাদা বড় বাড়ি (Family Estate)** — প্রতিটা ভাই-বোনের জন্য সম্পূর্ণ আলাদা বাড়ি, নিজের ছাদ, নিজের plumbing, নিজের সব।

### VM Architecture (স্তরে স্তরে সাজানো)

![VM Architecture](/07-docker/class/images/class-1/03-vm-architecture.svg)

নিচ থেকে উপরে স্তরগুলো:

| Layer | কাজ |
|-------|-----|
| **Physical Hardware** | CPU, RAM, Storage, Network — আসল যন্ত্রপাতি |
| **Host OS (Optional)** | Hypervisor-এর জন্য driver ও system service দেয় |
| **Hypervisor (VM Manager)** | VM তৈরি করে, resource বণ্টন করে, VM গুলোকে আলাদা রাখে |
| **Guest OS (প্রতি VM-এ আলাদা)** | প্রতিটা VM-এর নিজস্ব full OS (Windows / Linux / macOS) |
| **Applications** | প্রতিটা VM-এর ভেতরে চলা app ও service |

> **📖 সংজ্ঞা — Hypervisor:** যে software VM তৈরি ও পরিচালনা করে এবং hardware resource ভাগ করে দেয়।
> **উদাহরণ ১:** VMware ESXi (server-এর জন্য)। **উদাহরণ ২:** Oracle VirtualBox (পার্সোনাল ল্যাপটপে)।

✅ **মনে রাখুন:** প্রতিটা VM-এ **আলাদা full OS** থাকে বলেই VM ভারী (heavy), size বড়, এবং boot হতে সময় বেশি লাগে।

---

## ৩. Container কী?

> **📖 সংজ্ঞা — Container:** একটা lightweight, isolated প্যাকেজ যেখানে একটা application ও তার সব dependency (library, config) একসাথে বান্ডিল থাকে। VM-এর মতো আলাদা OS নেয় না — **host machine-এর OS kernel শেয়ার করে**, তাই অনেক হালকা ও দ্রুত।
>
> **উদাহরণ ১:** একটা Node.js API-কে তার সব library সহ একটা container-এ প্যাক করে যেকোনো machine-এ হুবহু একইভাবে চালানো।
> **উদাহরণ ২:** একটা database (MySQL) container হিসেবে সেকেন্ডে চালু করে ফেলা।

Container-কে বলা যায় **একটা apartment building** — এক building (এক OS kernel), ভেতরে অনেক আলাদা flat (container), প্রতিটা flat নিজের মতো isolated, কিন্তু ছাদ-ভিত্তি (kernel) শেয়ার করা।

### জনপ্রিয় Container Technology

- **Docker** — সবচেয়ে জনপ্রিয়, industry standard
- **Podman** — Docker-এর daemonless বিকল্প
- **LXC** (Linux Containers) — লো-লেভেল Linux container
- **Kubernetes (K8s)** — অনেক container একসাথে চালানো ও manage করার orchestration tool

✅ Container **application-level** virtualization, VM **OS-level** virtualization।

---

## ৪. VM vs Container — মূল পার্থক্য

![VM vs Containers](/07-docker/class/images/class-1/02-vm-vs-container.svg)

**VM = বড় আলাদা বাড়ি** (ভারী, দামি, redundant) ↔ **Container = ছোট apartment** (হালকা, দ্রুত, efficient)।

![Containers vs VM Comparison](/07-docker/class/images/class-1/04-comparison-table.svg)

| Feature | 🐳 Docker Container | 🖥️ Virtual Machine (VM) |
|---------|--------------------|--------------------------|
| **Isolation** | Process-level | Full OS-level |
| **Performance** | Lightweight, fast | Heavy, slower |
| **Resource Usage** | Efficient | Resource-intensive |
| **Startup Time** | Seconds ⚡ | Minutes ⏳ |
| **Portability** | Highly portable | Less portable |
| **আলাদা OS?** | না — host kernel শেয়ার করে | হ্যাঁ — প্রতি VM-এ full OS |

✅ **এক লাইনে:** Container-রা host OS-এর kernel শেয়ার করে বলে **দ্রুত ও বেশি portable**; VM **শক্তিশালী isolation** দেয় কিন্তু **বেশি resource** খায়।

---

## ৫. Docker vs containerd

দুটো একই জিনিস নয় — একটা আরেকটার উপর কাজ করে:

| জিনিস | কী |
|-------|-----|
| **Docker** | সম্পূর্ণ platform/tool (build, ship, run — সব)। ব্যবহারকারী এর সাথে কাজ করে (`docker` CLI দিয়ে)। |
| **containerd** | Docker-এর ভেতরে থাকা **core container runtime** — container তৈরি, চালু, বন্ধ, delete-এর আসল কাজ এটাই করে। |

✅ সহজ করে: **Docker হলো পুরো গাড়ি**, **containerd হলো তার ইঞ্জিন**।

---

## ৬. Docker Container Architecture

![Docker Container Architecture](/07-docker/class/images/class-1/05-docker-container-architecture.svg)

নিচ থেকে উপরে:

| Layer | কাজ |
|-------|-----|
| **Physical Hardware** | CPU, RAM, Storage, Network |
| **Host OS / Shared Kernel** | একটাই OS kernel, যা সব container শেয়ার করে |
| **Docker Engine** | container build, run ও manage করে |
| **Containers (1, 2, 3...)** | প্রতিটা container-এ আলাদা app + dependency (Web App, API, Database) |
| **Applications** | container-এর ভেতরে চলা service |

✅ **VM Architecture-এর সাথে পার্থক্য:** VM-এ প্রতিটার আলাদা **Guest OS** ছিল; এখানে সব container **একটাই host kernel** শেয়ার করে — এজন্যই container হালকা।

---

## ৭. Docker Client–Server Architecture

Docker একটা **client–server** model-এ চলে। CLI দিয়ে কমান্ড দিলে সেটা daemon-এর কাছে যায়, daemon আসল কাজ করে।

![Docker Client-Server Architecture](/07-docker/class/images/class-1/07-docker-client-server.svg)

| Component | কাজ |
|-----------|-----|
| **Docker Client (`docker` CLI)** | আমরা যে কমান্ড টাইপ করি (`docker run`, `docker build`...) |
| **Docker Daemon (`dockerd`)** | background service, যা image/container/network/volume সব manage করে |
| **containerd** | container-এর পুরো lifecycle (create/start/stop/delete) সামলায় |
| **REST API / Socket** | Client ও Daemon-এর যোগাযোগ হয় `/var/run/docker.sock`-এর মাধ্যমে |
| **Docker Registry** | image রাখার জায়গা (Docker Hub, Amazon ECR) — এখান থেকে `pull`/`push` হয় |

### containerd-এর ভেতরের গুরুত্বপূর্ণ টার্ম

- **Namespaces** → প্রতিটা container-কে আলাদা "নিজের জগৎ" দেয় (isolation)। এক container অন্যটার process দেখতে পায় না।
- **Cgroups** (control groups) → প্রতি container কত CPU/RAM ব্যবহার করবে তার লিমিট ঠিক করে।
- **Images / Volumes** → blueprint (image) ও persistent data storage (volume)।

> **📖 সংজ্ঞা — Docker Registry:** Docker image সংরক্ষণ ও শেয়ার করার অনলাইন repository।
> **উদাহরণ ১:** Docker Hub (public, সবচেয়ে বড়)। **উদাহরণ ২:** Amazon ECR (AWS-এর private registry)।

---

## ৮. Docker Image কোথায় থাকতে পারে?

| জায়গা | ব্যাখ্যা |
|--------|----------|
| **Local PC** | নিজের machine-এ Dockerfile দিয়ে build করা image |
| **Docker Hub** | public/cloud registry — pull করা যায়, নিজের image push-ও করা যায় |
| **Amazon ECR** | AWS-এর private container registry |

✅ নিজের বানানো image **Docker Hub-এ push** করা যায়, আবার অন্যের বানানো image **pull** করে নেওয়া যায়। (বিস্তারিত → সেকশন ২৭)

---

## ৯. Dockerfile → Image → Container (সবচেয়ে জরুরি ধারণা)

![Dockerfile to Container Analogy](/07-docker/class/images/class-1/06-dockerfile-image-container.svg)

| ধাপ | বাড়ির analogy | Docker |
|-----|---------------|--------|
| ১ | notebook-এ লেখা home requirement | **Dockerfile** (instruction-এর সেট) |
| ২ | Civil engineer-এর বানানো blueprint | **Docker Image** (blueprint) |
| ৩ | তৈরি হয়ে যাওয়া আসল বাড়ি | **Running Container** (image-এর চলন্ত instance) |

✅ **মূল কথা:**

- **Dockerfile** = কী কী করতে হবে তার নির্দেশনা (text file)
- **Image** = ওই নির্দেশনা থেকে build হওয়া read-only blueprint
- **Container** = image চালু করলে যে জীবন্ত/running instance তৈরি হয়

✅ **`docker run hello-world` দিলে কী হয়?** image থেকে সরাসরি একটা container বানিয়ে চালিয়ে দেয়। অর্থাৎ **image run করলেই container তৈরি হয়ে চালু হয়ে যায়।**

> **📖 সংজ্ঞা — Dockerfile:** ছোট ছোট instruction (FROM, COPY, RUN, CMD...)-এর একটা text file, যা থেকে Docker image তৈরি হয়।
> **উদাহরণ ১:** `FROM node:20` দিয়ে Node.js app image। **উদাহরণ ২:** `FROM nginx:alpine` দিয়ে web server image।

---

## ১০. Ubuntu-তে Docker Install ও Permission সমস্যা

```bash
# Docker install করা
sudo apt install docker.io -y

# Docker service চালু করা
sudo systemctl start docker

# machine reboot হলেও Docker যাতে auto-start হয়
sudo systemctl enable docker

# ঠিকমতো install হয়েছে কিনা check
docker --version
```

✅ **Production tip:** `docker.io` হলো Ubuntu-র নিজস্ব repo-র package — শেখার জন্য একদম ঠিক আছে। তবে production/latest version-এর জন্য **Docker-এর official repo** (`docker-ce`) recommended।

### প্রথম কমান্ড ও `permission denied`

```bash
docker ps
```

✅ **কেন এই error?** Docker daemon-এর সাথে কথা বলা হয় **Docker socket** (`/var/run/docker.sock`)-এর মাধ্যমে। এই socket-এর access শুধু `docker` group-এর user-রা পায়। Install-এর সময় একটা `docker` group তৈরি হয়, **কিন্তু ওই group-এ কোনো user শুরুতে যোগ করা থাকে না** — তাই সাধারণ user permission পায় না।

> **📖 সংজ্ঞা — Docker socket:** `dockerd` (daemon) যে Unix socket-এ শোনে, client তার মাধ্যমেই daemon-কে কমান্ড পাঠায়। ঠিকানা: `/var/run/docker.sock`।

```bash
# system এর group গুলো দেখা
cat /etc/group

# বর্তমান user কে docker group এ যোগ করা
sudo usermod -aG docker $USER

# Docker service ঠিকঠাক চলছে কিনা check
sudo systemctl status docker
```

এরপরও `docker ps` দিলে **এখনো permission denied দেখাতে পারে**, কারণ group পরিবর্তন current session-এ apply হয়নি।

```bash
# সবচেয়ে সহজ (কিন্তু ভারী) উপায়
sudo reboot

# reboot ছাড়াও করা যায় — নতুন group session নেওয়া
newgrp docker        # অথবা logout করে আবার login
```

### Test

```bash
docker run hello-world
docker ps       # শুধু বর্তমানে চালু (running) container
docker ps -a    # সব container — চালু + বন্ধ (exited)
```

✅ `hello-world` কাজ শেষ করে **exited** অবস্থায় চলে যায়, তাই সেটা দেখতে **`docker ps -a`** লাগে। এই পার্থক্যটা পুরো debugging-এর ভিত্তি (পর্ব ৫ দেখুন)।

---

## ১১. 🛠️ হাতে-কলমে: nginx দিয়ে custom App (Docker Recipe)

**উদ্দেশ্য:** nginx container চলবে, কিন্তু default page না দেখিয়ে **আমাদের নিজের `index.html`** দেখাবে।

### ধাপ ১ — folder ও ফাইল

```bash
mkdir nginx-demo
cd nginx-demo
vim index.html
```

### ধাপ ২ — Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

| লাইন | কাজ |
|------|-----|
| `FROM nginx:alpine` | base image হিসেবে হালকা `alpine`-ভিত্তিক nginx |
| `COPY index.html ...` | আমাদের HTML-কে nginx-এর default html ফোল্ডারে বসানো |
| `EXPOSE 80` | container ৮০ পোর্ট ব্যবহার করে — এটা মূলত **documentation** |
| `CMD ["nginx","-g","daemon off;"]` | container চালু হলে nginx-কে **foreground-এ** চালানো |

✅ **`daemon off;` কেন জরুরি?** nginx সাধারণত background-এ (daemon হিসেবে) চলে যায়, তখন container-এর প্রধান process শেষ ভেবে **container সাথে সাথে exit করে ফেলে**। মূল নিয়ম: **container ততক্ষণই বাঁচে যতক্ষণ তার main process (PID 1) চলে।**

✅ **`EXPOSE` বনাম `-p`:** `EXPOSE` শুধু documentation — এটা একা port খোলে না। বাইরে থেকে access করতে হলে run-এর সময় `-p host:container` দিয়ে **publish** করতে হয়।

### ধাপ ৩–৫ — build, দেখা ও run

```bash
docker build -t my-nginx-app:v1 .
docker images
docker run -d --name my-nginx-app-container -p 8080:80 my-nginx-app:v1
```

| অংশ | মানে |
|-----|------|
| `-t my-nginx-app:v1` | image-এর `name:tag` |
| `.` | build context — বর্তমান folder-এর Dockerfile ব্যবহার করো |
| `-d` | detached mode — background-এ চালাও |
| `--name ...` | container-এর নাম |
| `-p 8080:80` | **host port 8080** → **container port 80** |

✅ **Port mapping নিয়ম: `-p host:container`**

- **ডান পাশের port (80)** পরিবর্তন করা যাবে না — এটা container-এর ভেতরের nginx-এর port
- **বাম পাশের port (8080)** যেকোনো ফাঁকা (free) host port হতে পারে
- ✅ **Industry standard:** সম্ভব হলে host ও container port একই রাখা (যেমন `80:80`)

### 🔍 প্রথম Postmortem

```bash
docker ps -a
docker logs my-nginx-app-container

# ভুল থাকলে: container ও image মুছে আবার build
docker rm -f my-nginx-app-container
docker rmi my-nginx-app:v1
```

✅ **AWS-এ চালাতে চাইলে:** EC2-র **Security Group** থেকে ব্যবহৃত port (যেমন 8080) **inbound rule-এ enable** করতে হবে, নইলে browser থেকে access পাওয়া যাবে না।

---

# পর্ব ২ — Image Build কৌশল

---

## ১২. লোকাল মেশিন থেকে সার্ভারে ফাইল পাঠানো (SCP)

```bash
scp -i <pem-key> <file-name>.zip ubuntu@<server-ip>:/home/ubuntu/

# উদাহরণ
scp -i docker-playground-key.pem doc-single-multi-main.zip ubuntu@35.165.15.171:/home/ubuntu/
```

| অংশ | পূর্ণরূপ / মানে | ব্যাখ্যা |
|---|---|---|
| `scp` | **S**ecure **C**o**p**y | SSH প্রোটোকলের উপর দিয়ে ফাইল কপি করে, তাই ডেটা encrypted থাকে |
| `-i` | `--identity_file` | কোন private key দিয়ে লগইন হবে (`.pem` ফাইল) |
| `ubuntu@35.165.15.171` | `user@host` | সার্ভারের ইউজারনেম আর IP address |
| `:/home/ubuntu/` | destination path | সার্ভারের কোন ফোল্ডারে ফাইলটা রাখা হবে |

✅ `.pem` key-এর permission ঠিক না থাকলে `scp`/`ssh` কাজ করবে না ("UNPROTECTED PRIVATE KEY FILE" error):

```bash
chmod 400 docker-playground-key.pem
```

✅ পুরো ফোল্ডার পাঠাতে `-r` (recursive) লাগে:

```bash
scp -i key.pem -r ./my-project ubuntu@35.165.15.171:/home/ubuntu/
```

Ubuntu-তে `unzip` ডিফল্টভাবে থাকে না:

```bash
sudo apt update
sudo apt install -y unzip
unzip file.zip
```

---

## ১৩. `.dockerignore` ও Build Context

### 📖 সংজ্ঞা

**`.dockerignore`** ✅ (নামটা সম্পূর্ণ ছোট হাতের অক্ষরে, শুরুতে একটা ডট) হলো এমন একটা ফাইল যেখানে লিখে দিই — কোন ফাইল/ফোল্ডারগুলো **build context**-এ Docker daemon-এর কাছে পাঠানো হবে না। ঠিক `.gitignore`-এর মতো, শুধু Git-এর বদলে Docker-এর জন্য।

**উদাহরণ ১:** `node_modules` লিখে দিলে লোকাল মেশিনের ভারী `node_modules` ইমেজে কপি হবে না — কন্টেইনারের ভেতরে `npm install` নিজেই সেটা তৈরি করে নেবে।
**উদাহরণ ২:** `.env` লিখে দিলে সিক্রেট API key বা database password ভুলেও ইমেজের ভেতরে ঢুকবে না।

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

| কারণ | ব্যাখ্যা |
|---|---|
| ⚡ Build দ্রুত হয় | কম ফাইল Docker daemon-এ পাঠাতে হয় |
| 📦 Image ছোট হয় | অপ্রয়োজনীয় ফাইল ইমেজে ঢোকে না |
| 🔒 নিরাপদ | `.env`, `.git` ইত্যাদি leak হয় না |
| 🧊 Cache ভালো কাজ করে | অপ্রাসঙ্গিক ফাইল বদলালেও layer cache নষ্ট হয় না |

### 📖 Build Context কী?

`docker build` কমান্ডের শেষে যে `.` (ডট) থাকে — সেটাই build context। মানে: **এই ফোল্ডারের সব ফাইল Docker engine-এর কাছে পাঠানো হবে**, যাতে `COPY`/`ADD` সেখান থেকে ফাইল নিতে পারে।

**উদাহরণ ১:** `docker build -t myapp .` → বর্তমান ফোল্ডারটাই context।
**উদাহরণ ২:** `docker build -t myapp ./frontend` → শুধু `frontend` ফোল্ডার context, বাইরের ফাইল `COPY` করা যাবে না।

---

## ১৪. NGINX Config ও SPA Fallback

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

| লাইন | কাজ |
|---|---|
| `server { ... }` | একটা virtual server ব্লক — এক NGINX-এ একাধিক সাইট চালানো যায় |
| `listen 80;` | কন্টেইনারের ভেতরে ৮০ পোর্টে HTTP request শুনবে |
| `server_name localhost;` | কোন domain-এর request এই ব্লক handle করবে |
| `root /usr/share/nginx/html;` | ওয়েবসাইটের ফাইল কোথায় (NGINX ইমেজের ডিফল্ট ফোল্ডার) |
| `index index.html;` | ফোল্ডার চাইলে ডিফল্টভাবে কোন ফাইল দেখাবে |
| `location / { ... }` | `/` দিয়ে শুরু হওয়া সব URL-এর নিয়ম |
| `try_files $uri $uri/ /index.html;` | ⭐ SPA fallback |

### ✅ `try_files` লাইনটাই সবচেয়ে গুরুত্বপূর্ণ — কেন?

React/Vue/Angular হলো **SPA (Single Page Application)**। রাউটিং হয় ব্রাউজারের JavaScript দিয়ে, সার্ভারে `/about` বা `/dashboard` নামে কোনো ফাইল **থাকেই না** — শুধু `index.html` থাকে।

`try_files $uri $uri/ /index.html;` মানে NGINX ধাপে ধাপে চেষ্টা করবে:

1. `$uri` → ঠিক ওই নামের ফাইল আছে কি? (যেমন `/logo.png`)
2. `$uri/` → ওই নামের ফোল্ডার আছে কি?
3. দুটোই না পেলে → `index.html` ফেরত দাও ✅

এটা **না দিলে** ইউজার `/dashboard`-এ গিয়ে refresh দিলে **404 Not Found** দেখবে — SPA deploy-এর সবচেয়ে common বাগ।

---

## ১৫. Single-Stage Build

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

| Instruction | কাজ |
|---|---|
| `FROM node:20-alpine` | বেস ইমেজ। `alpine` = খুব হালকা Linux distro (~5MB) |
| `WORKDIR /app` | ইমেজের ভেতরে `/app` ফোল্ডার, এরপরের সব কমান্ড সেখান থেকে চলবে (`cd`-এর মতো) |
| `COPY package*.json ./` | `package.json` ও `package-lock.json` কপি |
| `RUN npm install` | সব dependency ইনস্টল |
| `COPY . .` | বাকি সোর্স কোড কপি (`.dockerignore`-এর ফাইল বাদ দিয়ে) |
| `ENV VITE_BUILD_TYPE="..."` | environment variable সেট |
| `RUN npm run build` | Production build → `dist/` ফোল্ডার |
| `EXPOSE 4173` | ডকুমেন্টেশন |
| `CMD [...]` | কন্টেইনার **রান** হওয়ার সময় যে কমান্ড চলবে |

### ✅ কেন `package*.json` আলাদা করে আগে কপি করি? (Layer Caching)

Dockerfile-এর প্রতিটা লাইন একটা করে **layer** তৈরি করে, আর Docker সেগুলো cache করে রাখে। যে লাইন থেকে কিছু বদলায়, সেই লাইন **এবং তার নিচের সব লাইন** আবার নতুন করে চলে।

- প্রথমেই `COPY . .` দিলে → কোডের একটা অক্ষর বদলালেও পুরো `npm install` আবার চলত (২–৩ মিনিট নষ্ট)
- আগে শুধু `package*.json` কপি করায় → dependency না বদলালে `npm install` layer cache থেকেই আসে ✅

### ✅ `--host 0.0.0.0` কেন দরকার?

ডিফল্টভাবে Vite preview server শুধু `localhost` (127.0.0.1)-এ শোনে — তখন সেটা **শুধু কন্টেইনারের ভেতর থেকেই** অ্যাক্সেসযোগ্য। `0.0.0.0` মানে "সব network interface-এ শোনো"। এটা না দিলে `-p` করা সত্ত্বেও ব্রাউজারে কিছু আসবে না।

`CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0"]` — মাঝের `--` npm-কে বলে: "এরপরের আর্গুমেন্টগুলো তোমার না, Vite-কে পাঠিয়ে দাও।"

```bash
docker build -f Dockerfile.single -t react-single:v1 .
docker run -d --name react-single -p 3001:4173 react-single:v1
```

| ফ্ল্যাগ | পূর্ণরূপ | মানে |
|---|---|---|
| `-f` | `--file` | কোন Dockerfile ব্যবহার হবে (নাম `Dockerfile` না হলে লাগে) |
| `-t` | `--tag` | ইমেজের `name:tag` |
| `-d` | `--detach` | ব্যাকগ্রাউন্ডে চলবে |
| `-p 3001:4173` | `--publish` | `host_port:container_port` |

> 🌐 `http://<server-ip>:3001` — ⚠️ AWS EC2 হলে Security Group-এ পোর্টটা Inbound rule-এ খুলতে হবে।

---

## ১৬. Multi-Stage Build

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

### ✅ "Stage" মানে কন্টেইনার নয়

`builder` আর `runner` — এগুলো **কন্টেইনার না**, এগুলো **build stage**, অর্থাৎ ইমেজ বানানোর সময়কার অস্থায়ী ধাপ। ✅

- প্রতিবার `FROM` লেখা মানেই একটা **নতুন stage** শুরু
- `AS builder` / `AS runner` হলো stage-এর **নাম বা alias**
- Build শেষে **শুধু সর্বশেষ stage-টাই** ফাইনাল ইমেজ হয়। আগের stage (এবং তাদের `node_modules`, সোর্স কোড, npm cache) **ফেলে দেওয়া হয়**
- ফাইনাল ইমেজ থেকে `docker run` করলে **একটাই কন্টেইনার** হয় — দুই stage থেকে দুটো কন্টেইনার নয়

### Alias না দিলে?

```dockerfile
COPY --from=0 /app/dist /usr/share/nginx/html
```

নাম্বার (০ থেকে শুরু) দিয়ে ডাকা যায়, কিন্তু পড়তে কষ্ট এবং stage-এর ক্রম বদলালে ভেঙে যায়। ✅ তাই সবসময় অর্থপূর্ণ নাম দেওয়াই best practice।

### `ENV VITE_BUILD_TYPE` — এটা আসলে কী?

1. **Vite-এর নিয়ম:** Vite শুধু `VITE_` দিয়ে শুরু হওয়া variable-গুলোকেই ফ্রন্টএন্ড কোডে ঢুকতে দেয় (`import.meta.env.VITE_BUILD_TYPE`)।
2. **কখন কাজ করছে:** `RUN npm run build`-এর **আগে** `ENV` দেওয়া হয়েছে, তাই build-এর সময় ভ্যালুটা `dist/`-এর JS ফাইলে **পাকাপাকিভাবে বসে যায়**।

এখানে এর একমাত্র কাজ — ব্রাউজারে খুলে বোঝা যে পেজটা Single-stage না Multi-stage build থেকে এসেছে। ✅ **ডেমোর জন্য, না দিলেও অ্যাপ চলবে।**

> ⚠️ ✅ **সিকিউরিটি নোট:** ফ্রন্টএন্ড build-এ দেওয়া `VITE_` variable ব্রাউজারে **সবাই দেখতে পায়**। এখানে কখনোই secret key, DB password, private token রাখা যাবে না।

### `COPY --from=builder` — মূল জাদু

দ্বিতীয় stage সম্পূর্ণ **নতুন ও খালি** — Node.js নেই, সোর্স কোড নেই, `node_modules` নেই, শুধু NGINX আছে।

```dockerfile
COPY --from=builder /app/dist /usr/share/nginx/html
```

মানে: *"builder stage-এর `/app/dist` নিয়ে এসে এই stage-এর `/usr/share/nginx/html`-এ রাখো।"* — শুধু দরকারি ফাইলটুকু তুলে আনা, বাকি ভারী জিনিস ফেলে দেওয়া। `--from=builder` না দিলে Docker `/app/dist` খুঁজে পাবে না → **build fail**।

| লাইন | ব্যাখ্যা |
|---|---|
| `COPY nginx.conf /etc/nginx/conf.d/default.conf` | নিজের config দিয়ে ডিফল্ট রিপ্লেস (SPA fallback পেতে) |
| `EXPOSE 80` | NGINX ডিফল্টভাবে ৮০ পোর্টে সার্ভ করে |
| `CMD ["nginx", "-g", "daemon off;"]` | foreground-এ চলবে (দেখুন সেকশন ১১) |

```bash
docker build -f Dockerfile.multi -t react-multi:v1 .
docker run -d --name react-multi -p 3002:80 react-multi:v1
```

---

## ১৭. Single vs Multi-Stage — কোনটা ভালো ও কেন?

| বিষয় | Single-Stage | Multi-Stage |
|---|---|---|
| **Image size** | বড় (~400–600 MB+) — Node.js, `node_modules`, সোর্স সবই থাকে | ছোট (~25–50 MB) — শুধু NGINX + `dist` ✅ |
| **সোর্স কোড ইমেজে থাকে?** | হ্যাঁ (leak-এর ঝুঁকি) | না ✅ |
| **Web server** | Vite preview server (dev-oriented) | NGINX (production-grade) ✅ |
| **Attack surface** | বড় — npm, Node runtime, বহু প্যাকেজ | ছোট ✅ |
| **Performance** | তুলনামূলক ধীর | দ্রুত (gzip, caching, static serving) ✅ |
| **Deploy/pull speed** | ধীর | দ্রুত ✅ |
| **Dockerfile জটিলতা** | সহজ | একটু বেশি |
| **কোথায় মানানসই** | দ্রুত টেস্ট, শেখা, ছোট experiment | **Production / real-world deployment** ✅ |

> 💡 **কারণটা এক লাইনে:** Build করার জন্য যা লাগে (Node, npm, compiler) আর অ্যাপ **চালানোর** জন্য যা লাগে (কিছু HTML/CSS/JS ফাইল) — এই দুটো এক জিনিস নয়। Multi-stage সেই দুটোকে আলাদা করে দেয়।

### ✅ আরেকটা ভালো অভ্যাস: `npm install`-এর বদলে `npm ci`

```dockerfile
RUN npm ci
```

| | `npm install` | `npm ci` |
|---|---|---|
| Lock file | দরকার হলে বদলে ফেলে | হুবহু `package-lock.json` মেনে চলে ✅ |
| গতি | ধীর | দ্রুত ✅ |
| ব্যবহার | লোকাল ডেভেলপমেন্ট | CI/CD ও Docker build ✅ |

এতে "আমার মেশিনে চলে, সার্ভারে চলে না" সমস্যা অনেকাংশে দূর হয়।

---

## ১৮. কন্টেইনারে ঢোকা ও Cleanup

```bash
docker exec -it react-single sh
exit
```

| ফ্ল্যাগ | পূর্ণরূপ | মানে |
|---|---|---|
| `exec` | execute | চলমান কন্টেইনারে নতুন কমান্ড চালায় |
| `-i` | `--interactive` | STDIN খোলা রাখে, টাইপ করা যায় |
| `-t` | `--tty` | terminal তৈরি করে (prompt দেখা যায়) |
| `sh` | shell | Alpine ইমেজে `bash` থাকে না, তাই `sh` ✅ |

```bash
# সব কন্টেইনার (চলমান + বন্ধ) জোর করে মুছে ফেলা
docker rm -f $(docker ps -a -q)

# সব ইমেজ জোর করে মুছে ফেলা
docker rmi -f $(docker images -a -q)
```

| অংশ | মানে |
|---|---|
| `-a` | `--all` → বন্ধ কন্টেইনার সহ সব |
| `-q` | `--quiet` → শুধু ID লিস্ট |
| `$( ... )` | ভেতরের কমান্ডের আউটপুটকে বাইরের কমান্ডের argument বানায় |
| `-f` | `--force` → চলমান অবস্থাতেও মুছে দেয় |
| `rmi` | **r**e**m**ove **i**mage |

> ⚠️ এই দুটো কমান্ড **সব কিছু** মুছে ফেলে — অন্য প্রজেক্টেরও। শেয়ার্ড বা প্রোডাকশন সার্ভারে চালানো যাবে না।

✅ **আরও সহজ বিকল্প:**

```bash
docker system prune -a            # অব্যবহৃত container, image, network
docker system prune -a --volumes  # সঙ্গে volume-ও (সাবধান!)
```

---

# পর্ব ৩ — Multi-container: Compose, Network, Volume

---

## ১৯. Docker Compose

### 📖 সংজ্ঞা

**Docker Compose** হলো একটা টুল যা একটা YAML ফাইল (`docker-compose.yaml`) ব্যবহার করে **একাধিক কন্টেইনার একসাথে define, run এবং maintain** করতে সাহায্য করে।

**উদাহরণ ১:** একটা expense tracker অ্যাপে frontend + backend + MongoDB — তিনটা সার্ভিস একটা কমান্ডে (`docker compose up -d`) চালু হয়ে যায়।
**উদাহরণ ২:** WordPress + MySQL — দুটো কন্টেইনার, একটা compose ফাইলে network আর volume সহ পুরোটা সেটআপ।

### কেন দরকার?

ধরা যাক প্রজেক্টে ৪–৫টা সার্ভিস — frontend, backend, database, cache — প্রত্যেকের নিজস্ব Dockerfile। Compose ছাড়া প্রতিবার:

1. প্রতিটা ফোল্ডারে আলাদা করে ঢুকতে হবে
2. আলাদা করে `docker build`
3. আলাদা করে `docker run`, সঙ্গে port, network, volume, env সব ম্যানুয়ালি
4. বন্ধ করতে আবার একটা একটা করে `docker stop`

Compose দিয়ে পুরোটা **একটা ফাইলে লিখে এক কমান্ডে চালানো যায়** ✅ — এবং ফাইলটা Git-এ থাকে, তাই টিমের সবাই হুবহু একই environment পায়।

```bash
sudo apt install -y docker.io docker-compose-v2
docker compose version
```

> ✅ পুরোনো `docker-compose` (হাইফেন সহ, Python-ভিত্তিক V1) এখন deprecated। এখন `docker compose` (স্পেস সহ, V2 plugin) ব্যবহার করা হয়।

### প্রধান কমান্ড

```bash
docker compose up -d --build    # build করে ব্যাকগ্রাউন্ডে সব সার্ভিস চালু
docker compose ps               # কোন সার্ভিস চলছে
docker compose logs -f          # সব সার্ভিসের live log
docker compose logs -f backend  # শুধু backend-এর log
docker compose down             # সব কন্টেইনার ও network বন্ধ + মুছে দেবে
docker compose down -v          # সঙ্গে volume-ও মুছবে (ডেটা চলে যাবে ⚠️)
docker compose restart backend  # নির্দিষ্ট সার্ভিস restart
docker compose config           # merge শেষে চূড়ান্ত কনফিগ দেখা ✅
```

| ফ্ল্যাগ | মানে |
|---|---|
| `up` | সার্ভিসগুলো তৈরি করে চালু করবে |
| `-d` | `--detach` → ব্যাকগ্রাউন্ডে |
| `--build` | ক্যাশ করা ইমেজ ব্যবহার না করে নতুন করে build ✅ কোড বদলালে দিতেই হবে |
| `-f` (logs-এ) | `--follow` → লগ লাইভ |
| `-v` (down-এ) | `--volumes` → named volume-ও মুছবে |

---

## ২০. 🧪 রিয়েল প্রজেক্ট: Expense Tracker (Compose + ৩ Service)

**MongoDB + Node.js backend + React frontend** — একটাই কমান্ডে চলে।

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

#### `mongo` সার্ভিস

| লাইন | ব্যাখ্যা |
|---|---|
| `image: mongo:7` | নিজের কোড নেই, তাই Docker Hub থেকে রেডিমেড ইমেজ (`build` লাগে না) |
| `container_name: expense-mongo` | নির্দিষ্ট নাম, নাহলে Compose `foldername-mongo-1` টাইপ নাম দেয় |
| `restart: unless-stopped` | crash বা reboot হলে নিজে থেকেই আবার চালু ✅ |
| `MONGO_INITDB_DATABASE` | কন্টেইনার **প্রথমবার** চালু হওয়ার সময় এই নামে database initialize |
| `volumes: - mongo-data:/data/db` | ⭐ MongoDB ডেটা `/data/db`-তে লেখে; সেটা volume-এ সেভ → **কন্টেইনার মুছলেও ডেটা বাঁচবে** ✅ |
| `networks: - expense-net` | এই কন্টেইনার `expense-net` নেটওয়ার্কে থাকবে |

> ✅ **mongo-তে কোনো `ports:` নেই — এটা ইচ্ছাকৃত ও সঠিক।** DB পোর্ট (27017) বাইরে খুলে দিলে ইন্টারনেট থেকে যে কেউ database-এ কানেক্ট করার চেষ্টা করতে পারে। Backend তো একই network-এর ভেতর থেকেই `mongo:27017`-এ পৌঁছাচ্ছে, তাই বাইরে খোলার দরকারই নেই।

#### `backend` সার্ভিস

| লাইন | ব্যাখ্যা |
|---|---|
| `build: context: ./backend` | নিজের কোড থেকে ইমেজ; `./backend`-ই build context |
| `NODE_ENV: production` | Node/Express-কে production মোডে চালায় ✅ |
| `PORT: 5000` | অ্যাপ কোন পোর্টে শুনবে |
| `MONGO_URI: mongodb://mongo:27017/...` | ⭐⭐ `localhost` নয়, **`mongo`** — compose সার্ভিসের নাম |
| `CORS_ORIGIN: http://localhost` | কোন origin-এর request backend গ্রহণ করবে |
| `depends_on: - mongo` | mongo আগে **শুরু** হবে, তারপর backend |

> ✅ **backend-এও `ports:` নেই কেন?** ব্রাউজার সরাসরি backend-এ কল করে না। ব্রাউজার শুধু frontend (port 80)-এ যায়, আর frontend-এর NGINX ভেতরে ভেতরে `/api/` request backend-এ পাঠায় (proxy)। এটাই **reverse proxy pattern** — production standard ✅
> ⚠️ ডিবাগের সময় সরাসরি API টেস্ট করতে সাময়িকভাবে `ports: - "5000:5000"` যোগ করা যায়।

#### `frontend` সার্ভিস

| লাইন | ব্যাখ্যা |
|---|---|
| `ports: - "80:80"` | হোস্টের ৮০ (ডিফল্ট HTTP) → কন্টেইনারের ৮০ (NGINX)। ব্রাউজারে শুধু `http://server-ip` লিখলেই হয় ✅ |
| `depends_on: - backend` | backend আগে শুরু হবে |

#### টপ-লেভেল ব্লক

```yaml
volumes:
  mongo-data:          # named volume ডিক্লেয়ার

networks:
  expense-net:
    driver: bridge     # user-defined bridge network
```

সার্ভিসের ভেতরে ব্যবহার করার আগে এখানে **ডিক্লেয়ার করা বাধ্যতামূলক**, নাহলে Compose error দেবে ✅

### 🔑 সবচেয়ে গুরুত্বপূর্ণ কনসেপ্ট: সার্ভিসের নামই hostname

```
mongodb://mongo:27017/expense_tracker
          ▲
          └── এটা কোনো domain বা IP নয় — compose সার্ভিসের নাম
```

Docker একই user-defined network-এ থাকা কন্টেইনারদের জন্য **built-in DNS** চালায়। backend যখন `mongo` নামে কানেক্ট করতে চায়, Docker নিজেই সেটাকে mongo কন্টেইনারের internal IP-তে অনুবাদ করে ✅

| ভুল ধারণা | সঠিক |
|---|---|
| `mongodb://localhost:27017` | `mongodb://mongo:27017` ✅ |
| `proxy_pass http://localhost:5000` | `proxy_pass http://backend:5000` ✅ |

কারণ প্রতিটা কন্টেইনারের নিজের আলাদা `localhost` আছে। backend কন্টেইনারের ভেতরে `localhost` মানে **সে নিজে** — mongo নয়।

> ⚠️ **`depends_on`-এর সীমাবদ্ধতা:** এটা শুধু নিশ্চিত করে mongo কন্টেইনার **start** হয়েছে, কিন্তু MongoDB connection নেওয়ার জন্য **ready** কিনা তা নয়। তাই বড় প্রজেক্টে `healthcheck` + `condition: service_healthy`, অথবা backend কোডে retry logic ✅ (বিস্তারিত → সেকশন ৩৬ সিনারিও ৭)

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
| `RUN npm install --omit=dev` | ⭐ শুধু **production dependency**; `devDependencies` (nodemon, eslint, jest) বাদ ✅ |
| `COPY src ./src` | পুরো ফোল্ডার নয়, শুধু `src` — ইমেজ ছোট ও পরিচ্ছন্ন ✅ |
| `CMD ["node", "src/server.js"]` | সরাসরি node দিয়ে সার্ভার চালু |

> ✅ **backend-এ multi-stage লাগে না কেন?** Node.js backend চালাতে Node runtime **রানটাইমেও দরকার** — বাদ দেওয়ার মতো ভারী build tool নেই। কিন্তু frontend-এ Node শুধু build-এর জন্য দরকার, চালানোর জন্য নয় — তাই সেখানে multi-stage কাজে লাগে ✅
>
> ✅ `--omit=dev` আগে `--production` নামে পরিচিত ছিল; npm 8+ থেকে `--omit=dev`। Production-এ `npm ci --omit=dev` আরও ভালো।
>
> ✅ `npm start`-এর বদলে সরাসরি `node` ভালো, কারণ npm একটা বাড়তি process তৈরি করে যা Docker-এর stop signal (SIGTERM) ঠিকমতো অ্যাপে পৌঁছাতে দেয় না।

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
| `COPY index.html ./index.html` | Vite-এ `index.html` হলো **entry point** (CRA-র মতো `public/`-এ নয়, root-এ থাকে) ✅ |
| `RUN npm run build` | `dist/`-এ static HTML/CSS/JS তৈরি |
| `COPY --from=builder /app/dist ...` | ⭐ builder stage থেকে শুধু `dist` তুলে এনে NGINX-এর web root-এ ✅ |

> ⚠️ এখানে `npm install` (dev সহ) দরকার, কারণ Vite নিজেই একটা `devDependency` — build করতে লাগবেই। তাই এখানে `--omit=dev` দেওয়া যাবে না ✅
> কিন্তু builder stage ফাইনাল ইমেজে থাকেই না, তাই ভারী `node_modules` ফাইনাল ইমেজে যাচ্ছে না। **এটাই multi-stage-এর সৌন্দর্য** ✅

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

**সংজ্ঞা:** এমন একটা সার্ভার যা ক্লায়েন্টের request নিজে গ্রহণ করে, তারপর পেছনে থাকা আসল সার্ভারে পাঠিয়ে দিয়ে উত্তরটা ক্লায়েন্টকে ফেরত দেয়। ক্লায়েন্ট জানেই না পেছনে কে আছে।

**উদাহরণ ১:** NGINX `/api/expenses` request পেয়ে সেটা `backend:5000`-এ পাঠায়, ব্রাউজার মনে করে সবকিছু একই সার্ভার থেকেই আসছে।
**উদাহরণ ২:** একটা সাইটে `/blog` গেলে WordPress সার্ভারে আর `/shop` গেলে অন্য সার্ভারে — ইউজার একটাই domain দেখে।

| Request | কোন ব্লকে | ফলাফল |
|---|---|---|
| `http://localhost/api/expenses` | `location /api/` | `http://backend:5000/api/expenses`-এ ফরওয়ার্ড ✅ |
| `http://localhost/dashboard` | `location /` | ফাইল না পেলে `index.html` → React router সামলাবে ✅ |
| `http://localhost/logo.png` | `location /` | আসল ফাইলটাই সার্ভ হবে |

> ✅ NGINX বেশি **নির্দিষ্ট (specific)** prefix-কে অগ্রাধিকার দেয়। তাই `/api/` দিয়ে শুরু হওয়া request কখনোই `location /`-এ যাবে না।

| লাইন | কাজ |
|---|---|
| `proxy_pass http://backend:5000/api/;` | ⭐ request কোথায় পাঠাবে। `backend` = compose সার্ভিসের নাম ✅ |
| `proxy_http_version 1.1;` | HTTP/1.1 — keep-alive ও WebSocket-এর জন্য দরকার |
| `proxy_set_header Host $host;` | ব্রাউজার যে domain চেয়েছিল সেটা backend-কে জানায় |
| `proxy_set_header X-Real-IP $remote_addr;` | ইউজারের আসল IP (নাহলে backend শুধু NGINX-এর IP দেখত) |
| `proxy_set_header X-Forwarded-For ...;` | কতগুলো proxy পেরিয়ে এসেছে তার চেইন |
| `proxy_set_header X-Forwarded-Proto $scheme;` | মূল request `http` না `https` ছিল |

✅ এই header না দিলে backend-এ logging, rate-limiting বা IP-ভিত্তিক নিরাপত্তা কাজ করবে না — সবার IP একই দেখাবে।

### ✅ এই সেটআপে CORS সমস্যা দূর হয়ে যায়

ব্রাউজারের চোখে **সব কিছু একই origin (`http://localhost`) থেকে আসছে** — frontend-ও, API-ও। ব্রাউজার আলাদা origin দেখে না, তাই CORS error-ই আসে না ✅

> ⚠️ **প্রোডাকশনে:** আসল domain-এ deploy করলে `CORS_ORIGIN` বদলে `https://yourdomain.com` করতে হবে, নাহলে API call block হবে ✅

### ▶️ পুরো অ্যাপ চালানো

```bash
cd expense-tracker
docker compose up -d --build
docker compose ps
docker compose logs -f backend
# ব্রাউজারে: http://<server-ip>
docker compose down       # বন্ধ করা (ডেটা থাকবে)
docker compose down -v    # বন্ধ + ডেটাও মুছে ফেলা ⚠️
```

### 🧭 Working Flow

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

1. ইউজার `http://server-ip` হিট করে → হোস্টের ৮০ পোর্ট → frontend কন্টেইনারের NGINX
2. NGINX `dist/`-এর React ফাইল সার্ভ করে (`location /`)
3. React থেকে `/api/expenses` কল → NGINX ধরে (`location /api/`) → `http://backend:5000/api/expenses`
4. Backend `mongodb://mongo:27017/expense_tracker` দিয়ে DB-তে কানেক্ট
5. Mongo ডেটা লেখে `/data/db`-তে → যা `mongo-data` **volume**-এ সেভ হয় ✅
6. শুধু frontend-এর পোর্ট বাইরে খোলা; backend ও mongo নেটওয়ার্কের ভেতরে সুরক্ষিত ✅

| নিরাপত্তা বিষয় | সুবিধা |
|---|---|
| শুধু একটা পোর্ট খোলা (80) | Attack surface সবচেয়ে ছোট ✅ |
| DB বাইরে exposed নয় | ইন্টারনেট থেকে database-এ কেউ পৌঁছাতে পারবে না ✅ |
| Backend proxy-র পেছনে | সরাসরি API hit করা যায় না |
| CORS সমস্যা নেই | সব same-origin |
| Volume-এ ডেটা | কন্টেইনার redeploy করলেও ডেটা অক্ষত ✅ |

> ✅ Compose V2-তে ফাইলের শুরুতে `version: "3.8"` লেখাটা আর দরকার নেই — এটা এখন obsolete এবং warning দেয়।

---

## ২১. Docker Network

### 📖 সংজ্ঞা

**Docker Network** হলো একটা virtual network যার মাধ্যমে কন্টেইনাররা একে অপরের ও বাইরের জগতের সাথে যোগাযোগ করে। এটা ঠিক করে দেয় — **কোন কন্টেইনার কার সাথে কথা বলতে পারবে**।

**উদাহরণ ১:** `expense-net`-এ frontend, backend, mongo রাখলাম — এরা নিজেদের মধ্যে নাম ধরে কথা বলতে পারবে।
**উদাহরণ ২:** আরেকটা প্রজেক্টের কন্টেইনার `blog-net`-এ রাখলাম — সে `expense-net`-এর mongo-তে কোনোভাবেই পৌঁছাতে পারবে না। এটাই **isolation**।

- 🔒 **নিরাপত্তা:** একটা অ্যাপ hack হলেও অন্য অ্যাপের DB নিরাপদ
- 🧭 **সহজ যোগাযোগ:** IP মনে রাখতে হয় না, **সার্ভিসের নামই hostname** ✅
- 🧹 **পরিচ্ছন্নতা:** প্রতিটা প্রজেক্ট নিজের গণ্ডিতে

> ✅ **গুরুত্বপূর্ণ পার্থক্য:** ডিফল্ট `bridge` network-এ থাকা কন্টেইনাররা **নাম দিয়ে** একে অপরকে খুঁজে পায় না (শুধু IP দিয়ে)। কিন্তু নিজে বানানো (user-defined) bridge network-এ automatic DNS resolution পাওয়া যায়। এজন্যই নিজের network বানানো best practice। Docker Compose নিজেই একটা user-defined network বানিয়ে দেয়।

| Driver | কাজ | কখন ব্যবহার |
|---|---|---|
| `bridge` | ডিফল্ট। একই হোস্টের কন্টেইনারদের প্রাইভেট নেটওয়ার্ক | সাধারণ সিঙ্গেল-সার্ভার অ্যাপ ✅ |
| `host` | কন্টেইনার সরাসরি হোস্টের নেটওয়ার্ক ব্যবহার করে, isolation থাকে না | সর্বোচ্চ নেটওয়ার্ক পারফরম্যান্স |
| `none` | কোনো নেটওয়ার্ক নেই, সম্পূর্ণ বিচ্ছিন্ন | নিরাপত্তা-সংবেদনশীল অফলাইন কাজ |
| `overlay` | একাধিক হোস্টের কন্টেইনার যুক্ত করে | Docker Swarm / multi-server cluster |

```bash
docker network create nure-net
docker network ls
docker network inspect nure-net

# run করার সময় network এ যুক্ত করা
docker run -d --name mysql-db --network=nure-net -e MYSQL_ROOT_PASSWORD=secret mysql:8

# চলমান কন্টেইনারকে যোগ / বিচ্ছিন্ন করা
docker network connect nure-net mysql-db
docker network disconnect nure-net mysql-db

docker network rm nure-net     # আগে সব কন্টেইনার disconnect থাকতে হবে
docker network prune           # অব্যবহৃত সব network মুছে ফেলা
```

> ✅ `docker run`-এ ইমেজের নাম শেষে দিতেই হয় (উপরে `mysql:8`), নাহলে Docker বুঝবে না কোন ইমেজ থেকে কন্টেইনার বানাতে হবে। আর `mysql` ইমেজ চালাতে `MYSQL_ROOT_PASSWORD` বাধ্যতামূলক, নাহলে কন্টেইনার চালু হয়েই বন্ধ হয়ে যাবে।

---

## ২২. Docker Volume

### 📖 সংজ্ঞা

**Docker Volume** হলো Docker-এর ম্যানেজ করা একটা স্টোরেজ, যা **হোস্ট মেশিনে** থাকে এবং কন্টেইনারের জীবনকালের সাথে যুক্ত নয়। কন্টেইনার মুছে গেলেও volume-এর ডেটা থেকে যায়।

**উদাহরণ ১:** `mongo-data` volume — MongoDB কন্টেইনার delete করে নতুন বানালেও পুরোনো সব ডেটা পাওয়া যায়।
**উদাহরণ ২:** ফাইল-আপলোড অ্যাপের `uploads` ফোল্ডার volume-এ রাখলে অ্যাপ redeploy করলেও ইউজারের ছবি হারায় না।

কন্টেইনারের নিজস্ব filesystem **ephemeral** ✅ (অস্থায়ী) — কন্টেইনার মুছে গেলে ভেতরের সব ডেটাও মুছে যায়। Database-এর ক্ষেত্রে এটা ভয়াবহ।

| # | কারণ | ব্যাখ্যা |
|---|---|---|
| ১ | **Data persistence** | কন্টেইনার stop/delete/update হলেও ডেটা টিকে থাকে ✅ |
| ২ | **Host isolation & performance** | Docker নিজে ম্যানেজ করে; container writable layer-এর চেয়ে I/O দ্রুত |
| ৩ | **Safe data sharing** | একই volume একাধিক কন্টেইনারে mount করে শেয়ার করা যায় |
| ৪ | **সহজ Backup ও Restore** ✅ | পুরো volume এক কমান্ডে tar করে ব্যাকআপ/রিস্টোর |
| ৫ | **Portability** ✅ | Linux, Windows, macOS — সব জায়গায় একইভাবে কাজ করে |

| | **Named Volume** | **Bind Mount** | **tmpfs** |
|---|---|---|---|
| কোথায় থাকে | `/var/lib/docker/volumes/` (Docker ম্যানেজ করে) | হোস্টের যেকোনো ফোল্ডার | শুধু RAM-এ |
| সিনট্যাক্স | `-v mydata:/data/db` | `-v /home/ubuntu/app:/app` | `--tmpfs /tmp` |
| ব্যবহার | **Production database** ✅ | Development-এ live code reload | সাময়িক/সংবেদনশীল ডেটা |
| সুবিধা | Portable, নিরাপদ, backup সহজ | কোড বদলালে সাথে সাথে দেখা যায় | খুব দ্রুত, ডিস্কে কিছু লেখে না |

```bash
docker volume create mydata
docker volume ls
docker volume inspect mydata        # কোথায় সেভ হচ্ছে সেটাসহ বিস্তারিত
docker volume rm mydata
docker volume prune                 # অব্যবহৃত সব volume

docker run -d --name mongo-db -v mydata:/data/db mongo:7
```

### Volume-এর ভেতরের ডেটা সরাসরি দেখা

```bash
sudo su
cd /var/lib/docker/volumes/<volume_name>/_data
ls -la
exit
```

> ⚠️ ✅ শেখার জন্য দেখা ঠিক আছে, কিন্তু এখানে **সরাসরি ফাইল এডিট/ডিলিট করা উচিত নয়** — database-এর ইন্টারনাল ফাইল নষ্ট হয়ে যেতে পারে। ডেটা দেখতে/বদলাতে কন্টেইনারের ভেতরে ঢুকে DB client দিয়ে করাই সঠিক উপায়।
> (Docker Desktop / macOS / Windows-এ এই path সরাসরি পাওয়া যাবে না, কারণ সেখানে Docker একটা VM-এর ভেতরে চলে।)

### ✅ Volume Backup ও Restore

```bash
# Backup: mydata volume কে backup.tar.gz বানানো
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar czf /backup/backup.tar.gz -C /data .

# Restore: backup থেকে ফেরত আনা
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar xzf /backup/backup.tar.gz -C /data
```

`--rm` মানে কাজ শেষে কন্টেইনারটা নিজে থেকেই মুছে যাবে।

---

## ২৩. কন্টেইনারের ভেতরে ঢুকে Database দেখা

```bash
docker exec -it <container-name> sh          # সাধারণ shell
docker exec -it <container-name> mongosh     # সরাসরি Mongo shell
```

```javascript
show dbs                     // সব database এর লিস্ট
use expense                  // expense database এ ঢোকা
show collections             // ওই DB-র সব collection
db.expenses.find()           // expenses collection এর সব ডেটা
db.expenses.find().pretty()  // সুন্দর ফরম্যাটে
db.expenses.countDocuments() // কতগুলো ডকুমেন্ট আছে
exit
```

> ✅ `db.<collection_name>.find()` — ডটের পরে **collection**-এর নাম বসবে, database-এর নাম নয়। Database সিলেক্ট করা হয়ে গেছে `use` দিয়ে, তাই `db` মানেই এখন সেই database।
> ✅ পুরোনো `mongo` shell এখন deprecated; MongoDB ৬+ ইমেজে `mongosh`। MySQL হলে `mysql -u root -p`, PostgreSQL হলে `psql -U postgres`।

---

# পর্ব ৪ — Image Lifecycle, Registry ও Security

---

## ২৪. MySQL Container চালানো

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

| অংশ | কাজ |
|---|---|
| `-d` | Detached mode — background-এ চলবে |
| `--name mysql-db` | পড়ার মতো নাম, পরে id-র বদলে ব্যবহার করা যাবে |
| `--network=my-network` | Custom bridge network-এ যুক্ত, যাতে অন্য container নাম দিয়ে connect করতে পারে |
| `-v mysql_data:/var/lib/mysql` | Named volume — database data container-এর বাইরে persist করে |
| `-e MYSQL_ROOT_PASSWORD=nure123` | root password (MySQL image-এর জন্য বাধ্যতামূলক) |
| `-e MYSQL_DATABASE=class-db` | প্রথমবার চালু হওয়ার সময় স্বয়ংক্রিয়ভাবে এই নামে database তৈরি |
| `-p 3306:3306` | Host-এর 3306 → container-এর 3306 |
| `mysql:latest` | কোন image থেকে container |

### ✅ আগে থেকে নিশ্চিত করার বিষয়

✅ **`--network=my-network` ব্যবহারের আগে network তৈরি থাকতে হবে**, না থাকলে Docker error দেবে:

```bash
docker network create my-network
docker network ls
```

✅ **Named volume আলাদা করে তৈরি করার দরকার নেই** — `-v mysql_data:/var/lib/mysql` লিখলে Docker নিজেই তৈরি করে নেয়:

```bash
docker volume ls
docker volume inspect mysql_data
```

✅ **Container ডিলিট করলেও volume-এর data থাকে।** Volume সহ মুছতে আলাদা করে `docker volume rm mysql_data` দিতে হয়।

✅ **Host-এ আগে থেকে MySQL চললে** 3306 port দখল হয়ে থাকবে এবং container start হবে না। তখন host port বদলাতে হবে: `-p 3307:3306`।

```bash
docker exec -it mysql-db mysql -u root -p
```

```sql
SHOW DATABASES;
USE class-db;
```

---

## ২৫. Code/Content আপডেট করলে কী করতে হবে

HTML, CSS, JS বা যেকোনো source code পরিবর্তন করলে **শুধু container restart করলে হবে না** — কারণ image তৈরির সময়ই code image-এর ভেতরে copy হয়ে যায়।

### ✅ সঠিক ধাপ (rebuild → remove old → run new)

```bash
# ১. নতুন করে image build (নতুন tag দিয়ে)
docker build -t simple-docker-app:v2 .

# ২. পুরনো container বন্ধ করে মুছে ফেলা
docker stop simple-nure
docker rm simple-nure

# ৩. নতুন image দিয়ে container চালানো
docker run -d --name simple-nure -p 8080:80 simple-docker-app:v2
```

✅ প্রতিবার **নতুন tag** (`v1`, `v2`, `v3`) দেওয়াই ভালো অভ্যাস — সমস্যা হলে আগের version-এ সহজে ফিরে যাওয়া যায় (rollback)।

> 💡 development-এ বারবার rebuild এড়াতে bind mount:
> `docker run -d -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx:latest`
> ফাইল পরিবর্তন করলেই সরাসরি container-এ দেখা যায়। তবে **production-এ সবসময় rebuild-ই করতে হবে।**

---

## ২৬. Docker Image ডিলিট করা

```bash
docker rmi <image-id-1> <image-id-2> -f     # একাধিক একসাথে
docker rmi simple-docker-app:v1 -f          # নাম:tag দিয়েও
```

| কমান্ড | কাজ |
|---|---|
| `docker images` | সব image-এর তালিকা |
| `docker rmi <id>` | Image ডিলিট |
| `-f` | Force — container ব্যবহার করছে এমন image-ও জোর করে মুছে দেয় |
| `docker image prune -a` | ব্যবহার না হওয়া সব image একসাথে |
| `docker system prune -a` | Unused image + container + network একসাথে (disk খালি করতে) |

✅ `-f` ছাড়া মুছতে গেলে image কোনো container ব্যবহার করলে Docker error দেবে। নিরাপদ উপায় — আগে container মুছে তারপর image।

---

## ২৭. Docker Hub-এ Image আপলোড (Push)

Image-এর নাম অবশ্যই এই format-এ হতে হবে:

```
<dockerhub-username>/<image-name>:<tag>
```

```bash
# ধাপ ১ — build
docker build -t simple-docker-app:v1 .

# ধাপ ২ — username দিয়ে tag
docker image tag simple-docker-app:v1 arifucoder/simple-docker-app:v1
docker images

# ধাপ ৩ — login
docker login -u arifucoder
docker logout                # আগে অন্য account login থাকলে

# ধাপ ৪ — push
docker push arifucoder/simple-docker-app:v1

# ধাপ ৫ — আপলোড করা image দিয়ে container চালানো
docker run -d --name simple-nure -p 8080:80 arifucoder/simple-docker-app:v1
```

✅ `docker tag` কোনো নতুন image তৈরি করে না — একই image-এর জন্য আরেকটি **নাম (alias)** বানায়। তাই `docker images`-এ দুটি নাম দেখা গেলেও IMAGE ID একই থাকবে, extra disk খরচ হয় না।

✅ Password-এর জায়গায় Docker Hub থেকে **Access Token (PAT)** generate করে paste করাই নিরাপদ ও recommended (Account Settings → Personal access tokens)।

✅ Image local-এ না থাকলে Docker নিজেই Docker Hub থেকে pull করে নেয়। আগে নামাতে চাইলে: `docker pull arifucoder/simple-docker-app:v1`

> ⚠️ যে username দিয়ে `docker login`, tag-এও ঠিক সেই username থাকতে হবে। অন্য কারও username দিয়ে tag করা image push করা যাবে না — `denied: requested access to the resource is denied` error আসবে।

| ধাপ | কমান্ড |
|---|---|
| ১ | `docker build -t simple-docker-app:v1 .` |
| ২ | `docker image tag simple-docker-app:v1 arifucoder/simple-docker-app:v1` |
| ৩ | `docker login -u arifucoder` |
| ৪ | `docker push arifucoder/simple-docker-app:v1` |
| ৫ | `docker run -d --name simple-nure -p 8080:80 arifucoder/simple-docker-app:v1` |

---

## ২৮. DevSecOps, CVE ও SBOM

### 📘 DevSecOps কী?

**DevSecOps = Development + Security + Operations** ✅
security-কে শেষে আলাদা ধাপ হিসেবে না রেখে, development ও deployment-এর **প্রতিটি ধাপেই** যুক্ত করা।

**উদাহরণ ১:** Developer code push করার সাথে সাথেই CI pipeline-এ স্বয়ংক্রিয়ভাবে image scan হয়; vulnerability পাওয়া গেলে build fail করে দেয়।
**উদাহরণ ২:** Image build হওয়ার আগেই যাচাই করা হয় যে password/API key কোনো `.env` বা hardcoded string আকারে image-এ ঢুকছে কি না।

### 📘 CVE কী?

**CVE = Common Vulnerabilities and Exposures** ✅
পাবলিকলি জানা security দুর্বলতাগুলোর আন্তর্জাতিক তালিকা, প্রতিটির আলাদা ID (যেমন `CVE-2023-44487`)। ফলে সারা পৃথিবীর সবাই একই ভাষায় একই সমস্যার কথা বলতে পারে।

**উদাহরণ ১:** পুরোনো version-এর `nginx` ব্যবহার করলে সেই version-এর পরিচিত CVE-র মাধ্যমে attacker সার্ভারে আক্রমণ করতে পারে।
**উদাহরণ ২:** পুরোনো `mysql` image-এ authentication bypass-সংক্রান্ত CVE থাকতে পারে, যেটি নতুন version-এ ঠিক করা হয়েছে।

### 📘 SBOM কী?

**SBOM = Software Bill of Materials** ✅
একটি image বা application-এর ভেতরে ঠিক কোন কোন package, library ও কোন version আছে — তার সম্পূর্ণ তালিকা। software-এর "উপকরণ তালিকা"।

**উদাহরণ ১:** SBOM দেখে বোঝা যায় image-এ `openssl 1.1.1` আছে, তাই নতুন OpenSSL vulnerability এলে সাথে সাথেই বোঝা যায় ঝুঁকিতে আছি কি না।
**উদাহরণ ২:** Log4j-এর মতো বড় vulnerability এলে SBOM থাকলে কয়েক মিনিটেই বের করা যায় কোন কোন image-এ সেই library আছে।

---

## ২৯. 🔍 Docker Scout — Image Security Scanning

Docker Scout দিয়ে জানা যায় আমাদের image-এ কোনো security vulnerability আছে কি না — যেমন পুরোনো version-এর `mysql`/`nginx`, অথবা image-এর ভেতরে sensitive environment variable রেখে দেওয়া।

```bash
docker scout version                      # ইনস্টল ও version যাচাই
docker scout quickview nginx:latest       # দ্রুত overview
docker scout cves nginx:latest            # সম্পূর্ণ vulnerability তালিকা
docker scout recommendations nginx:latest # সমাধানের পরামর্শ
docker scout sbom nginx:latest            # image এর SBOM
```

| কমান্ড | কাজ |
|---|---|
| `quickview` | কতগুলো Critical/High/Medium/Low সমস্যা আছে, সংক্ষেপে |
| `cves` | প্রতিটি CVE-র বিস্তারিত তালিকা |
| `recommendations` | কোন base image-এ গেলে সমস্যা কমবে |
| `sbom` | Image-এর ভেতরের সব package ও version |

✅ `docker scout` ব্যবহারের জন্য সাধারণত **Docker Hub-এ login থাকতে হয়**।
✅ Severity level: **Critical > High > Medium > Low**। প্রথমেই Critical ও High ঠিক করা উচিত।

### 🛡️ Vulnerability কমানোর বাস্তব উপায় (market practice)

| সমস্যা | সমাধান |
|---|---|
| `latest` tag ব্যবহার | নির্দিষ্ট version pin করা — `nginx:1.27-alpine` |
| ভারী base image | ছোট image — `alpine` বা `slim` variant |
| root user দিয়ে app চালানো | Dockerfile-এ `USER appuser` দিয়ে non-root user |
| Password/API key `-e` বা Dockerfile-এ রাখা | Docker secrets বা runtime env; image-এ কখনো নয় |
| অপ্রয়োজনীয় file image-এ ঢোকা | `.dockerignore` ব্যবহার |
| পুরোনো base image | নিয়মিত `docker pull` করে base image আপডেট ও rebuild |

> ⚠️ `MYSQL_ROOT_PASSWORD=nure123`-এর মতো password শেখার সময় ঠিক আছে, কিন্তু production-এ কখনোই command বা Dockerfile-এ plain text password রাখা যাবে না।

---

# পর্ব ৫ — Container Debugging

> মূল কথা: কোনো container-এ সমস্যা হলে **অনুমান না করে, ধাপে ধাপে প্রমাণ দেখে** সমাধান করা।

---

## ৩০. ডিবাগিং-এর ৭টি ধাপ

```
Observe → Inspect → Reproduce → Isolate → Fix → Verify → Prevent
```

| ধাপ | বাংলায় মানে | বাস্তবে কী করি |
|---|---|---|
| **Observe** | সমস্যাটা খেয়াল করা | mail/alert এসেছে, টিমের কেউ বলেছে, বা মনিটরিং-এ নিজে দেখেছি |
| **Inspect** | ভেতরে তাকানো | `docker ps -a`, `docker logs`, `docker inspect` দিয়ে আসল অবস্থা দেখা |
| **Reproduce** | সমস্যাটা আবার ঘটানো | নিজের মেশিনে/স্টেজিং-এ একই কনফিগে চালিয়ে সমস্যাটা আবার দেখানো |
| **Isolate** | আলাদা করে ফেলা | কোন একটি service/container/layer-এ সমস্যা, সেটিকে বাকিদের থেকে আলাদা করে টেস্ট করা |
| **Fix** | ঠিক করা | কারণ বুঝে শুধু আসল জায়গাটায় পরিবর্তন করা |
| **Verify** | যাচাই করা | fix-এর পর container আবার চালিয়ে প্রমাণ করা যে সমস্যা গেছে |
| **Prevent** | পুনরাবৃত্তি ঠেকানো | healthcheck, restart policy, log, documentation, CI check যোগ করা |

✅ **Reproduce ধাপটা বাদ দেওয়া যাবে না।** যে সমস্যা আবার ঘটাতে পারিনি, সেটার fix ঠিক হয়েছে কিনা কখনোই প্রমাণ করা যাবে না।

✅ **Isolate** মানে শুধু "আলাদা করে ফেলা" নয় — মানে সমস্যার পরিধি ছোট করে আনা। যেমন: পুরো stack বন্ধ করে শুধু database container চালিয়ে দেখা সে একা ঠিকমতো ওঠে কিনা।

---

## ৩১. সোনালি নিয়ম (Golden Rules) ✅

| নিয়ম | কারণ |
|---|---|
| ✅ container fail করলে **আগে rebuild করা যাবে না** | rebuild করলে আগের container ও তার log মুছে যায়, প্রমাণ হারিয়ে যায় |
| ✅ issue না দেখে **Dockerfile পরিবর্তন করা যাবে না** | কারণ না জেনে কোড বদলানো মানে অন্ধভাবে ঠিক করার চেষ্টা |
| ✅ প্রথম কাজ সবসময় **log পড়া** | ৮০% সমস্যার উত্তর log-এই লেখা থাকে |
| ✅ একবারে **একটি জিনিস** বদলাও | একসাথে ৩টা বদলালে কোনটা কাজ করেছে বোঝা যাবে না |
| ✅ `docker ps` নয়, `docker ps -a` | বন্ধ হয়ে যাওয়া (exited) container শুধু `-a` দিলেই দেখা যায় |

---

## ৩২. প্রাথমিক Inspection কমান্ড

```bash
# সব container দেখা (বন্ধ হয়ে যাওয়াগুলোসহ)
docker ps -a

# log দেখা
docker logs <container_id>

# শেষ ১০০ লাইন + সময়সহ log
docker logs --tail 100 --timestamps <container_id>

# live log follow করা
docker logs -f <container_id>

# চলমান container এর ভেতরে ঢোকা
docker exec -it <container_id> sh

# resource ব্যবহার (CPU/Memory)
docker stats

# container এর ভেতরের process
docker top <container_id>
```

✅ `docker logs` শুধু সেই লেখাগুলোই দেখায় যা application **stdout / stderr**-এ পাঠায়। যদি অ্যাপ log লেখে কোনো ফাইলে (যেমন `/var/log/app.log`), তাহলে `docker logs` খালি দেখাবে — তখন `docker exec` দিয়ে ভেতরে ঢুকে ফাইলটা পড়তে হবে।

✅ **দুটো আলাদা কাজ:** `docker logs` = application-এর error/output; `docker inspect` = container-এর configuration (IP, volume, network, env)।

---

## ৩৩. Exit Code — container কী কারণে বন্ধ হলো

| Exit Code | মানে | বাস্তব কারণ | করণীয় |
|---|---|---|---|
| `Exited (0)` | Success | কাজ শেষ করে স্বাভাবিকভাবে বন্ধ | ✅ কিন্তু server/API-এর ক্ষেত্রে `0`-ও সমস্যা — মানে foreground-এ কোনো process চলছিল না |
| `Exited (1)` | General error | অ্যাপ crash, exception, config ভুল | `docker logs` পড়তেই হবে |
| `Exited (126)` | Command found কিন্তু execute হয়নি | permission নেই, ফাইল executable নয়, script-এ ভুল | `chmod +x`, ENTRYPOINT/CMD চেক |
| `Exited (127)` | Command not found | binary নেই, PATH ভুল, নামের typo | image-এ command আছে কিনা দেখা |
| `Exited (137)` | **SIGKILL (128 + 9)** | ✅ বেশিরভাগ সময় **OOM (Out Of Memory)** — memory limit পার হলে kernel জোর করে kill করে; অথবা `docker kill` | memory limit বাড়ানো / leak খোঁজা |
| `Exited (139)` | SIGSEGV (128 + 11) | segmentation fault, native binary crash | architecture/library mismatch দেখা |
| `Exited (143)` | SIGTERM (128 + 15) | `docker stop` দিলে graceful shutdown | সাধারণত স্বাভাবিক |

✅ **সূত্র:** signal-জনিত exit code = `128 + signal number`. তাই `137 = 128 + 9 = SIGKILL`, `143 = 128 + 15 = SIGTERM`।

---

## ৩৪. `docker inspect` দিয়ে গভীরে দেখা

```bash
# পুরো State অংশ JSON আকারে
docker inspect --format '{{json .State}}' a42ba15afb3e

# শুধু status (running / exited / restarting)
docker inspect --format '{{.State.Status}}' a42ba15afb3e

# শুধু exit code
docker inspect --format '{{.State.ExitCode}}' b7c0ae1dacbf

# OOM এর কারণে kill হয়েছে কিনা — 137 এর আসল প্রমাণ
docker inspect --format '{{.State.OOMKilled}}' <container_id>

# healthcheck এর অবস্থা ও শেষ কয়েকটি ফলাফল
docker inspect --format '{{json .State.Health}}' <container_id>

# environment variables
docker inspect --format '{{json .Config.Env}}' <container_id>

# কোন network এ যুক্ত আছে
docker inspect --format '{{json .NetworkSettings.Networks}}' <container_id>
```

✅ `docker inspect` এবং `docker container inspect` — দুটোই একই কাজ করে। তবে `docker inspect` image/network/volume-এর ক্ষেত্রেও চলে, তাই container বোঝাতে স্পষ্টভাবে `docker container inspect` লেখা ভালো অভ্যাস।

---

## ৩৫. Docker Compose দিয়ে Debugging

```bash
# নির্দিষ্ট compose file দিয়ে container চালু (rebuild সহ)
docker compose -f compose.base.yml up -d --build

# কোন service কী অবস্থায় আছে
docker compose -f compose.base.yml ps

# সব বন্ধ করে দেওয়া
docker compose -f compose.base.yml down

# volume সহ পুরোপুরি মুছে ফেলা (ডেটা চলে যাবে, সাবধান)
docker compose -f compose.base.yml down -v
```

### একাধিক `-f` — Overlay / Override ফাইল ✅

```bash
docker compose -f compose.base.yml -f scenarios/03-wrong-db-host.yml logs --tail 100 api
docker compose -f compose.base.yml -f scenarios/03-wrong-db-host.yml exec api env
```

একাধিক `-f` দিলে Docker Compose ফাইলগুলোকে **উপরে-নিচে মিলিয়ে (merge)** নেয় — পরের ফাইলের মান আগের ফাইলের মানকে override করে। ডিবাগিং শেখার জন্য দারুণ কৌশল: base ঠিক রেখে শুধু একটা সমস্যা "চাপিয়ে" দেওয়া যায়।

✅ **উপরের কমান্ড দুটির গঠন ঠিকভাবে বোঝা জরুরি:**

| অংশ | আসলে কী |
|---|---|
| `api` | **service-এর নাম** (compose file-এ যা লেখা আছে) — এটা কোনো function নয় |
| `env` | container-এর **ভেতরে চালানো Linux command** — সব environment variable ছাপায় |
| `logs --tail 100` | ঐ service-এর শেষ ১০০ লাইন log |
| `exec` | চলমান container-এর ভেতরে কমান্ড চালানো |

অর্থাৎ `exec api env` মানে — "`api` নামের service-এর container-এর ভেতরে গিয়ে `env` কমান্ডটা চালাও"।

📎 প্র্যাকটিসের জন্য repo: `https://github.com/Nure/debugging`
🖼️ ক্লাসের ডায়াগ্রাম (steps to solve the problem): [ছবি লিংক](https://gist.github.com/user-attachments/assets/6405b58f-0ef2-47ec-99f0-9ec38d6ab194)

---

## ৩৬. ১৫টি বাস্তব সমস্যা (Real-World Scenarios)

### ১) Container exits during startup — শুরুতেই বন্ধ হয়ে যাওয়া

**লক্ষণ:** `docker ps -a`-তে `Exited (1)` বা `Exited (0)`, container কয়েক সেকেন্ডও টেকে না।
**কারণ:** config ভুল, দরকারি env var নেই, dependency অনুপস্থিত, অথবা foreground-এ কোনো process নেই (যেমন `CMD ["bash"]`)।

```bash
docker ps -a
docker logs <container_id>
docker inspect --format '{{.State.ExitCode}}' <container_id>
```

**Fix:** log-এর শেষ লাইনটাই সাধারণত আসল কারণ। ✅ container ততক্ষণই বাঁচে যতক্ষণ তার **main process (PID 1)** বেঁচে থাকে (দেখুন সেকশন ১১ — `daemon off;`)।

---

### ২) Restart loops — বারবার restart হতে থাকা

**লক্ষণ:** status বারবার `Restarting`, log-এ একই error বারবার।
**কারণ:** `restart: always` দেওয়া আছে, কিন্তু অ্যাপ প্রতিবার চালু হয়েই crash করছে।

```bash
docker ps -a
docker inspect --format '{{.RestartCount}}' <container_id>
docker logs --tail 50 <container_id>
```

**Fix:** ✅ ডিবাগের সময় **restart policy সাময়িকভাবে বন্ধ করতে হয়** (`restart: "no"`), নইলে container বারবার নতুন হয়ে যাওয়ায় log ধরা কঠিন। আসল crash-এর কারণ ঠিক করে তারপর policy ফিরিয়ে আনতে হবে।

---

### ৩) Wrong environment variables — ভুল env var

**লক্ষণ:** অ্যাপ চালু হয়, কিন্তু DB connect হয় না, বা "undefined"/"null" এরর।
**কারণ:** নামের typo (`DB_HOST` বনাম `DATABASE_HOST`), `.env` ফাইল লোড হয়নি, override ফাইল ভুল মান দিচ্ছে।

```bash
docker compose -f compose.base.yml exec api env
docker inspect --format '{{json .Config.Env}}' <container_id>
docker compose -f compose.base.yml config
```

**Fix:** ✅ `docker compose config` সব merge শেষে **চূড়ান্ত কনফিগ** দেখায় — কোন মান আসলে কার্যকর হচ্ছে তা এখানে ধরা পড়ে।

---

### ৪) `localhost` used between containers

**লক্ষণ:** `ECONNREFUSED 127.0.0.1:5432` জাতীয় এরর।
**কারণ:** ✅ প্রতিটি container-এর **নিজস্ব network namespace** থাকে। container-এর ভেতরে `localhost` মানে **ঐ container নিজেই**, host মেশিন বা অন্য container নয়।

```bash
# ভুল
DB_HOST=localhost
# ঠিক (service এর নাম db হলে)
DB_HOST=db

# যাচাই
docker compose -f compose.base.yml exec api ping -c 2 db
```

**Fix:** compose-এ service-এর নাম দিয়ে ডাকতে হবে, কারণ Docker-এর internal DNS ঐ নামটাকেই resolve করে (দেখুন সেকশন ২০ ও ২১)।

---

### ৫) Wrong port mapping — ভুল পোর্ট ম্যাপিং

**লক্ষণ:** browser-এ কিছুই আসে না, "connection refused"।
**কারণ:** ✅ `-p HOST:CONTAINER` — বাঁ পাশে **host**, ডান পাশে **container**। উল্টো লিখলে ভুল পোর্টে সংযোগ যায়।

```bash
docker ps                       # PORTS কলাম দেখুন
docker port <container_id>

# app container এর ভেতরে 3000 এ চললে:
docker run -p 8080:3000 myapp   # host:8080 → container:3000
```

**মনে রাখতে হবে:** container-থেকে-container যোগাযোগে `ports` লাগেই না — internal network-এ এমনিতেই কাজ করে। `ports` শুধু **বাইরে থেকে (host)** ঢোকার জন্য।

---

### ৬) Application bound to `127.0.0.1` — ভুল ঠিকানায় bind

**লক্ষণ:** port mapping ঠিক আছে, container চলছে, তবু host থেকে ঢোকা যাচ্ছে না।
**কারণ:** অ্যাপ container-এর ভেতরে শুধু `127.0.0.1`-এ শুনছে, তাই বাইরের request গ্রহণ করছে না।

```bash
# Node.js
app.listen(3000, "0.0.0.0")

# Python / Flask
app.run(host="0.0.0.0", port=5000)

# ভেতরে গিয়ে যাচাই
docker exec -it <container_id> sh -c "netstat -tulpn || ss -tulpn"
```

**Fix:** ✅ container-এ চলা অ্যাপ সবসময় **`0.0.0.0`**-তে bind করতে হবে (Vite-এ `--host 0.0.0.0`, দেখুন সেকশন ১৫)।

---

### ৭) Database not ready — DB প্রস্তুত হওয়ার আগেই অ্যাপ চালু

**লক্ষণ:** অ্যাপ প্রথমবার crash করে, কিন্তু হাতে restart দিলে ঠিক চলে।
**কারণ:** ✅ `depends_on` শুধু container **চালু হওয়া** পর্যন্ত অপেক্ষা করে, ডেটাবেস **connection নেওয়ার জন্য প্রস্তুত** কিনা তা দেখে না।

```yaml
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5
  api:
    depends_on:
      db:
        condition: service_healthy
```

**Fix:** healthcheck + `condition: service_healthy`, এবং অ্যাপের কোডেও **retry logic** (প্রোডাকশনে DB যেকোনো সময় সাময়িক বন্ধ হতে পারে)।

---

### ৮) Volume hiding files — volume ফাইল ঢেকে দেওয়া

**লক্ষণ:** image-এ ফাইল ছিল, কিন্তু container-এ নেই; "module not found" জাতীয় এরর।
**কারণ:** ✅ কোনো ডিরেক্টরিতে volume/bind mount করলে সেটি image-এর ঐ ডিরেক্টরিকে **ঢেকে (mask) দেয়**।

```yaml
volumes:
  - ./src:/app          # host এর src পুরো /app কে ঢেকে দিল
  - /app/node_modules   # anonymous volume দিয়ে node_modules কে রক্ষা করা হলো
```

```bash
docker exec -it <container_id> ls -la /app
docker inspect --format '{{json .Mounts}}' <container_id>
```

---

### ৯) Permission errors — অনুমতিজনিত সমস্যা

**লক্ষণ:** `EACCES: permission denied`, log/upload ফোল্ডারে লিখতে না পারা।
**কারণ:** container-এর user (যেমন UID 1000) আর host ফাইলের owner আলাদা; অথবা non-root user দিয়ে root-মালিকানাধীন ফোল্ডারে লেখার চেষ্টা।

```bash
docker exec -it <container_id> id
docker exec -it <container_id> ls -la /app/uploads
```

```dockerfile
RUN chown -R node:node /app
USER node
```

**Fix:** ✅ শুধু `USER root` দিয়ে সমস্যা "চাপা" দেওয়া খারাপ অভ্যাস — নিরাপত্তার ঝুঁকি তৈরি হয়। সঠিক owner ও permission ঠিক করাই আসল সমাধান।

---

### ১০) Health check failing

**লক্ষণ:** status `unhealthy`, অথবা `service_healthy` শর্তে আটকে থাকা।
**কারণ:** healthcheck-এ ব্যবহৃত tool (যেমন `curl`) image-এ নেই, ভুল endpoint/port, বা `start_period` খুব কম।

```bash
docker inspect --format '{{json .State.Health}}' <container_id>
docker exec -it <container_id> curl -f http://localhost:3000/health
```

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 10s
  timeout: 3s
  retries: 3
  start_period: 30s     # অ্যাপ চালু হতে যতটুকু সময় লাগে
```

---

### ১১) Memory limit ও Exit code 137

**লক্ষণ:** হঠাৎ container বন্ধ, log-এ কোনো error নেই, `Exited (137)`।
**কারণ:** ✅ memory limit ছাড়িয়ে গেলে kernel-এর **OOM killer** process-টিকে SIGKILL পাঠায় — অ্যাপ কিছু লেখার সুযোগই পায় না, তাই log ফাঁকা থাকে।

```bash
docker inspect --format '{{.State.OOMKilled}}' <container_id>   # true হলে নিশ্চিত
docker stats
```

```yaml
deploy:
  resources:
    limits:
      memory: 512M
```

**Fix:** limit বাড়ানো, অথবা আসল memory leak খোঁজা। Java/Node-এর ক্ষেত্রে heap size container limit-এর চেয়ে ছোট রাখা।

---

### ১২) Stale image or build cache — পুরোনো image / cache

**লক্ষণ:** কোড বদলেছি, কিন্তু container-এ পুরোনো আচরণই দেখাচ্ছে।
**কারণ:** নতুন করে build হয়নি, Docker পুরোনো layer cache থেকে নিয়েছে, অথবা `latest` tag-এর পুরোনো কপি local-এ রয়ে গেছে।

```bash
docker compose -f compose.base.yml up -d --build
docker compose -f compose.base.yml build --no-cache
docker compose -f compose.base.yml pull
docker image ls
docker inspect --format '{{.Image}}' <container_id>
```

**Fix:** ✅ প্রোডাকশনে `latest` tag নয়, **নির্দিষ্ট version tag** (যেমন `myapp:1.4.2`) — তাহলে কোন কোড চলছে তা নিয়ে সন্দেহ থাকে না (দেখুন সেকশন ২৫ ও ২৯)।

---

### ১৩) Missing startup executable

**লক্ষণ:** `Exited (127)` — command not found, বা `exec format error`, বা `Exited (126)`।

| কারণ | ব্যাখ্যা |
|---|---|
| binary image-এ নেই | slim/alpine image-এ অনেক tool থাকে না |
| ফাইল executable নয় | `chmod +x entrypoint.sh` দেওয়া হয়নি → 126 |
| ✅ Windows-এর CRLF line ending | `entrypoint.sh`-এ `\r` থাকলে Linux command খুঁজে পায় না |
| ভুল WORKDIR / path | relative path ভুল জায়গা থেকে খুঁজছে |
| architecture mismatch | ARM মেশিনে বানানো image AMD64-এ চালানো → `exec format error` |

```bash
docker run --rm -it --entrypoint sh myapp
ls -la /app
which node
file /app/entrypoint.sh
```

---

### ১৪) Different container networks — আলাদা network

**লক্ষণ:** service নাম দিয়েও connect হচ্ছে না — "host not found"।
**কারণ:** container দুটি ভিন্ন Docker network-এ আছে (যেমন দুটি আলাদা compose project থেকে চালু)। ✅ Docker-এর DNS দিয়ে নাম resolve **শুধু একই network-এর ভেতরেই** কাজ করে।

```bash
docker network ls
docker inspect --format '{{json .NetworkSettings.Networks}}' <container_id>
docker network inspect <network_name>
```

```yaml
networks:
  shared-net:
    external: true     # দুই প্রজেক্টের container একই network এ আনার উপায়
```

---

### ১৫) Reverse-proxy errors — 502 Bad Gateway

**লক্ষণ:** nginx/traefik থেকে `502 Bad Gateway` বা `504 Gateway Timeout`।
**কারণ:** ✅ 502 মানে proxy চলছে, কিন্তু **পেছনের অ্যাপ (upstream) সাড়া দিচ্ছে না**।

| কারণ | সমাধান |
|---|---|
| `proxy_pass http://localhost:3000` লেখা | service-এর নাম দিতে হবে: `http://api:3000` |
| ভুল upstream port | container-এর internal port দিতে হবে, host port নয় |
| অ্যাপ container crash করেছে | `docker compose ps` + অ্যাপের log দেখা |
| proxy অ্যাপের আগে চালু হয়েছে | healthcheck / `depends_on` দিয়ে ক্রম ঠিক করা |
| অ্যাপ 127.0.0.1-এ bind করেছে | `0.0.0.0`-তে bind করা (সিনারিও ৬) |

```bash
docker compose -f compose.base.yml logs --tail 100 nginx
docker compose -f compose.base.yml exec nginx curl -I http://api:3000
```

---

## 🔎 দ্রুত রেফারেন্স — লক্ষণ থেকে কমান্ড

| যা দেখছি | প্রথমেই যে কমান্ড |
|---|---|
| container নেই / বন্ধ | `docker ps -a` |
| কেন বন্ধ হলো | `docker logs --tail 100 <id>` |
| exit code কত | `docker inspect --format '{{.State.ExitCode}}' <id>` |
| memory-তে মারা গেছে কিনা | `docker inspect --format '{{.State.OOMKilled}}' <id>` |
| env ঠিক আছে কিনা | `docker compose exec <service> env` |
| network/DNS সমস্যা | `docker compose exec <service> ping <other_service>` |
| port ঠিক আছে কিনা | `docker ps` → PORTS কলাম / `docker port <id>` |
| healthcheck অবস্থা | `docker inspect --format '{{json .State.Health}}' <id>` |
| ফাইল আছে কিনা | `docker exec -it <id> ls -la /app` |
| চূড়ান্ত compose কনফিগ | `docker compose config` |

---

# 📖 পরিশিষ্ট ক — শব্দকোষ (সংজ্ঞা + ২টি উদাহরণ)

> পর্বের ভেতরে যেসব সংজ্ঞা এসেছে (VM, Hypervisor, Container, Dockerfile, Registry, Docker socket, `.dockerignore`, Build Context, Compose, Network, Volume, Reverse Proxy, DevSecOps, CVE, SBOM) সেগুলোর বাইরে ডিবাগিং-এ যেসব টার্ম দরকার —

### Exit Code
container-এর main process বন্ধ হওয়ার সময় যে সংখ্যা রেখে যায়, যা বলে দেয় কীভাবে বন্ধ হয়েছে।
- **উদাহরণ ১:** `Exited (0)` — backup script কাজ শেষ করে সফলভাবে বন্ধ হয়েছে।
- **উদাহরণ ২:** `Exited (137)` — 512MB limit-এর container 600MB memory নেওয়ায় OOM killer তাকে kill করেছে।

### Bind Address (`0.0.0.0` বনাম `127.0.0.1`)
অ্যাপ কোন network interface-এ request শুনবে তার ঠিকানা।
- **উদাহরণ ১:** `127.0.0.1:3000` — শুধু ঐ container-এর ভেতর থেকেই ঢোকা যাবে, বাইরে থেকে নয়।
- **উদাহরণ ২:** `0.0.0.0:3000` — সব interface-এ শুনবে, তাই `-p 8080:3000` দিয়ে host থেকে ঢোকা যাবে।

### Layer Caching
Dockerfile-এর প্রতিটা instruction একটা layer বানায়, যা Docker পরের build-এ পুনরায় ব্যবহার করে।
- **উদাহরণ ১:** `package.json` না বদলালে `npm install` layer cache থেকে আসে, build কয়েক সেকেন্ডে শেষ।
- **উদাহরণ ২:** cache-এর কারণে পুরোনো কোড থেকে যাওয়ায় `--no-cache` দিয়ে নতুন করে build করা।

### Volume Masking
কোনো ডিরেক্টরিতে volume mount করলে image-এর ভেতরের ঐ ডিরেক্টরি ঢাকা পড়ে যাওয়া।
- **উদাহরণ ১:** `./src:/app` mount করায় image-এ থাকা `/app/node_modules` আর দেখা যাচ্ছে না।
- **উদাহরণ ২:** খালি named volume `/var/lib/mysql`-এ mount করায় image-এর প্রাথমিক ডেটা ঢাকা পড়েছে।

### Healthcheck
Docker নিয়মিত একটি কমান্ড চালিয়ে দেখে container সত্যিই কাজ করার মতো অবস্থায় আছে কিনা।
- **উদাহরণ ১:** `curl -f http://localhost:3000/health` প্রতি ১০ সেকেন্ডে চালানো।
- **উদাহরণ ২:** PostgreSQL-এ `pg_isready -U postgres` দিয়ে DB connection নিতে প্রস্তুত কিনা দেখা।

### Restart Policy
container বন্ধ হলে Docker স্বয়ংক্রিয়ভাবে আবার চালু করবে কিনা তার নিয়ম।
- **উদাহরণ ১:** `restart: unless-stopped` — সার্ভার reboot হলেও container আবার উঠবে।
- **উদাহরণ ২:** `restart: on-failure:3` — ব্যর্থ হলে সর্বোচ্চ ৩ বার চেষ্টা করবে।

### OOM Kill (Out Of Memory)
memory limit পার হলে Linux kernel জোর করে process বন্ধ করে দেওয়া।
- **উদাহরণ ১:** বড় CSV ফাইল একবারে memory-তে লোড করায় 512MB limit-এর container kill হয়ে গেল।
- **উদাহরণ ২:** Node.js অ্যাপে memory leak-এর কারণে ধীরে ধীরে RAM বেড়ে ২ দিন পর 137 exit code।

### SPA (Single Page Application)
এমন frontend অ্যাপ যেখানে পুরো সাইট একটাই HTML ফাইলে চলে, রাউটিং হয় ব্রাউজারের JavaScript দিয়ে।
- **উদাহরণ ১:** React অ্যাপে `/dashboard`-এ গেলে সার্ভারে নতুন page request যায় না।
- **উদাহরণ ২:** Vue/Angular অ্যাপে refresh দিলে 404 এড়াতে NGINX-এ `try_files ... /index.html` লাগে।

---

# 📋 পরিশিষ্ট খ — মাস্টার কমান্ড চিটশিট

```bash
# ---------- Setup ----------
sudo apt install docker.io docker-compose-v2 -y
sudo systemctl start docker && sudo systemctl enable docker
sudo usermod -aG docker $USER && newgrp docker
docker --version && docker compose version

# ---------- Image ----------
docker build -t myapp:v1 .
docker build -f Dockerfile.multi -t react-multi:v1 .
docker build --no-cache -t myapp:v1 .
docker images
docker image tag myapp:v1 username/myapp:v1
docker rmi <id1> <id2> -f
docker image prune -a

# ---------- Registry ----------
docker login -u username
docker push username/myapp:v1
docker pull username/myapp:v1
docker logout

# ---------- Container ----------
docker run -d --name web -p 8080:80 myapp:v1
docker ps
docker ps -a
docker logs -f --tail 100 --timestamps web
docker exec -it web sh
docker stop web && docker rm web
docker rm -f web
docker stats
docker top web
docker port web

# ---------- Inspect / Debug ----------
docker inspect web
docker inspect --format '{{.State.Status}}' web
docker inspect --format '{{.State.ExitCode}}' web
docker inspect --format '{{.State.OOMKilled}}' web
docker inspect --format '{{json .State.Health}}' web
docker inspect --format '{{json .Config.Env}}' web
docker inspect --format '{{json .Mounts}}' web
docker inspect --format '{{json .NetworkSettings.Networks}}' web

# ---------- Compose ----------
docker compose up -d --build
docker compose ps
docker compose logs -f backend
docker compose exec api env
docker compose config
docker compose restart backend
docker compose down
docker compose down -v
docker compose -f compose.base.yml -f scenarios/03-wrong-db-host.yml up -d

# ---------- Network ----------
docker network create my-network
docker network ls
docker network inspect my-network
docker network connect my-network web
docker network prune

# ---------- Volume ----------
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker volume rm mydata
docker volume prune

# ---------- Security ----------
docker scout quickview nginx:latest
docker scout cves nginx:latest
docker scout recommendations nginx:latest
docker scout sbom nginx:latest

# ---------- Cleanup ----------
docker system prune -a
docker system prune -a --volumes
```

---

# ⭐ এক নজরে মূল পয়েন্ট (Class 1–4)

## ভিত্তি (Class 1)

- ✅ **Physical Server → VM → Container** — এই বিবর্তনের মূল কারণ **isolation** ও **resource efficiency**।
- ✅ **VM**-এ প্রতিটার আলাদা **full OS** → ভারী, ধীর, বেশি resource; **Hypervisor** VM manage করে।
- ✅ **Container** host-এর **OS kernel শেয়ার** করে → হালকা, দ্রুত (seconds), highly portable।
- ✅ **Docker** = পুরো platform; **containerd** = তার ভেতরের core runtime (ইঞ্জিন)।
- ✅ Docker চলে **client–server** model-এ: **Client (CLI) → Docker socket → Daemon (dockerd) → containerd**।
- ✅ মূল প্রবাহ: **Dockerfile → (build) → Image → (run) → Container**।
- ✅ Image থাকতে পারে **Local / Docker Hub / Amazon ECR**-এ।
- ✅ প্রথম `permission denied` = user `docker` group-এ নেই → `usermod -aG docker $USER` + **reboot/`newgrp docker`/relogin**।
- ✅ `docker ps` = শুধু চালু, `docker ps -a` = চালু + বন্ধ সব।
- ✅ Port mapping **`-p host:container`** — ডান পাশ (container port) fixed, বাম পাশ (host port) free যেকোনোটা।
- ✅ `EXPOSE` শুধু documentation; আসল access-এর জন্য `-p` লাগে।
- ✅ `daemon off;` না দিলে nginx container exit করে ফেলে — **container বাঁচে যতক্ষণ PID 1 চলে**।
- ✅ AWS-এ চালালে **Security Group**-এ port enable করতে ভুলবেন না।

## Build কৌশল (Class 2)

- ✅ `scp -i key.pem file ubuntu@ip:/path/` দিয়ে সার্ভারে ফাইল পাঠানো; key-তে `chmod 400` লাগে।
- ✅ `.dockerignore` (ছোট হাতের অক্ষরে) build দ্রুত করে, ইমেজ ছোট রাখে ও secret leak ঠেকায়।
- ✅ `COPY package*.json ./` আলাদা করে আগে দিলে **layer caching** কাজে লাগে।
- ✅ **Stage ≠ Container।** `AS builder`/`AS runner` হলো build stage-এর নাম; শেষ stage-টাই ফাইনাল ইমেজ, একটাই কন্টেইনার চলে।
- ✅ `COPY --from=builder /app/dist ...` — multi-stage-এর মূল কৌশল: শুধু দরকারি ফাইল তুলে আনা।
- ✅ **Production-এ সবসময় Multi-stage** — ইমেজ ~১০x ছোট, সোর্স কোড থাকে না, NGINX দ্রুত ও নিরাপদ।
- ✅ `try_files $uri $uri/ /index.html;` না দিলে SPA-তে refresh করলে 404 আসে।
- ✅ `VITE_` env variable ব্রাউজারে দৃশ্যমান — কখনো secret রাখা যাবে না।
- ✅ Docker build-এ `npm install`-এর চেয়ে **`npm ci`** ভালো (lock file হুবহু মানে, দ্রুত)।
- ✅ Alpine ইমেজে `bash` নেই — `docker exec -it <name> sh` ব্যবহার করতে হয়।

## Multi-container (Class 2)

- ✅ **Docker Compose** = এক YAML + এক কমান্ডে multi-container অ্যাপ: `docker compose up -d --build`।
- ✅ একই network-এ থাকা সার্ভিসরা **নাম দিয়ে** যোগাযোগ করে (`mongodb://mongo:27017`) — `localhost` নয়।
- ✅ **Network** যোগাযোগ ও isolation নিয়ন্ত্রণ করে; user-defined bridge-এ automatic DNS পাওয়া যায়।
- ✅ **Volume** ডেটা persist করে — কন্টেইনার মুছলেও DB-র ডেটা থাকে; **database মানেই volume**।
- ✅ Expense Tracker-এ **শুধু frontend-এর `ports: "80:80"`** আছে; backend ও mongo network-এর ভেতরে সুরক্ষিত।
- ✅ NGINX-এর `location /api/ { proxy_pass http://backend:5000/api/; }` = **reverse proxy** → CORS সমস্যা দূর হয়।
- ✅ Backend-এ `npm install --omit=dev`; কিন্তু frontend build-এ Vite নিজেই devDependency, তাই সেখানে দেওয়া যাবে না।
- ✅ Backend-এ multi-stage লাগে না (Node runtime রানটাইমেও দরকার), frontend-এ লাগে।
- ✅ `CMD ["node", "src/server.js"]` — `npm start`-এর চেয়ে ভালো, stop signal সরাসরি অ্যাপে পৌঁছায়।
- ✅ Compose-এ `volumes:` ও `networks:` টপ-লেভেলে **ডিক্লেয়ার করা বাধ্যতামূলক**; `version:` লাইনটা এখন obsolete।

## Image lifecycle ও Security (Class 3)

- ✅ Named volume ছাড়া DB container মুছলে সব data হারিয়ে যায়।
- ✅ `--network=my-network` ব্যবহারের আগে `docker network create` দিয়ে তৈরি করে নিতে হবে।
- ✅ **Code আপডেট করলে image আবার build করতে হবে** — শুধু restart-এ কাজ হবে না।
- ✅ `docker logs` = application-এর error/output; `docker inspect` = configuration।
- ✅ Docker Hub-এ push করতে নাম অবশ্যই `username/image-name:tag`, এবং login username ও tag username এক হতে হবে।
- ✅ `docker tag` নতুন image বানায় না — একই image ID-র জন্য আরেকটি নাম (alias)।
- ✅ Login-এ password-এর বদলে **Access Token (PAT)** নিরাপদ।
- ✅ **DevSecOps = Development + Security + Operations**; **CVE** = পরিচিত দুর্বলতার আন্তর্জাতিক তালিকা; **SBOM** = image-এর ভেতরের সব package ও version-এর তালিকা।
- ✅ Docker Scout-এর ৩টি প্রধান কমান্ড: `quickview` → `cves` → `recommendations`।
- ✅ **Production-এ `latest` tag নয়, নির্দিষ্ট version; password কখনো image-এর ভেতরে নয়।**

## Debugging (Class 4)

- ✅ ধাপ: **Observe → Inspect → Reproduce → Isolate → Fix → Verify → Prevent**।
- ✅ container fail করলে **আগে rebuild নয়** — আগে `docker ps -a` ও `docker logs`; issue না দেখে Dockerfile বদলানো যাবে না।
- ✅ Exit code: `0` সফল, `1` general error, `126` execute করা যায়নি, `127` command not found, `137` SIGKILL/OOM, `139` SIGSEGV, `143` SIGTERM।
- ✅ signal-এর exit code = **128 + signal number** (137 = 128+9)।
- ✅ `137` এলে নিশ্চিত হতে: `docker inspect --format '{{.State.OOMKilled}}' <id>`।
- ✅ container-এর ভেতরে **`localhost` মানে ঐ container নিজেই** — অন্য container-কে ডাকতে হবে **service-এর নাম** দিয়ে।
- ✅ অ্যাপকে **`0.0.0.0`**-তে bind করতে হবে, `127.0.0.1`-এ নয়।
- ✅ `depends_on` শুধু চালু হওয়া বোঝায়, **ready হওয়া নয়** — তাই healthcheck + `condition: service_healthy`।
- ✅ Volume কোনো ডিরেক্টরিকে **ঢেকে (mask)** দিতে পারে — "ফাইল হারিয়ে গেছে" মনে হলে আগে `Mounts` দেখুন।
- ✅ কোড বদলেও পুরোনো আচরণ দেখলে: `--build`, দরকারে `build --no-cache`।
- ✅ `502 Bad Gateway` মানে proxy ঠিক আছে, **upstream অ্যাপ সাড়া দিচ্ছে না**।
- ✅ Compose কমান্ডে `api` হলো **service-এর নাম**, `env` হলো container-এর ভেতরে চালানো **command** — কোনোটাই function নয়।
- ✅ একাধিক `-f` দিলে compose ফাইলগুলো merge হয়, পরেরটি আগেরটিকে override করে।
- ✅ ডিবাগের সময় restart policy সাময়িকভাবে বন্ধ রাখলে log ধরা সহজ হয়।