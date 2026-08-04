# Docker Class 4 — Container Debugging (কনটেইনার ডিবাগিং)

> এই ক্লাসের মূল বিষয়: কোনো Docker container-এ সমস্যা হলে **অনুমান না করে, ধাপে ধাপে প্রমাণ দেখে** সমাধান করা।

---

## ১. ডিবাগিং-এর ৭টি ধাপ (The Debugging Workflow)

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

✅ **Isolate** মানে শুধু "আলাদা করে ফেলা" নয় — মানে হলো সমস্যার পরিধি ছোট করে আনা। যেমন: পুরো stack বন্ধ করে শুধু database container চালিয়ে দেখা সে একা ঠিকমতো ওঠে কিনা।

---

## ২. সোনালি নিয়ম (Golden Rules) ✅

| নিয়ম | কারণ |
|---|---|
| ✅ container fail করলে **আগে rebuild করা যাবে না** | rebuild করলে আগের container ও তার log মুছে যায়, প্রমাণ হারিয়ে যায় |
| ✅ issue না দেখে **Dockerfile পরিবর্তন করা যাবে না** | কারণ না জেনে কোড বদলানো মানে অন্ধভাবে ঠিক করার চেষ্টা |
| ✅ প্রথম কাজ সবসময় **log পড়া** | ৮০% সমস্যার উত্তর log-এই লেখা থাকে |
| ✅ একবারে **একটি জিনিস** বদলাও | একসাথে ৩টা বদলালে কোনটা কাজ করেছে বোঝা যাবে না |
| ✅ `docker ps` নয়, `docker ps -a` | বন্ধ হয়ে যাওয়া (exited) container শুধু `-a` দিলেই দেখা যায় |

---

## ৩. প্রাথমিক কমান্ড (Basic Inspection Commands)

```bash
# সব container দেখা (বন্ধ হয়ে যাওয়াগুলোসহ)
docker ps -a

# log দেখা
docker logs <container_id>

# শেষ ১০০ লাইন + সময়সহ log
docker logs --tail 100 --timestamps <container_id>

# live log follow করা
docker logs -f <container_id>

# চলমান container-এর ভেতরে ঢোকা
docker exec -it <container_id> sh

# resource ব্যবহার (CPU/Memory) দেখা
docker stats

# container-এর ভেতরের process দেখা
docker top <container_id>
```

✅ `docker logs` শুধু সেই লেখাগুলোই দেখায় যা application **stdout / stderr**-এ পাঠায়। যদি অ্যাপ log লিখে কোনো ফাইলে (যেমন `/var/log/app.log`), তাহলে `docker logs` খালি দেখাবে — তখন `docker exec` দিয়ে ভেতরে ঢুকে ফাইলটা পড়তে হবে।

---

## ৪. Exit Code — container কী কারণে বন্ধ হলো

| Exit Code | মানে | বাস্তব কারণ | করণীয় |
|---|---|---|---|
| `Exited (0)` | Success | কাজ শেষ করে স্বাভাবিকভাবে বন্ধ | ✅ কিন্তু server/API-এর ক্ষেত্রে `0`-ও সমস্যা — মানে foreground-এ কোনো process চলছিল না |
| `Exited (1)` | General error | অ্যাপ crash, exception, config ভুল | `docker logs` পড়তেই হবে |
| `Exited (126)` | Command found কিন্তু execute হয়নি | permission নেই, ফাইল executable নয়, script-এ ভুল | `chmod +x`, ENTRYPOINT/CMD চেক |
| `Exited (127)` | Command not found | binary নেই, PATH ভুল, নামের typo | image-এ command আছে কিনা দেখা |
| `Exited (137)` | **SIGKILL (128 + 9)** | ✅ বেশিরভাগ সময় **OOM (Out Of Memory)** — memory limit পার হলে kernel জোর করে kill করে; অথবা `docker kill` | memory limit বাড়ানো / leak খোঁজা |
| `Exited (139)` | SIGSEGV (128 + 11) | segmentation fault, native binary crash | architecture/library mismatch দেখা |
| `Exited (143)` | SIGTERM (128 + 15) | `docker stop` দিলে graceful shutdown | সাধারণত স্বাভাবিক |

✅ **সূত্র:** signal-জনিত exit code = `128 + signal number`. তাই `137 = 128 + 9 = SIGKILL`, `143 = 128 + 15 = SIGTERM`.

---

## ৫. `docker inspect` দিয়ে গভীরে দেখা

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

# environment variables দেখা
docker inspect --format '{{json .Config.Env}}' <container_id>

