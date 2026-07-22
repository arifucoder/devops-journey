# 🖥️ Computer Networking — Class 1

> ক্লাসের নোট, সহজ ভাষায় সাজানো। টেকনিক্যাল শব্দগুলো ইংরেজিতেই রাখা হয়েছে।

---

## 🏤 নেটওয়ার্কিং আসলে কী? (Post Office উপমা)

Internet অনেকটা অফলাইনের **post office**-এর মতো করেই কাজ করে।

- Post office থেকে কোনো জিনিস পাঠাতে গেলে যেমন **নিজের নাম আর ঠিকানা** দিতে হয়, ঠিক তেমনি প্রতিটা computer-এরও একটা নিজস্ব **address** থাকে।
- Post office যেমন **tracking** করে — জিনিসটা এখন কোথায় আছে, কখন delivery হবে — নেটওয়ার্কিংয়েও ঠিক তেমন একটা ব্যবস্থা আছে, যাকে বলে **TCP handshake** (দুই device আগে "হ্যান্ডশেক" করে নিশ্চিত হয়ে তারপর data পাঠায়)।

### 📦 Data Packet (খামের উপমা)

আগের যুগে ধরো একটা **১০০০ পৃষ্ঠার বই** পাঠাতে চাইলে সেটা একসাথে পাঠানো যেত না। তখন বইটাকে ভাগ ভাগ করে **১০০০টা খামে (envelope)** ভরে পাঠাতে হতো।

- সিরিয়াল ঠিক রাখার জন্য প্রতিটা খামে **1, 2, 3 … 1000** এভাবে নাম্বার লিখে দেওয়া হতো, যাতে ওপাশে গিয়ে ঠিক ক্রমে আবার জোড়া লাগানো যায়।

ঠিক একইভাবে YouTube video হোক বা যেকোনো data — সেটা ছোট ছোট টুকরো হয়ে আসে। এই টুকরোগুলোকেই বলে **data packet**। কয়েক **millisecond**-এর মধ্যেই এই packet-গুলো আবার একসাথে **assemble** হয়ে আমাদের সামনে চলে আসে।

👉 এই পুরো ব্যাপারটাকেই বলে **Computer Networking**।

---

## 💻 Computer কী?

একাধিক computer যখন একে অপরের সাথে **communicate** করতে পারে, সেটাই হলো **computer networking**।

> ⚠️ **মনে রাখবে:** ক্লাসে অনেক সময় বলা হয় "COMPUTER = Common Operating Machine Purposely Used for Technological and Educational Research"। এটা মূলত একটা মজার **backronym** (পরে বানানো পূর্ণরূপ) — আসল ইতিহাস নয়। "Computer" শব্দটা এসেছে ইংরেজি **"compute"** (হিসাব করা) থেকে। ক্লাসে জিজ্ঞেস করলে বলতে পারো, তবে জেনে রেখো এটা প্রকৃত acronym নয়।

---

## 🌐 Internet কী?

**Internet** হলো পৃথিবীজুড়ে ছড়িয়ে থাকা অসংখ্য **computer network-এর সমষ্টি** (collection of computer networks around the world)।

---

## 📍 IP Address

প্রতিটা smart device — যেমন PC, laptop, mobile, smart fan, smart AC — সবগুলোরই একটা করে **IP address** থাকে। এই address দিয়েই device-টাকে চেনা যায় এবং তার কাছে data পাঠানো যায়।

---

## 📜 Protocol কী?

**Protocol** হলো কিছু **নিয়ম বা rules-এর সেট**, যেগুলো ঠিক করে দেয় — data কীভাবে এক device থেকে আরেক device-এ, এক network থেকে আরেক network-এ পাঠানো হবে। এটা নিশ্চিত করে যে data **সঠিক address-এ, সঠিকভাবে** পৌঁছেছে।

> ⚠️ **ছোট্ট সংশোধন:** protocol নিজে "addressing system" নয়। *Addressing*-এর কাজটা করে **IP (Internet Protocol)**। Protocol হলো মূলত **যোগাযোগের নিয়মকানুন**, আর সেই নিয়মের ভেতরেই ঠিকানা ঠিক রাখার ব্যাপারটা আসে।

### গুরুত্বপূর্ণ কিছু Protocol

