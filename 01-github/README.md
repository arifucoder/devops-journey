# 🌿 Git ও GitHub Cheatsheet

দৈনন্দিন কাজে সবচেয়ে বেশি লাগে এমন Git ও GitHub command-এর সম্পূর্ণ রেফারেন্স — এক জায়গায়, বাংলায়।

## 📚 Table of Contents

- [Class 1 — Setup ও প্রথম Repository](#class-1--setup-ও-প্রথম-repository)
- [Class 2 — দৈনন্দিন Workflow (add, commit, diff, log)](#class-2--দৈনন্দিন-workflow-add-commit-diff-log)
- [Class 3 — Branching ও Merging](#class-3--branching-ও-merging)
- [Class 4 — Undo ও উদ্ধার (restore, reset, revert, stash)](#class-4--undo-ও-উদ্ধার-restore-reset-revert-stash)
- [Class 5 — Remote ও GitHub (push, pull, fetch)](#class-5--remote-ও-github-push-pull-fetch)
- [Class 6 — Collaboration (Fork, Pull Request, Conflict)](#class-6--collaboration-fork-pull-request-conflict)
- [Quick Reference](#-quick-reference)

---

# Class 1 — Setup ও প্রথম Repository

## Repository শুরু করা

| Command | Description |
|---------|-------------|
| `git init` | নতুন local Git repository তৈরি করে |
| `git clone <repo-url>` | GitHub থেকে পুরো repository local-এ copy করে |
| `git branch -M main` | বর্তমান branch-এর নাম `main` করে দেয় <br>`-M` = **Move (force rename)** |

## Remote — GitHub-এর সাথে সংযোগ

`remote` = আপনার local repo কোন online repo-র সাথে যুক্ত, তার নাম-ঠিকানা।

| Command | Description |
|---------|-------------|
| `git remote add origin <repo-url>` | Local repo-র সাথে GitHub repo যুক্ত করে (`origin` নামে) |
| `git remote -v` | কোন কোন remote যুক্ত আছে, URL সহ দেখায় <br>`-v` = **verbose** |
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

# Class 2 — দৈনন্দিন Workflow (add, commit, diff, log)

## অবস্থা দেখা

| Command | Description |
|---------|-------------|
| `git status` | কোন file add/commit হয়েছে বা হয়নি তা দেখায় |
| `git status -sb` | সংক্ষিপ্ত (short) আকারে + branch-এর নাম সহ |

## Stage করা (add)

| Command | Description |
|---------|-------------|
| `git add file.txt` | নির্দিষ্ট একটা file staging area-তে যোগ করে |
| `git add .` | সব পরিবর্তিত file একসাথে stage করে |
| `git restore --staged file.txt` | Stage করা file আবার unstage করে |

## Commit করা

| Command | Description |
|---------|-------------|
| `git commit -m "message"` | Staged file গুলো message সহ commit করে |
| `git commit -am "message"` | Tracked file একসাথে add + commit করে (নতুন file-এ কাজ করবে না) |
| `git commit --amend` | শেষ commit-এর message বা content ঠিক করে |

## পার্থক্য দেখা (diff)

| Command | Description |
|---------|-------------|
| `git diff` | Unstaged পরিবর্তন দেখায় |
| `git diff --staged` | Staged পরিবর্তন দেখায় |
| `git diff --stat` | শুধু সারমর্ম — কোন file-এ কতটুকু বদলেছে |

## ইতিহাস দেখা (log)

| Command | Description |
|---------|-------------|
| `git log` | পূর্ণ commit ইতিহাস |
| `git log --oneline` | প্রতি commit এক লাইনে |
| `git log --oneline --graph --all --decorate` | সব branch-এর history গ্রাফ আকারে — **সেরা combo** |

## একটা commit-এর বিস্তারিত (show)

| Command | Description |
|---------|-------------|
| `git show abc123` | নির্দিষ্ট commit-এর পুরো info + diff দেখায় |
| `git show HEAD` | সর্বশেষ commit দেখায় |
| `git show HEAD~2` | ২টা commit আগেরটা দেখায় |
| `git show abc123 --stat` | শুধু কোন কোন file বদলেছে তার সারমর্ম |
| `git show abc123:file.txt` | ওই commit-এ `file.txt`-এর content দেখায় |

---

# Class 3 — Branching ও Merging

## Branch দেখা

| Command | Description |
|---------|-------------|
| `git branch` | সব local branch-এর list (`*` = বর্তমান branch) |
| `git branch -r` | শুধু remote branch দেখায় <br>`-r` = **remote** |
| `git branch -a` | Local + remote সব branch <br>`-a` = **all** |

## Branch তৈরি ও পরিবর্তন

| Command | Description |
|---------|-------------|
| `git branch feature-login` | নতুন branch তৈরি হবে, কিন্তু switch হবে না |
| `git switch -c feature-login` | নতুন branch তৈরি + সাথে সাথে switch (**আধুনিক, recommended**) |
| `git checkout -b feature-login` | উপরেরটার পুরনো বিকল্প |
| `git switch main` | `main` branch-এ চলে যায় |
| `git switch -` | আগের branch-এ ফিরে যায় (toggle) |

> **📌 `switch` না `checkout`?** নতুন Git-এ branch-এর কাজের জন্য `switch` আর file restore-এর জন্য `restore` — এই দুটোই recommended। `checkout` দুটো কাজই করে বলে confusing।

## Branch-এর নাম বদলানো ও মোছা

| Command | Description |
|---------|-------------|
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

| Command | Description |
|---------|-------------|
| `git merge --squash feature` | Feature branch-এর সব change একসাথে stage করে (commit history আসে না) |
| `git commit -m "message"` | `--squash` নিজে commit করে না — নিজেকেই commit করতে হয় |

## Cherry-pick — নির্দিষ্ট commit বেছে আনা

| Command | Description |
|---------|-------------|
| `git cherry-pick abc123` | অন্য branch থেকে একটা commit current branch-এ আনে |
| `git cherry-pick abc123 def456` | একাধিক commit একসাথে |
| `git cherry-pick abc123^..def456` | একটা range-এর সব commit (`abc123` সহ তার পর থেকে `def456` পর্যন্ত) |
| `git cherry-pick --no-commit abc123` | পরিবর্তন আনবে কিন্তু auto-commit করবে না |
| `git cherry-pick --continue` | Conflict সমাধানের পর চালিয়ে যাওয়া |
| `git cherry-pick --abort` | বাতিল করে আগের অবস্থায় ফেরা |

## পুরনো commit ঠিক করা (Interactive Rebase)

```bash
git rebase -i HEAD~3      # শেষ ৩টি commit দেখাবে; যেটা ঠিক করবেন তার পাশে 'pick' বদলে 'edit' লিখে save করুন
git commit --amend        # commit-এর message বা content ঠিক করে save করুন
git rebase --continue     # rebase সম্পূর্ণ হবে — এবার git log-এ সব commit আবার দেখা যাবে
```

> **মনে রাখার সারমর্ম:** branch তৈরি করুন → কাজ করুন → main-এ ফিরে merge করুন → branch delete করুন।

---

# Class 4 — Undo ও উদ্ধার (restore, reset, revert, stash)

## Restore ও Reset

| Command | Description |
|---------|-------------|
| `git restore --staged index.html` | `index.html`-কে unstage করে (কাজ হারায় না) |
| `git reset --hard HEAD~1` | শেষ commit বাতিল, কাজসহ ⚠️ **বিপজ্জনক** |
| `git reset --hard abc123` | নির্দিষ্ট commit-এ ফিরে যায় |
| `git reflog` | HEAD-এর পুরো ইতিহাস — **হারানো commit এখান থেকে খুঁজে বের করুন** |

> **💡 উদ্ধারের নিয়ম:** ভুল করে `reset --hard` দিয়ে ফেললে ভয় পাবেন না — `git reflog` দিয়ে হারানো commit-এর hash খুঁজে নিয়ে `git reset --hard <hash>` দিলেই ফিরে আসবে।

## Revert — শেয়ার্ড/পাবলিক branch-এ safe undo

`reset` history মুছে দেয়, তাই অন্যরা যে branch ব্যবহার করছে সেখানে **`revert`** ব্যবহার করতে হয় — এটা পুরনো commit বাতিল করে **নতুন একটা commit** বানায়।

| Command | Description |
|---------|-------------|
| `git revert abc123` | নির্দিষ্ট commit-এর পরিবর্তন বাতিল করে নতুন commit যোগ করে |
| `git revert HEAD` | সর্বশেষ commit revert করে |
| `git revert -m 1 <merge-commit>` | Merge commit revert করতে mainline নির্দিষ্ট করে দিতে হয় |
| `git revert --no-commit abc123` | Revert করবে কিন্তু auto-commit করবে না |

## Stash — কাজ সাময়িকভাবে সরিয়ে রাখা

Branch switch করার সময় অসম্পূর্ণ কাজ যাতে হারিয়ে না যায় বা অকারণে commit করতে না হয়, তার জন্য `stash`।

| Command | Description |
|---------|-------------|
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

# Class 5 — Remote ও GitHub (push, pull, fetch)

## ⬆️ Push — GitHub-এ আপলোড

| Command | Description |
|---------|-------------|
| `git push -u origin main` | প্রথমবার push — `main` পাঠায় এবং tracking সেট করে <br>`-u` = **set upstream** |
| `git push` | Tracking সেট থাকলে পরে শুধু এটুকুই যথেষ্ট |
| `git push origin <branch-name>` | নির্দিষ্ট branch push করে |
| `git push --force` | জোর করে push ⚠️ remote-এর history মুছে যেতে পারে |
| `git push origin --delete <branch>` | GitHub থেকে branch delete করে |

## ⬇️ Pull ও Fetch — GitHub থেকে ডাউনলোড

| Command | Description |
|---------|-------------|
| `git fetch` | পরিবর্তন আনে কিন্তু merge করে না (আগে দেখে নিতে চাইলে) |
| `git fetch origin` | নির্দিষ্ট remote-এর সব update নামায় |
| `git pull` | নতুন পরিবর্তন এনে local কোডের সাথে merge করে |
| `git pull origin main` | নির্দিষ্টভাবে `origin`-এর `main` থেকে pull করে |
| `git pull --rebase` | Merge commit না বানিয়ে নিজের commit গুলো নতুন কোডের উপরে বসায় |

> **fetch vs pull:** `pull` = `fetch` + `merge`। আগে দেখে নিয়ে তারপর সিদ্ধান্ত নিতে চাইলে `fetch` নিরাপদ।

## Remote Branch নিয়ে কাজ

| Command | Description |
|---------|-------------|
| `git push -u origin feature-login` | নতুন branch remote-এ push করে (tracking সহ) |
| `git checkout -b local-name origin/remote-name` | Remote branch থেকে local branch তৈরি করে |

---

# Class 6 — Collaboration (Fork, Pull Request, Conflict)

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
git switch -c something-branch          # branch create + checkout একসাথে
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

## 🚀 Quick Reference

### দৈনন্দিন কাজ

| কাজ | Command |
|------|---------|
| অবস্থা দেখা | `git status -sb` |
| সব stage করা | `git add .` |
| Commit করা | `git commit -m "message"` |
| History দেখা | `git log --oneline --graph --all` |
| Push করা | `git push` |
| Pull করা | `git pull` |

### Branch

| কাজ | Command |
|------|---------|
| Branch তৈরি + switch | `git switch -c branch-name` |
| Branch-এ যাওয়া | `git switch main` |
| Merge করা | `git merge feature-login` |
| Branch delete | `git branch -d feature-login` |

### উদ্ধার

| কাজ | Command |
|------|---------|
| Unstage করা | `git restore --staged file.txt` |
| শেষ commit ঠিক করা | `git commit --amend` |
| Commit বাতিল (safe) | `git revert HEAD` |
| Commit বাতিল (local) | `git reset --hard HEAD~1` |
| হারানো commit খোঁজা | `git reflog` |
| কাজ সরিয়ে রাখা | `git stash` / `git stash pop` |

### Fork Workflow

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