# কোন network এ যুক্ত আছে
docker inspect --format '{{json .NetworkSettings.Networks}}' <container_id>
```

✅ `docker inspect` এবং `docker container inspect` — দুটোই একই কাজ করে। তবে `docker inspect` image/network/volume-এর ক্ষেত্রেও চলে, তাই container বোঝাতে স্পষ্টভাবে `docker container inspect` লেখা ভালো অভ্যাস।

---

## ৬. Docker Compose দিয়ে কাজ

```bash
# নির্দিষ্ট compose file দিয়ে container চালু করা (rebuild সহ)
docker compose -f compose.base.yml up -d --build

# কোন service কী অবস্থায় আছে দেখা
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

একাধিক `-f` দিলে Docker Compose ফাইলগুলোকে **উপরে-নিচে মিলিয়ে (merge)** নেয় — পরের ফাইলের মান আগের ফাইলের মানকে override করে। ডিবাগিং শেখার জন্য এটা দারুণ কৌশল: base ঠিক রেখে শুধু একটা সমস্যা "চাপিয়ে" দেওয়া যায়।

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

## ৭. ১৫টি বাস্তব সমস্যা (Real-World Scenarios)

### ১) Container exits during startup — শুরুতেই বন্ধ হয়ে যাওয়া

**লক্ষণ:** `docker ps -a` তে `Exited (1)` বা `Exited (0)`, container কয়েক সেকেন্ডও টেকে না।
**কারণ:** config ভুল, দরকারি env var নেই, dependency অনুপস্থিত, অথবা foreground-এ কোনো process নেই (যেমন Dockerfile-এ `CMD ["bash"]`)।

```bash
docker ps -a
docker logs <container_id>
docker inspect --format '{{.State.ExitCode}}' <container_id>
```

**Fix:** log-এর শেষ লাইনটাই সাধারণত আসল কারণ। ✅ মনে রাখতে হবে — container ততক্ষণই বাঁচে যতক্ষণ তার **main process (PID 1)** বেঁচে থাকে।

---

### ২) Restart loops — বারবার restart হতে থাকা

**লক্ষণ:** status বারবার `Restarting`, log-এ একই error বারবার।
**কারণ:** `restart: always` দেওয়া আছে, কিন্তু অ্যাপ প্রতিবার চালু হয়েই crash করছে।

```bash
docker ps -a
docker inspect --format '{{.RestartCount}}' <container_id>
docker logs --tail 50 <container_id>
```

**Fix:** ✅ ডিবাগ করার সময় **restart policy সাময়িকভাবে বন্ধ করে দিতে হয়** (`restart: "no"`), নইলে container বারবার নতুন হয়ে যাওয়ায় log ধরা কঠিন হয়। আসল crash-এর কারণ ঠিক করে তারপর policy ফিরিয়ে আনতে হবে।

---

### ৩) Wrong environment variables — ভুল env var

**লক্ষণ:** অ্যাপ চালু হয়, কিন্তু DB connect হয় না, বা "undefined"/"null" এরর।
**কারণ:** নামের typo (`DB_HOST` বনাম `DATABASE_HOST`), `.env` ফাইল লোড হয়নি, override ফাইল ভুল মান দিচ্ছে।

```bash
docker compose -f compose.base.yml exec api env
docker inspect --format '{{json .Config.Env}}' <container_id>
docker compose -f compose.base.yml config
```

**Fix:** ✅ `docker compose config` কমান্ডটি সব merge শেষে **চূড়ান্ত কনফিগ** দেখায় — কোন মান আসলে কার্যকর হচ্ছে তা এখানে ধরা পড়ে।

---

### ৪) `localhost` used between containers — কনটেইনারের মধ্যে localhost

**লক্ষণ:** `ECONNREFUSED 127.0.0.1:5432` জাতীয় এরর।
**কারণ:** ✅ প্রতিটি container-এর **নিজস্ব network namespace** থাকে। তাই container-এর ভেতরে `localhost` মানে **ঐ container নিজেই**, host মেশিন বা অন্য container নয়।

**Fix:** compose-এ service-এর নাম দিয়ে ডাকতে হবে, কারণ Docker-এর internal DNS ঐ নামটাকেই resolve করে।

```bash
# ভুল
DB_HOST=localhost
# ঠিক (service এর নাম db হলে)
DB_HOST=db

# যাচাই করা
docker compose -f compose.base.yml exec api ping -c 2 db
```

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

**মনে রাখতে হবে:** container-থেকে-container যোগাযোগে `ports` লাগেই না — সেটা internal network-এ এমনিতেই কাজ করে। `ports` শুধু **বাইরে থেকে (host)** ঢোকার জন্য।

---

