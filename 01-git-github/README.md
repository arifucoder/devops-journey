# 🌿 Git ও GitHub সম্পূর্ণ গাইড (বাংলায়)

দৈনন্দিন কাজে সবচেয়ে বেশি লাগে এমন Git ও GitHub-এর সব command, নিয়ম ও workflow — এক জায়গায়, গুছিয়ে, বাংলায়।

---

## 📚 সূচিপত্র

- [Part 1 — Setup ও Configuration](#part-1--setup-ও-configuration)
  - [প্রথম Repository তৈরি](#প্রথম-repository-তৈরি)
  - [Remote — GitHub-এর সাথে সংযোগ](#remote--githubএর-সাথে-সংযোগ)
  - [Git Config — Global vs Local (নাম/ইমেইল)](#git-config--global-vs-local-নামইমেইল)
  - [এক Folder-এ আলাদা Account + Access Token দিয়ে Push](#এক-folderএ-আলাদা-account--access-token-দিয়ে-push)
  - [SSH Key Setup (macOS)](#ssh-key-setup-macos)
- [Part 2 — দৈনন্দিন Workflow](#part-2--দৈনন্দিন-workflow)
  - [Git-এর তিনটা জায়গা](#gitএর-তিনটা-জায়গা)
  - [git status — অবস্থা দেখা](#git-status--অবস্থা-দেখা)
  - [git add — Stage করা](#git-add--stage-করা)
  - [git commit — Commit করা](#git-commit--commit-করা)
  - [git diff — পার্থক্য দেখা](#git-diff--পার্থক্য-দেখা)
  - [git log — ইতিহাস দেখা](#git-log--ইতিহাস-দেখা)
  - [git show — একটা commit-এর বিস্তারিত](#git-show--একটা-commitএর-বিস্তারিত)
- [Part 3 — Semantic Commit Messages](#part-3--semantic-commit-messages)
- [Part 4 — Branching ও Merging](#part-4--branching-ও-merging)
- [Part 5 — Undo ও উদ্ধার](#part-5--undo-ও-উদ্ধার)
- [Part 6 — Remote ও GitHub (push, pull, fetch)](#part-6--remote-ও-github-push-pull-fetch)
- [Part 7 — Collaboration (Fork, Pull Request, Conflict)](#part-7--collaboration-fork-pull-request-conflict)
- [Part 8 — Quick Reference](#part-8--quick-reference)

---

# Part 1 — Setup ও Configuration

## প্রথম Repository তৈরি

| Command | কাজ কী |
|---------|--------|
| `git init` | নতুন local Git repository তৈরি করে |
| `git clone <repo-url>` | GitHub থেকে পুরো repository local-এ copy করে |
| `git branch -M main` | বর্তমান branch-এর নাম `main` করে দেয় (`-M` = **Move / force rename**) |

## Remote — GitHub-এর সাথে সংযোগ

`remote` = আপনার local repo কোন online repo-র সাথে যুক্ত, তার নাম-ঠিকানা।

| Command | কাজ কী |
|---------|--------|
| `git remote add origin <repo-url>` | Local repo-র সাথে GitHub repo যুক্ত করে (`origin` নামে) |
| `git remote -v` | কোন কোন remote যুক্ত আছে, URL সহ দেখায় (`-v` = **verbose**) |
| `git remote set-url origin <new-url>` | `origin`-এর URL পরিবর্তন করে |
| `git remote rename origin upstream` | Remote-এর নাম বদলায় |
| `git remote remove origin` | Remote মুছে ফেলে |

> **💡 নতুন project GitHub-এ তোলার পূর্ণ workflow:**
> ```bash
> git init
> git add .
> git commit -m "first commit"
> git branch -M main
> git remote add origin https://github.com/username/repo.git
> git push -u origin main
> ```

---

## Git Config — Global vs Local (নাম/ইমেইল)

Commit-এ যে নাম-ইমেইল দেখায় সেটা আসে Git config থেকে। Config দুই স্তরে থাকে:

- **Global** (`--global`) → আপনার পুরো PC-র সব repo-তে ডিফল্ট হিসেবে কাজ করে। থাকে `~/.gitconfig` file-এ।
- **Local** (কোনো flag ছাড়া, repo-র ভেতরে চালালে) → শুধু ওই একটা repo/folder-এ কাজ করে। থাকে `.git/config` file-এ।

> **📌 নিয়ম:** Local config সবসময় global-কে **override** করে। অর্থাৎ কোনো folder-এ local user.email সেট করা থাকলে, ওই folder-এ global-এর বদলে local-টাই কাজ করবে।

**Global (main account) সেট করা:**

```bash
git config --global user.name "arifucoder"
git config --global user.email "arifucoder@gmail.com"
```

**এখন কী সেট আছে দেখা:**

```bash
git config --global user.name      # global নাম
git config --global user.email     # global ইমেইল
git config user.email              # এই folder-এ এখন কোনটা কার্যকর
```

> ⚠️ `user.email`-টা অবশ্যই GitHub account-এ **verified email** হতে হবে (**Settings → Emails**)। নাহলে commit-এর পাশে profile picture/link দেখাবে না। এই setting device-ভিত্তিক — প্রতিটা PC/Mac-এ আলাদা করে সেট করতে হয়, আর শুধু নতুন commit-এ কাজ করে (পুরনো commit-এর নাম বদলায় না)।

---

## এক Folder-এ আলাদা Account + Access Token দিয়ে Push

**পরিস্থিতি:** আপনার PC-তে globally আপনার **main GitHub account** সেট করা আছে। কিন্তু একটা নির্দিষ্ট folder থেকে আপনি **অন্য একটা GitHub account**-এ push করতে চান — Personal Access Token (PAT) ব্যবহার করে। Global account-কে না ঘেঁটে শুধু ওই একটা folder-এ আলাদা identity দিতে হবে।

### ধাপ ১ — শুধু ওই folder-এ local নাম/ইমেইল সেট করুন

```bash
cd /path/to/that/repo          # প্রথমে ওই নির্দিষ্ট folder-এ ঢুকুন
git config user.name "second-account-username"
git config user.email "second-account-email@example.com"
```

> এখানে `--global` **লেখা হয়নি** বলেই এটা শুধু এই repo-তে কাজ করবে। বাকি সব folder-এ আপনার main account (global) অটুট থাকবে।

### ধাপ ২ — Remote URL-এ Access Token বসিয়ে দিন

দ্বিতীয় account-এ push করার জন্য ওই account-এর token ব্যবহার করতে হবে। remote URL-এ token বসিয়ে দিন:

```bash
git remote set-url origin https://<ACCESS_TOKEN>@github.com/second-username/repo-name.git
```

অথবা username সহ:

```bash
git remote set-url origin https://second-username:<ACCESS_TOKEN>@github.com/second-username/repo-name.git
```

> **🔑 Token কোথায় পাবেন:** দ্বিতীয় account-এ GitHub.com → **Settings → Developer settings → Personal access tokens → Generate new token**। Token-টা `repo` scope দিয়ে বানান এবং নিরাপদে রাখুন — একবার হারালে আর দেখা যায় না।

### ধাপ ৩ — যাচাই ও Push

```bash
git config user.name       # এই folder-এ দ্বিতীয় account দেখাচ্ছে কিনা
git config user.email
git remote -v              # remote-এ token/second-username আছে কিনা
git add .
git commit -m "message"
git push origin main
```

### কোন config কোথা থেকে আসছে বুঝতে

```bash
git config --local --list                 # শুধু এই repo-র config
git config --global --list                # পুরো PC-র global config
git config --show-origin user.email       # user.email কোন file থেকে আসছে তা দেখায়
```

> **💡 সারমর্ম:** Global = main account (সব জায়গায়)। এক folder-এ ঢুকে `--global` ছাড়া `user.name`/`user.email` সেট করলে + remote URL-এ token বসালে → শুধু ওই folder দ্বিতীয় account দিয়ে চলবে, বাকি সব আগের মতোই থাকবে।

> **⚠️ নিরাপত্তা:** Token সরাসরি remote URL-এ বসালে সেটা `.git/config`-এ plain text হিসেবে থেকে যায়। বেশি নিরাপত্তা চাইলে Git credential manager বা SSH key (নিচে দেখুন) ব্যবহার করা ভালো।

---

## SSH Key Setup (macOS)

> GitHub username change করার পর বা নতুন machine-এ পুরনো SSH key remove করে নতুন key setup করার সম্পূর্ণ গাইড।

### Step 1 — GitHub থেকে পুরনো key delete করুন

**GitHub.com → Settings → SSH and GPG keys** → প্রতিটা পুরনো key-এর পাশে **Delete** → confirm করুন। (Password বা 2FA চাইতে পারে।)

### Step 2 — PC থেকে পুরনো key file delete করুন

```bash
ls ~/.ssh                                    # কী কী key আছে দেখুন
rm ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub   # key file গুলো delete (নাম অনুযায়ী)
```

### Step 3 — SSH agent থেকে clear করুন

```bash
ssh-add -D
```

### Step 4 — Config file check করুন

```bash
cat ~/.ssh/config
```

শুধু GitHub-এর পুরনো key-এর entry থাকলে পুরো file মুছে দিতে পারেন:

```bash
rm ~/.ssh/config
```

> ⚠️ অন্য কোনো server-এর configuration থাকলে পুরো file না মুছে শুধু GitHub-এর অংশটুকু edit করে বাদ দিন।

সব clean হয়েছে কিনা যাচাই করুন:

```bash
ls ~/.ssh
```

এখন শুধু `known_hosts` আর `known_hosts.old` থাকার কথা — এ দুটো রেখে দিলে সমস্যা নেই।

### Step 5 — নতুন SSH key generate করুন

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

- Email-এর জায়গায় আপনার **GitHub account-এর email** দিন
- File location জিজ্ঞেস করলে **Enter** (default থাকুক)
- Passphrase চাইলে দিন, বা খালি রেখে **Enter**

### Step 6 — SSH agent-এ key add করুন

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Step 7 — Public key copy করুন

```bash
cat ~/.ssh/id_ed25519.pub
pbcopy < ~/.ssh/id_ed25519.pub    # macOS-এ সরাসরি clipboard-এ copy
```

> ⚠️ শুধু `.pub` file-টাই (public key) copy করবেন। `id_ed25519` (private key) **কখনোই** কোথাও paste করবেন না।

### Step 8 — GitHub-এ key add করুন

**GitHub.com → Settings → SSH and GPG keys → New SSH key**

- **Title:** device চেনার মতো একটা নাম (যেমন `My MacBook`)
- **Key type:** `Authentication Key` (default)
- **Key:** copy করা public key paste করুন → **Add SSH key**

### Step 9 — Connection test করুন

```bash
ssh -T git@github.com
```

প্রথমবার `Are you sure you want to continue connecting?` জিজ্ঞেস করলে `yes` দিন। তারপর দেখবেন:

```
Hi NEW-USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

এটা দেখলেই বুঝবেন সব ঠিকঠাক ✅

### Step 10 — Repo-গুলোর remote URL update করুন

```bash
cd /path/to/your/repo
git remote -v                                                    # এখন কী set করা আছে
git remote set-url origin git@github.com:NEW-USERNAME/repo-name.git   # নতুন username দিয়ে update
git remote -v                                                    # যাচাই
git pull                                                         # ঠিকঠাক কাজ করলে সব সেট 🎉
```

---

# Part 2 — দৈনন্দিন Workflow

## Git-এর তিনটা জায়গা

সব command এই তিন ধাপের মাঝেই কাজ করে — এটা মাথায় রাখলে বাকি সব সহজ।

```
Working Directory  →  Staging Area  →  Repository
   (আপনি যেখানে         (git add)        (git commit)
    edit করেন)
```

- **Working Directory** — আপনার আসল file, যেখানে আপনি লিখছেন
- **Staging Area** — যেসব পরিবর্তন পরের commit-এ যাবে, তাদের অপেক্ষার ঘর
- **Repository** — commit হিসেবে স্থায়ীভাবে জমা হওয়া ইতিহাস

---

## git status — অবস্থা দেখা

এই মুহূর্তে repository-র অবস্থা দেখায় — কোন file বদলেছে, কোনটা staged, কোনটা untracked, আর কোন branch-এ আছেন।

```bash
git status          # পূর্ণ অবস্থা
git status -s        # short/সংক্ষিপ্ত ফরম্যাটে
git status -sb       # সংক্ষিপ্ত + branch-এর নামসহ
```

**`-s` output পড়ার নিয়ম:**

| চিহ্ন | মানে |
|---|---|
| `??` | Untracked — Git এই file-টা এখনো চেনে না |
| `M` (বাম ঘরে) | Staged modification |
| `M` (ডান ঘরে) | Modified কিন্তু এখনো staged হয়নি |
| `A` | নতুন file যোগ করা হয়েছে (staged) |
| `D` | Deleted |

> 💡 কনফিউজড লাগলে সবার আগে `git status` চালান — এটা সাধারণত পরের করণীয়ও suggest করে দেয়।

---

## git add — Stage করা

Working directory-র পরিবর্তন **staging area**-তে নিয়ে যায়, অর্থাৎ "এগুলো পরের commit-এ রাখতে চাই" — Git-কে জানায়। `git add` না করলে commit-এ কিছুই যাবে না।

```bash
git add file.txt           # নির্দিষ্ট একটা file
git add file1.js file2.js  # একাধিক file একসাথে
git add .                  # বর্তমান folder ও তার ভেতরের সব পরিবর্তন
git add -A                 # পুরো repo-র সব পরিবর্তন (delete সহ)
git add *.md               # নির্দিষ্ট pattern-এর সব file
git add -p                 # একই file-এর কিছু অংশ নেবেন, কিছু নেবেন না
```

`-p` (patch mode) একেকটা টুকরো দেখিয়ে জিজ্ঞেস করে — `y` (নেব), `n` (নেব না), `s` (আরও ছোট টুকরো), `q` (বেরিয়ে যাও)। বড় কাজকে ছোট ছোট পরিষ্কার commit-এ ভাগ করার জন্য দারুণ।

**ভুল করে add করলে (unstage):**

```bash
git restore --staged file.txt   # আধুনিক পদ্ধতি
git reset HEAD file.txt          # পুরোনো পদ্ধতি, একই কাজ
```

---

## git commit — Commit করা

Staging area-র পরিবর্তনগুলোর একটা স্থায়ী **snapshot** তৈরি করে ইতিহাসে জমা রাখে। প্রতিটা commit-এর unique ID (hash), লেখকের নাম, সময় ও message থাকে।

```bash
git commit                        # editor খুলে message লিখতে বলবে
git commit -m "Add login form"    # সরাসরি message দিয়ে commit
git commit -am "Fix typo"         # tracked file গুলো add + commit একসাথে
                                  # ⚠️ untracked (নতুন) file এতে যাবে না
```

**শেষ commit ঠিক করা:**

```bash
git commit --amend               # শেষ commit-এর message/content বদলান
git commit --amend --no-edit     # ভুলে যাওয়া file যোগ করুন, message একই থাকবে
```

> ⚠️ `--amend` ইতিহাস বদলে দেয়। যে commit ইতিমধ্যে push হয়ে গেছে, সেটাতে amend করবেন না (যদি না আপনি একাই ওই branch ব্যবহার করেন)।

---

## git diff — পার্থক্য দেখা

`git status` বলে "কোন file বদলেছে", আর `git diff` বলে "সেই file-এ ঠিক **কোন লাইন** বদলেছে"।

```bash
git diff                    # unstaged পরিবর্তন (working dir vs staging)
git diff --staged           # staged পরিবর্তন (staging vs শেষ commit)
git diff HEAD               # শেষ commit-এর তুলনায় সব পরিবর্তন
git diff file.txt           # শুধু একটা file-এর diff
git diff --stat             # শুধু সারমর্ম — কোন file-এ কয় লাইন বদলেছে
```

**তুলনা করার আরও উপায়:**

```bash
git diff main feature       # দুই branch-এর মধ্যে পার্থক্য
git diff abc123 def456      # দুই commit-এর মধ্যে পার্থক্য
git diff HEAD~1 HEAD        # শেষ commit-টা ঠিক কী বদলেছিল
```

**Diff পড়ার নিয়ম:**

| চিহ্ন | মানে |
|---|---|
| `-` (লাল) | এই লাইনটা মুছে গেছে |
| `+` (সবুজ) | এই লাইনটা যোগ হয়েছে |
| `@@ -3,7 +3,8 @@` | কোন লাইন নম্বর থেকে পরিবর্তন শুরু |

> 💡 commit করার আগে সবসময় `git diff --staged` চালিয়ে দেখে নিন ঠিক কী commit হতে যাচ্ছে। অনেক ভুল এখানেই ধরা পড়ে।

---

## git log — ইতিহাস দেখা

Commit-এর ইতিহাস দেখায় — নতুনটা সবার উপরে। লম্বা output হলে **`q` চেপে** বেরোবেন, তীর চিহ্নে scroll করবেন।

```bash
git log                     # পূর্ণ ইতিহাস
git log --oneline           # প্রতি commit এক লাইনে (hash + message)
git log -5                  # শেষ ৫টা commit
git log --author="Rahim"    # নির্দিষ্ট লেখকের commit
git log --since="2 weeks ago"   # গত ২ সপ্তাহের commit
git log file.txt            # শুধু এই file-এর ইতিহাস
git log --grep="bug"        # message-এ "bug" আছে এমন commit
git log -p                  # প্রতিটা commit-এর সাথে পুরো diff
```

**সবচেয়ে কাজের combo (branch structure সহ):**

```bash
git log --oneline --graph --all --decorate
```

- `--graph` — branch ও merge-এর গঠন ASCII ছবিতে
- `--all` — সব branch দেখায়
- `--decorate` — branch ও tag-এর নাম দেখায়

**এটা alias বানিয়ে রাখুন:**

```bash
git config --global alias.lg "log --oneline --graph --all --decorate"
```

এরপর শুধু `git lg` লিখলেই হবে।

---

## git show — একটা commit-এর বিস্তারিত

```bash
git show abc123           # নির্দিষ্ট commit-এর পুরো info + diff
git show HEAD             # সর্বশেষ commit
git show HEAD~2           # ২টা commit আগেরটা
git show abc123 --stat    # শুধু কোন কোন file বদলেছে তার সারমর্ম
git show abc123:file.txt  # ওই commit-এ file.txt-এর content
```

---

# Part 3 — Semantic Commit Messages

> কমিট মেসেজ লেখার একটি গোছানো নিয়ম, যা দেখলেই বোঝা যায় কমিটে কী পরিবর্তন হয়েছে।

## কেন ব্যবহার করবেন?

কমিট হিস্টোরি পড়া সহজ হয়, এক নজরে কী বদলেছে বোঝা যায়, changelog অটোমেট করা যায়, পুরো টিম একই নিয়ম মেনে চলে, আর `semantic-release`-এর মতো টুল দিয়ে version bump অটোমেট করা যায়।

## ফরম্যাট

```
<type>(<scope>): <subject>
```

- **type** → পরিবর্তনের ধরন (আবশ্যক)
- **scope** → কোন অংশে পরিবর্তন (ঐচ্ছিক)
- **subject** → সংক্ষিপ্ত বর্ণনা (আবশ্যক)

**উদাহরণ:** `feat(auth): add google login support`

## Type-গুলোর তালিকা

| Type | কখন ব্যবহার করবেন | উদাহরণ |
|------|-------------------|---------|
| `feat` | নতুন ফিচার যোগ করলে | `feat: add dark mode toggle` |
| `fix` | বাগ ঠিক করলে | `fix: resolve login crash on empty password` |
| `docs` | শুধু ডকুমেন্টেশন পরিবর্তন | `docs: update README installation guide` |
| `style` | ফরম্যাটিং পরিবর্তন (লজিকে প্রভাব নেই) | `style: fix indentation in header.js` |
| `refactor` | কোড পুনর্গঠন (ফিচারও না, ফিক্সও না) | `refactor: simplify user validation logic` |
| `perf` | পারফরম্যান্স উন্নত করলে | `perf: optimize image loading` |
| `test` | টেস্ট যোগ/আপডেট করলে | `test: add unit tests for cart module` |
| `build` | বিল্ড সিস্টেম/ডিপেন্ডেন্সি পরিবর্তন | `build: upgrade webpack to v5` |
| `ci` | CI/CD কনফিগারেশন পরিবর্তন | `ci: add GitHub Actions workflow` |
| `chore` | টুকিটাকি কাজ (src/test-এ পরিবর্তন নেই) | `chore: update .gitignore` |
| `revert` | আগের কোনো কমিট revert করলে | `revert: revert commit abc1234` |

## Scope ব্যবহার

প্রজেক্টের **কোন অংশে** পরিবর্তন হয়েছে তা বোঝায়:

```
feat(api): add user endpoint
fix(ui): correct button alignment
docs(readme): add contribution guide
```

## Subject লেখার নিয়ম

- ✅ **Imperative mood** → `add` লিখুন, `added`/`adds` নয়
- ✅ ছোট হাতের অক্ষরে শুরু করুন
- ✅ ৫০ অক্ষরের মধ্যে রাখুন
- ❌ শেষে ফুলস্টপ (`.`) দেবেন না

```
✅ feat: add payment gateway
❌ feat: Added payment gateway.
```

## Body ও Footer (ঐচ্ছিক)

```
fix(auth): prevent session timeout on active users

Session was expiring even when the user was actively
using the app. Added an activity listener to refresh
the token automatically.

Closes #142
```

- **Body** → কেন এবং কীভাবে পরিবর্তন করা হয়েছে
- **Footer** → issue reference (`Closes #142`) বা breaking change

## Breaking Change

API/আচরণে বড় পরিবর্তন হলে `!` চিহ্ন বা `BREAKING CHANGE:` ফুটার:

```
feat(api)!: change response format to JSON:API spec

BREAKING CHANGE: all endpoints now return data in
JSON:API format. Clients must update their parsers.
```

## ⚡ কুইক চিটশিট

```
feat:     নতুন ফিচার
fix:      বাগ ফিক্স
docs:     ডকুমেন্টেশন
style:    ফরম্যাটিং
refactor: কোড পুনর্গঠন
perf:     পারফরম্যান্স
test:     টেস্ট
build:    বিল্ড/ডিপেন্ডেন্সি
ci:       CI/CD
chore:    টুকিটাকি কাজ
revert:   কমিট revert
```

> 💡 **টিপ:** প্রতিটি কমিট ছোট রাখুন এবং একটি কমিটে একটিমাত্র logical change রাখুন।
> 🔗 আরও: [Conventional Commits](https://www.conventionalcommits.org/) · [Semantic Versioning](https://semver.org/)

---

# Part 4 — Branching ও Merging

## Branch দেখা

| Command | কাজ কী |
|---------|--------|
| `git branch` | সব local branch-এর list (`*` = বর্তমান branch) |
| `git branch -r` | শুধু remote branch (`-r` = **remote**) |
| `git branch -a` | local + remote সব branch (`-a` = **all**) |

## Branch তৈরি ও পরিবর্তন

| Command | কাজ কী |
|---------|--------|
| `git branch feature-login` | নতুন branch তৈরি হবে, কিন্তু switch হবে না |
| `git switch -c feature-login` | নতুন branch তৈরি + সাথে সাথে switch (**আধুনিক, recommended**) |
| `git checkout -b feature-login` | উপরেরটার পুরনো বিকল্প |
| `git switch main` | `main` branch-এ চলে যায় |
| `git switch -` | আগের branch-এ ফিরে যায় (toggle) |
| `git checkout abc123` | নির্দিষ্ট একটা commit-এ যায় (detached HEAD) |

> **📌 `switch` না `checkout`?** নতুন Git-এ (2.23+) branch-এর কাজের জন্য `switch` আর file restore-এর জন্য `restore` — এই দুটোই recommended। `checkout` দুটো কাজই করে বলে confusing। পুরোনো tutorial-এ `checkout` দেখলে ঘাবড়াবেন না, কাজ একই।

## Branch-এর নাম বদলানো ও মোছা

| Command | কাজ কী |
|---------|--------|
| `git branch -m new-name` | বর্তমান branch-এর নাম বদলায় |
| `git branch -m old-name new-name` | নির্দিষ্ট branch-এর নাম বদলায় |
| `git branch -d feature-login` | Merge হয়ে গেলে delete হবে (**safe**) |
| `git branch -D feature-login` | Merge না হলেও জোর করে delete ⚠️ |
| `git push origin --delete feature-login` | Remote branch delete করে |

## Merge করা

```bash
git checkout main           # প্রথমে যে branch-এ merge করবেন সেখানে যান
git merge feature-login     # feature-login-এর সব কাজ main-এ যুক্ত হবে
```

## Squash Merge — সব commit একসাথে

| Command | কাজ কী |
|---------|--------|
| `git merge --squash feature` | Feature branch-এর সব change একসাথে stage করে (commit history আসে না) |
| `git commit -m "message"` | `--squash` নিজে commit করে না — নিজেকেই commit করতে হয় |

## Cherry-pick — নির্দিষ্ট commit বেছে আনা

| Command | কাজ কী |
|---------|--------|
| `git cherry-pick abc123` | অন্য branch থেকে একটা commit current branch-এ আনে |
| `git cherry-pick abc123 def456` | একাধিক commit একসাথে |
| `git cherry-pick abc123^..def456` | একটা range-এর সব commit (`abc123` সহ তার পর থেকে `def456` পর্যন্ত) |
| `git cherry-pick --no-commit abc123` | পরিবর্তন আনবে কিন্তু auto-commit করবে না |
| `git cherry-pick --continue` | Conflict সমাধানের পর চালিয়ে যাওয়া |
| `git cherry-pick --abort` | বাতিল করে আগের অবস্থায় ফেরা |

## পুরনো commit ঠিক করা (Interactive Rebase)

```bash
git rebase -i HEAD~3      # শেষ ৩টি commit দেখাবে; যেটা ঠিক করবেন তার পাশে 'pick' বদলে 'edit' লিখে save
git commit --amend        # commit-এর message/content ঠিক করে save করুন
git rebase --continue     # rebase সম্পূর্ণ — এবার git log-এ সব commit আবার দেখা যাবে
```

> **মনে রাখার সারমর্ম:** branch তৈরি করুন → কাজ করুন → main-এ ফিরে merge করুন → branch delete করুন।

---

# Part 5 — Undo ও উদ্ধার

## Restore ও Reset

| Command | কাজ কী |
|---------|--------|
| `git restore file.txt` | file-এর save-না-করা পরিবর্তন ফেলে দেয় ⚠️ (ফেরানো যায় না) |
| `git restore --staged index.html` | `index.html`-কে unstage করে (কাজ হারায় না) |
| `git reset --soft HEAD~1` | শেষ commit বাতিল, কিন্তু কাজ staged থাকে |
| `git reset --hard HEAD~1` | শেষ commit বাতিল, কাজসহ ⚠️ **বিপজ্জনক** |
| `git reset --hard abc123` | নির্দিষ্ট commit-এ ফিরে যায় |
| `git reflog` | HEAD-এর পুরো ইতিহাস — **হারানো commit এখান থেকে খুঁজে বের করুন** |

> **💡 উদ্ধারের নিয়ম:** ভুল করে `reset --hard` দিয়ে ফেললে ভয় পাবেন না — `git reflog` দিয়ে হারানো commit-এর hash খুঁজে নিয়ে `git reset --hard <hash>` দিলেই ফিরে আসবে।

## Revert — শেয়ার্ড/পাবলিক branch-এ safe undo

`reset` history মুছে দেয়, তাই অন্যরা যে branch ব্যবহার করছে সেখানে **`revert`** ব্যবহার করতে হয় — এটা পুরনো commit বাতিল করে **নতুন একটা commit** বানায়।

| Command | কাজ কী |
|---------|--------|
| `git revert abc123` | নির্দিষ্ট commit-এর পরিবর্তন বাতিল করে নতুন commit যোগ করে |
| `git revert HEAD` | সর্বশেষ commit revert করে |
| `git revert -m 1 <merge-commit>` | Merge commit revert করতে mainline নির্দিষ্ট করে দিতে হয় |
| `git revert --no-commit abc123` | Revert করবে কিন্তু auto-commit করবে না |

## Stash — কাজ সাময়িকভাবে সরিয়ে রাখা

Branch switch করার সময় অসম্পূর্ণ কাজ যাতে হারিয়ে না যায় বা অকারণে commit করতে না হয়, তার জন্য `stash`।

| Command | কাজ কী |
|---------|--------|
| `git stash` | Current changes stash করে (default message সহ) |
| `git stash push -m "message"` | নাম দিয়ে stash করে |
| `git stash list` | কতগুলো stash জমা আছে দেখায় |
| `git stash show` | সর্বশেষ stash-এর changes দেখায় |
| `git stash pop` | সর্বশেষ stash ফিরিয়ে আনে + list থেকে মুছে ফেলে |
| `git stash apply` | ফিরিয়ে আনে, কিন্তু list-এ থেকেই যায় |
| `git stash apply stash@{2}` | নির্দিষ্ট stash ফিরিয়ে আনে (index ধরে) |
| `git stash drop stash@{1}` | নির্দিষ্ট stash মুছে ফেলে |
| `git stash clear` | সব stash মুছে ফেলে |

---

# Part 6 — Remote ও GitHub (push, pull, fetch)

## ⬆️ Push — GitHub-এ আপলোড

| Command | কাজ কী |
|---------|--------|
| `git push -u origin main` | প্রথমবার push — `main` পাঠায় ও tracking সেট করে (`-u` = **set upstream**) |
| `git push` | Tracking সেট থাকলে পরে শুধু এটুকুই যথেষ্ট |
| `git push origin <branch-name>` | নির্দিষ্ট branch push করে |
| `git push --force` | জোর করে push ⚠️ remote-এর history মুছে যেতে পারে |
| `git push origin --delete <branch>` | GitHub থেকে branch delete করে |

## ⬇️ Pull ও Fetch — GitHub থেকে ডাউনলোড

| Command | কাজ কী |
|---------|--------|
| `git fetch` | পরিবর্তন আনে কিন্তু merge করে না (আগে দেখে নিতে চাইলে) |
| `git fetch origin` | নির্দিষ্ট remote-এর সব update নামায় |
| `git pull` | নতুন পরিবর্তন এনে local কোডের সাথে merge করে |
| `git pull origin main` | নির্দিষ্টভাবে `origin`-এর `main` থেকে pull করে |
| `git pull --rebase` | Merge commit না বানিয়ে নিজের commit গুলো নতুন কোডের উপরে বসায় |

> **fetch vs pull:** `pull` = `fetch` + `merge`। আগে দেখে নিয়ে তারপর সিদ্ধান্ত নিতে চাইলে `fetch` নিরাপদ।

## Remote Branch নিয়ে কাজ

| Command | কাজ কী |
|---------|--------|
| `git push -u origin feature-login` | নতুন branch remote-এ push করে (tracking সহ) |
| `git checkout -b local-name origin/remote-name` | Remote branch থেকে local branch তৈরি করে |

---

# Part 7 — Collaboration (Fork, Pull Request, Conflict)

কোনো repository-তে collaborate করার **২টি উপায়** আছে।

## পদ্ধতি ১: সরাসরি Access দেওয়া

Repository-র মালিক নিচের ধাপে collaborator যুক্ত করবেন:

> **Settings → Collaborators and Teams → Add People**

GitHub username/email দিয়ে add করলেই সরাসরি access পাওয়া যাবে।

## পদ্ধতি ২: Fork করে কাজ করা

### ধাপ ১ — Fork ও Clone

Original repo-তে গিয়ে **Fork** বাটনে ক্লিক করে repo-টি নিজের account-এ fork করুন, তারপর:

```bash
git clone <fork-url>
```

### ধাপ ২ — Remote সেটআপ

```bash
git remote -v                                   # বর্তমান remote দেখুন
git remote rename origin bongodev               # origin-কে original owner-এর নামে rename
git remote add arifucoder <amar-fork-kora-url>  # নিজের fork নতুন remote হিসেবে add
```

> **📌 মনে রাখবেন:**
> - `bongodev` = original repo (**upstream**)
> - `arifucoder` = আমার fork করা repo

### ধাপ ৩ — main/master আপডেট করা

⚠️ **আমরা কখনোই master/main branch-এ কাজ করব না!** আলাদা branch বানিয়ে সেখানে কাজ করব।

```bash
git fetch bongodev                  # original repo থেকে সব update আনুন
git reset --hard bongodev/master    # ⚠️ local master-এ যা আছে সব ফেলে দিয়ে original দিয়ে আপডেট করবে
```

### ধাপ ৪ — Branch তৈরি, কাজ ও Commit

```bash
git switch -c something-branch             # branch create + checkout একসাথে
git commit -m "[arifucoder] git message"   # commit message অবশ্যই এই ফরম্যাটে
```

### ধাপ ৫ — নিজের Repo-তে Push

```bash
git push arifucoder something-branch
```

### ধাপ ৬ — Pull Request তৈরি

Original repo-তে গিয়ে:

1. **Pull Request** → **New Pull Request**
2. **Compare across forks**-এ ক্লিক করুন
3. সেট করুন — **Base:** original repo + master/main | **Compare:** আমার repo + আমার branch
4. **Create Pull Request**-এ ক্লিক
5. Title + Description দিন:
   - **Title:** `[amar-github-user-name] message`
   - **Description:** বিস্তারিত message
6. Finally **Create Pull Request**

✅ এবার **Pull Requests** ট্যাবে গেলে নিজের PR দেখতে পাবেন।

### ধাপ ৭ — Review

- PR আসলে **team থেকে reviewer assign** করা যায়।
- যেখানে feed থাকবে সেখানে **`+` sign**-এ ক্লিক করে comment দিতে হবে।

## Merge Conflict Resolve করার পদ্ধতি

**১.** VS Code-এর terminal খুলুন

**২.** Original repo থেকে fetch করুন:
```bash
git fetch bongodev
```

**৩.** Rebase করুন:
```bash
git rebase bongodev/master
```

**৪.** এবার VS Code-এ **merge conflict** দেখাবে।

**৫.** Git icon → **Merge Changes**-এর under-এ file গুলো এক এক করে ক্লিক করে **প্রয়োজন মতো আপডেট** করুন।

**৬.** সব ঠিক করার পর:
```bash
git add .
git rebase --continue    # vim/nano-তে commit message edit করতে বলবে — edit করে save করুন
```

**৭.** **অবশ্যই** log চেক করুন:
```bash
git log
```

**৮.** নিজের repo-তে force push করুন:
```bash
git push arifucoder branch-name -f
```

✅ এবার pull request **Ready to Merge** হয়ে যাবে! 🎉

---

# Part 8 — Quick Reference

## দৈনন্দিন কাজ

| কাজ | Command |
|------|---------|
| অবস্থা দেখা | `git status -sb` |
| সব stage করা | `git add .` |
| অংশে অংশে stage | `git add -p` |
| Commit করা | `git commit -m "message"` |
| শেষ commit ঠিক করা | `git commit --amend` |
| পার্থক্য দেখা | `git diff --staged` |
| History দেখা | `git log --oneline --graph --all` |
| Push করা | `git push` |
| Pull করা | `git pull` |

## Config

| কাজ | Command |
|------|---------|
| Global নাম সেট | `git config --global user.name "name"` |
| Global ইমেইল সেট | `git config --global user.email "email"` |
| এক folder-এ local নাম | `git config user.name "name"` |
| এক folder-এ local ইমেইল | `git config user.email "email"` |
| Config কোথা থেকে আসছে | `git config --show-origin user.email` |
| Token সহ remote | `git remote set-url origin https://<TOKEN>@github.com/user/repo.git` |

## Branch

| কাজ | Command |
|------|---------|
| Branch তৈরি + switch | `git switch -c branch-name` |
| Branch-এ যাওয়া | `git switch main` |
| Merge করা | `git merge feature-login` |
| Branch delete (safe) | `git branch -d feature-login` |
| Cherry-pick | `git cherry-pick abc123` |

## উদ্ধার

| কাজ | Command |
|------|---------|
| Unstage করা | `git restore --staged file.txt` |
| Commit বাতিল (safe) | `git revert HEAD` |
| Commit বাতিল (local) | `git reset --hard HEAD~1` |
| হারানো commit খোঁজা | `git reflog` |
| কাজ সরিয়ে রাখা | `git stash` / `git stash pop` |

## Fork Workflow

| কাজ | Command |
|------|---------|
| Remote rename | `git remote rename origin bongodev` |
| Remote add | `git remote add arifucoder <fork-url>` |
| Fetch করা | `git fetch bongodev` |
| Local master আপডেট | `git reset --hard bongodev/master` |
| Commit | `git commit -m "[username] message"` |
| Push | `git push arifucoder branch-name` |
| Rebase | `git rebase bongodev/master` |
| Force push | `git push arifucoder branch-name -f` |