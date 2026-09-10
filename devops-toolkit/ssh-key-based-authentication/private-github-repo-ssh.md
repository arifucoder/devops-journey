# Private GitHub Repository → VPS/AWS Instance এ SSH দিয়ে Clone করা (CI/CD Setup)

যদি একটি **private GitHub repository** থাকে এবং সেটাকে CI/CD এর মাধ্যমে বা সরাসরি কোনো **VPS/AWS instance** এ clone করতে চান, তাহলে সেই মেশিনে একটি **SSH key pair (public + private key)** তৈরি করে GitHub এর সাথে যুক্ত (link) করে নিতে হবে। নিচে ধাপে ধাপে পুরো প্রক্রিয়াটি দেওয়া হলো।

---

## ধাপ ১: VPS এর Home Directory তে যাওয়া

প্রথমে আপনার VPS/AWS instance এ SSH করে ঢুকুন এবং home directory তে যান (`~` মানে হলো home directory):

```bash
cd ~
```

এরপর `ll` (বা `ls -la`) কমান্ড দিয়ে দেখুন `.ssh` নামে একটি folder আছে কিনা:

```bash
ll
```

---

## ধাপ ২: `.ssh` Folder এ ঢোকা

```bash
cd .ssh
ll
```

এখানে ঢুকে দেখুন ভেতরে আগে থেকে কোনো key file আছে কিনা।

---

## ধাপ ৩: নতুন SSH Key Generate করা

নিচের কমান্ড দিয়ে নতুন একটি SSH key তৈরি করুন:

```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

কমান্ড রান করার পর কয়েকটি প্রশ্ন আসবে:

1. **"Enter file in which to save the key"** → এখানে একটা নাম দিন (যেমন: `github_key`)
2. **Passphrase** → খালি রেখে সরাসরি Enter চাপুন
3. আবার Enter চাপুন (confirm করার জন্য)

এই ধাপ শেষ হলে **২টি file** তৈরি হবে:
- `github_key` → এটা **private key** (কোথাও শেয়ার করবেন না)
- `github_key.pub` → এটা **public key** (এটাই GitHub এ যুক্ত করতে হবে)

---

## ধাপ ৪: Public Key কপি করা

`.pub` extension এর file টি খুলে ভেতরের কন্টেন্ট কপি করুন:

```bash
cat github_key.pub
```

পুরো output টি (ssh-rsa দিয়ে শুরু হওয়া লাইনটি) কপি করে রাখুন।

---

## ধাপ ৫: GitHub এ Public Key যুক্ত করা

1. GitHub প্রোফাইলে যান → **Settings**
2. বাম পাশের মেনু থেকে **SSH and GPG keys** এ ক্লিক করুন
3. **New SSH key** বাটনে ক্লিক করুন
4. একটি **Title/Name** দিন (যেমন: `My VPS Key`)
5. **Key** ফিল্ডে আগে কপি করা public key টি পেস্ট করুন
6. **Add SSH key** বাটনে ক্লিক করে সেভ করুন

---

## ধাপ ৬: VPS এ ফিরে গিয়ে SSH Agent চালু করা

আবার VPS এর home directory তে ফিরে যান:

```bash
cd ~
```

SSH agent চালু (restart) করুন:

```bash
eval "$(ssh-agent -s)"
```

> লক্ষ্য করুন: `$` এবং `(` এর মাঝে কোনো space থাকবে না — সঠিক লেখা হলো `$(ssh-agent -s)`।

---

## ধাপ ৭: Private Key কে SSH Agent এ যুক্ত করা

```bash
ssh-add ~/.ssh/github_key
```

সফল হলে এরকম একটি message দেখাবে যে identity টি (key) agent এ যুক্ত হয়েছে।

---

## ধাপ ৮ (Optional কিন্তু সুপারিশকৃত): Connection Test করা

Clone করার আগে যাচাই করে নিন GitHub এর সাথে SSH connection ঠিকভাবে কাজ করছে কিনা:

```bash
ssh -T git@github.com
```

সফল হলে GitHub থেকে একটি welcome message আসবে, যেখানে আপনার username উল্লেখ থাকবে।

---

## ধাপ ৯: Repository Clone করা

এখন SSH link ব্যবহার করে private repository টি clone করতে পারবেন:

```bash
git clone git@github.com:username/repository-name.git
```

এভাবে CI/CD pipeline এ বা সরাসরি VPS/AWS instance এ private repository টি ব্যবহার করা যাবে।

---

## সংক্ষিপ্ত সারমর্ম (Summary)

| ধাপ | কাজ |
|---|---|
| ১-২ | `.ssh` folder এ যাওয়া |
| ৩ | `ssh-keygen` দিয়ে key pair তৈরি করা |
| ৪-৫ | Public key কপি করে GitHub এ যুক্ত করা |
| ৬-৭ | SSH agent চালু করে private key যুক্ত করা |
| ৮ | Connection টেস্ট করা |
| ৯ | `git clone` দিয়ে repository নামানো |