### ৬) Application bound to `127.0.0.1` — ভুল ঠিকানায় bind

**লক্ষণ:** port mapping ঠিক আছে, container চলছে, তবু host থেকে ঢোকা যাচ্ছে না।
**কারণ:** অ্যাপ container-এর ভেতরে শুধু `127.0.0.1`-এ শুনছে, তাই বাইরের কোনো request গ্রহণ করছে না।

**Fix:** ✅ container-এ চলা অ্যাপ সবসময় **`0.0.0.0`**-তে bind করতে হবে।

```bash
# Node.js
app.listen(3000, "0.0.0.0")

# Python / Flask
app.run(host="0.0.0.0", port=5000)

# ভেতরে গিয়ে যাচাই
docker exec -it <container_id> sh -c "netstat -tulpn || ss -tulpn"
```

---

### ৭) Database not ready — ডেটাবেস প্রস্তুত হওয়ার আগেই অ্যাপ চালু

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

**Fix:** healthcheck + `condition: service_healthy`, এবং অ্যাপের কোডেও **retry logic** রাখা (প্রোডাকশনে DB যেকোনো সময় সাময়িক বন্ধ হতে পারে)।

---

### ৮) Volume hiding files — volume ফাইল ঢেকে দেওয়া

**লক্ষণ:** image-এ ফাইল ছিল, কিন্তু container-এ নেই; "module not found" জাতীয় এরর।
**কারণ:** ✅ কোনো ডিরেক্টরিতে volume/bind mount করলে সেটি image-এর ঐ ডিরেক্টরিকে **ঢেকে (mask) দেয়** — নিচের ফাইলগুলো আর দেখা যায় না।

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

### ১০) Health check failing — healthcheck ফেল করা

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
**কারণ:** নতুন করে build হয়নি, অথবা Docker পুরোনো layer cache থেকে নিয়েছে, অথবা `latest` tag-এর পুরোনো কপি local-এ রয়ে গেছে।

```bash
docker compose -f compose.base.yml up -d --build
docker compose -f compose.base.yml build --no-cache
docker compose -f compose.base.yml pull
docker image ls
docker inspect --format '{{.Image}}' <container_id>
```

**Fix:** ✅ প্রোডাকশনে `latest` tag ব্যবহার না করে **নির্দিষ্ট version tag** (যেমন `myapp:1.4.2`) ব্যবহার করাই নিরাপদ — তাহলে কোন কোড চলছে তা নিয়ে সন্দেহ থাকে না।

---

### ১৩) Missing startup executable — startup command না পাওয়া

**লক্ষণ:** `Exited (127)` — command not found, বা `exec format error`, বা `Exited (126)`।
**কারণসমূহ:**

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
**কারণ:** container দুটি ভিন্ন Docker network-এ আছে (যেমন দুটি আলাদা compose project থেকে চালু হয়েছে)। ✅ Docker-এর DNS দিয়ে নাম resolve **শুধু একই network-এর ভেতরেই** কাজ করে।

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
**কারণ:** ✅ 502 মানে proxy চলছে, কিন্তু **পেছনের অ্যাপ (upstream) সাড়া দিচ্ছে না**। সাধারণ কারণ:

| কারণ | সমাধান |
|---|---|
| `proxy_pass http://localhost:3000` লেখা | service-এর নাম দিতে হবে: `http://api:3000` |
| ভুল upstream port | container-এর internal port দিতে হবে, host port নয় |
| অ্যাপ container crash করেছে | `docker compose ps` + অ্যাপের log দেখা |
| proxy অ্যাপের আগে চালু হয়েছে | healthcheck / `depends_on` দিয়ে ক্রম ঠিক করা |
| অ্যাপ 127.0.0.1-এ bind করেছে | `0.0.0.0`-তে bind করা (দেখুন সিনারিও ৬) |

```bash
docker compose -f compose.base.yml logs --tail 100 nginx
docker compose -f compose.base.yml exec nginx curl -I http://api:3000
```

---

## ৮. প্রয়োজনীয় শব্দকোষ (Definitions + উদাহরণ)

### Exit Code
container-এর main process বন্ধ হওয়ার সময় যে সংখ্যা রেখে যায়, যা বলে দেয় কীভাবে বন্ধ হয়েছে।
- **উদাহরণ ১:** `Exited (0)` — backup script কাজ শেষ করে সফলভাবে বন্ধ হয়েছে।
- **উদাহরণ ২:** `Exited (137)` — 512MB limit-এর container 600MB memory নেওয়ায় OOM killer তাকে kill করেছে।