| Protocol | পূর্ণরূপ | কী কাজে লাগে |
|---|---|---|
| **TCP/IP** | Transmission Control Protocol / Internet Protocol | Internet-এ data কীভাবে transmit হবে তার মূল নিয়ন্ত্রক (core suite of protocols) |
| **UDP** | User Datagram Protocol | দ্রুতগতির data পাঠানোর জন্য (video conferencing ইত্যাদি)। খুব **speedy**, তবে মাঝেমধ্যে কিছু data **loss** হতে পারে |
| **HTTP** | HyperText Transfer Protocol | Web communication — browser আর web server-এর মধ্যে যোগাযোগ |
| **FTP** | File Transfer Protocol | এক জায়গা থেকে আরেক জায়গায় **file transfer** করা |
| **SMTP** | Simple Mail Transfer Protocol | **email** পাঠানোর জন্য |
| **DNS** | Domain Name System | Domain name (যেমন `google.com`) কে IP address-এ অনুবাদ করে দেয় |

> 💡 **DNS একটু সহজ করে:** আমরা মনে রাখি নাম (`youtube.com`), কিন্তু computer বোঝে সংখ্যা (IP address)। DNS হলো সেই "ফোনবুক", যে নামটা দেখে ঠিক IP address বের করে দেয়।
>
> 📝 অনেক জায়গায় DNS-কে "Domain Name **Service**"-ও বলা হয়, তবে standard নাম হলো "Domain Name **System**"।

---

## 🔍 নিজের PC-র IP Address জানার উপায়

Terminal-এ নিচের command চালালেই নিজের public IP চলে আসবে:

```bash
curl ifconfig.me -s
```

> ⚠️ **টাইপো ঠিক করা হলো:** command-টা `iconfig.me` নয়, হবে **`ifconfig.me`** (মাঝে একটা **`f`** আছে)। `-s` মানে "silent" — অতিরিক্ত progress লেখা লুকিয়ে রাখে।

---

## 🔢 IP-এর প্রকারভেদ

### ১. Version অনুযায়ী

- **IPv4** — পুরনো ও সবচেয়ে প্রচলিত। যেমনঃ `192.168.0.1`
- **IPv6** — নতুন সংস্করণ, অনেক বেশি address ধারণ করতে পারে। যেমনঃ `2001:0db8:85a3::8a2e:0370:7334`

### ২. স্থায়িত্ব অনুযায়ী

- **Static IP** — নির্দিষ্ট, বদলায় না। (server-এ বেশি ব্যবহার হয়)
- **Dynamic IP** — সময়ের সাথে বদলাতে থাকে। সাধারণত ISP প্রতিবার connect হলে নতুন করে দেয়।

---

## 🚪 Port

শুধু IP address জানলেই হয় না — একটা device-এর ভেতরে তো অনেকগুলো application একসাথে চলে (browser, email, game ইত্যাদি)।

- **IP address** ঠিক করে দেয় → **কোন device**-এ data যাবে।
- **Port** ঠিক করে দেয় → সেই device-এর ভেতরে **কোন application**-এ data যাবে।

এভাবেই data একেবারে **নির্দিষ্ট device-এর নির্দিষ্ট application** পর্যন্ত সঠিকভাবে পৌঁছায়।

> 💡 কিছু পরিচিত port নাম্বার (bonus):
> `HTTP → 80`, `HTTPS → 443`, `FTP → 21`, `SMTP → 25`, `DNS → 53`

---

## ✅ এক নজরে সারসংক্ষেপ

1. **Networking** = post office-এর মতো, address দিয়ে data এক জায়গা থেকে আরেক জায়গায় পাঠানো।
2. **Data packet** = বড় data ভাগ হয়ে numbered টুকরো হিসেবে যায়, ওপাশে গিয়ে জোড়া লাগে।
3. **Computer networking** = একাধিক computer-এর নিজেদের মধ্যে যোগাযোগ।
4. **Internet** = পৃথিবীজুড়ে সব network-এর সমষ্টি।
5. **IP address** = প্রতিটা device-এর ঠিকানা।
6. **Protocol** = যোগাযোগের নিয়মকানুন (TCP/IP, UDP, HTTP, FTP, SMTP, DNS)।
7. **Port** = device-এর ভেতরে নির্দিষ্ট application-এর ঠিকানা।

---

*📚 Computer Networking — Class 1 শেষ*