### Bind Address (`0.0.0.0` বনাম `127.0.0.1`)
অ্যাপ কোন network interface-এ request শুনবে তার ঠিকানা।
- **উদাহরণ ১:** `127.0.0.1:3000` — শুধু ঐ container-এর ভেতর থেকেই ঢোকা যাবে, বাইরে থেকে নয়।
- **উদাহরণ ২:** `0.0.0.0:3000` — সব interface-এ শুনবে, তাই `-p 8080:3000` দিয়ে host থেকে ঢোকা যাবে।

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
- **উদাহরণ ১:** `restart: always` — সার্ভার reboot হলেও container আবার উঠবে।
- **উদাহরণ ২:** `restart: on-failure:3` — ব্যর্থ হলে সর্বোচ্চ ৩ বার চেষ্টা করবে।

### OOM Kill (Out Of Memory)
memory limit পার হলে Linux kernel জোর করে process বন্ধ করে দেওয়া।
- **উদাহরণ ১:** বড় CSV ফাইল একবারে memory-তে লোড করায় 512MB limit-এর container kill হয়ে গেল।
- **উদাহরণ ২:** Node.js অ্যাপে memory leak-এর কারণে ধীরে ধীরে RAM বেড়ে ২ দিন পর 137 exit code।

### Build Cache
আগের build-এর layer পুনরায় ব্যবহার করে build দ্রুত করার ব্যবস্থা।
- **উদাহরণ ১:** `package.json` না বদলালে `npm install` layer cache থেকে নেওয়া হয়।
- **উদাহরণ ২:** cache-এর কারণে পুরোনো কোড থেকে যাওয়ায় `--no-cache` দিয়ে নতুন করে build করা।

### Reverse Proxy
বাইরের request প্রথমে গ্রহণ করে ভেতরের সঠিক service-এ পাঠিয়ে দেয় এমন সার্ভার।
- **উদাহরণ ১:** nginx `example.com/api` → `api:3000` container-এ পাঠায়।
- **উদাহরণ ২:** Traefik একই 80 পোর্টে একাধিক domain-কে আলাদা container-এ রুট করে।

---

## ৯. দ্রুত রেফারেন্স — লক্ষণ থেকে কমান্ড

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

## এক নজরে মূল পয়েন্ট

- ✅ ডিবাগিং-এর ধাপ: **Observe → Inspect → Reproduce → Isolate → Fix → Verify → Prevent**
- ✅ container fail করলে **আগে rebuild নয়** — আগে `docker ps -a` ও `docker logs`; issue না দেখে Dockerfile বদলানো যাবে না।
- ✅ `docker ps` বন্ধ container দেখায় না, তাই সবসময় **`docker ps -a`**।
- ✅ Exit code: `0` সফল, `1` general error, `126` execute করা যায়নি, `127` command not found, `137` SIGKILL/OOM, `143` SIGTERM।
- ✅ signal-এর exit code = **128 + signal number** (137 = 128+9)।
- ✅ `137` এলে নিশ্চিত হতে দেখুন: `docker inspect --format '{{.State.OOMKilled}}' <id>`।
- ✅ container-এর ভেতরে **`localhost` মানে ঐ container নিজেই** — অন্য container-কে ডাকতে হবে **service-এর নাম** দিয়ে।
- ✅ অ্যাপকে **`0.0.0.0`**-তে bind করতে হবে, `127.0.0.1`-এ নয়।
- ✅ Port mapping-এর নিয়ম: **`-p HOST:CONTAINER`**।
- ✅ `depends_on` শুধু চালু হওয়া বোঝায়, **প্রস্তুত (ready) হওয়া নয়** — তাই healthcheck + `condition: service_healthy` লাগে।
- ✅ Volume কোনো ডিরেক্টরিকে **ঢেকে (mask)** দিতে পারে — "ফাইল হারিয়ে গেছে" মনে হলে আগে `Mounts` দেখুন।
- ✅ কোড বদলেও পুরোনো আচরণ দেখলে: `--build`, দরকারে `build --no-cache`; প্রোডাকশনে `latest` নয়, **version tag**।
- ✅ `502 Bad Gateway` মানে proxy ঠিক আছে, **upstream অ্যাপ সাড়া দিচ্ছে না**।
- ✅ Compose কমান্ডে `api` হলো **service-এর নাম**, আর `env` হলো container-এর ভেতরে চালানো **command** — দুটোর কোনোটাই function নয়।
- ✅ একাধিক `-f` দিলে compose ফাইলগুলো merge হয়, পরেরটি আগেরটিকে override করে।
- ✅ ডিবাগ করার সময় restart policy সাময়িকভাবে বন্ধ রাখলে log ধরা সহজ হয়।