# PM2 দিয়ে Node.js Server Manage করা

Node.js server সবসময় background-এ চালু রাখার জন্য, আর হঠাৎ বন্ধ হয়ে গেলে auto-restart করার জন্য process manager লাগে। এরকম অনেক process manager আছে, তার মধ্যে **PM2** সবচেয়ে popular।

## Install করা

VPS-এ globally install করতে:

```bash
npm i -g pm2
```

Version check করে দেখা install হয়েছে কিনা:

```bash
pm2 -v
```

## App চালু করা

Backend project folder-এর ভেতরে গিয়ে এই command চালাতে হবে:

```bash
pm2 start app.js --name server_name
```

Status দেখার command:

```bash
pm2 status
```

Status-এ **online** দেখালে বুঝতে হবে process ঠিকভাবে চলছে।

## Delete, Stop, Restart, Reload

Process delete করার জন্য (id বা name যেকোনোটা দেওয়া যাবে):

```bash
pm2 delete id/name
```

Server stop করার command:

```bash
pm2 stop id/name
```

Server restart করার command:

```bash
pm2 restart id/name
```

**restart** না করে সাধারণত **reload** করা উচিত। কারণ restart করলে সামান্য কিছু সময়ের জন্য হলেও server বন্ধ থাকে। কিন্তু reload করলে server zero downtime নিয়ে চালু থাকে।

Reload করার command:

```bash
pm2 reload id/name
```

## Ecosystem File

একসাথে একাধিক app বা config manage করার জন্য একটা **ecosystem file** বানাতে হয়। এটা folder না, একটা **file** — application-এর root folder-এ `ecosystem.config.js` নামে তৈরি করতে হবে:

```bash
vim ecosystem.config.js
```

```javascript
module.exports = {
  apps: [
    {
      name: "app",
      cwd: "/home/ubuntu/applications/application-name",
      script: "./app.js",
      env: {
        NODE_ENV: "development",
      },
      env_production: {
        NODE_ENV: "production",
      },
    },
    {
      name: "worker",
      script: "worker.js",
    },
  ],
};
```

এখানে `apps` array-এর ভেতরে প্রতিটা `{}` মানে একটা আলাদা application।

- **name** — application-এর নাম
- **cwd** — application-এর root folder-এর path
- **script** — যে file থেকে app চালু হবে
- **env** — environment variable গুলো, দরকার হলে add করতে হবে

এই file দিয়ে app চালু করার command:

```bash
pm2 start ecosystem.config.js
```

## Reboot-এর পরও App Auto-Start করানো

সার্ভার reboot নিলে PM2 নিজে থেকে আবার চালু হয় না, app-ও চালু হয় না। এটা ঠিক করতে হলে আগে **pm2 startup** চালাতে হবে:

```bash
pm2 startup
```

এই command চালালে PM2 একটা command দেখাবে (সাধারণত `sudo` সহ) — সেই command-টা কপি করে আলাদাভাবে চালাতে হবে।

তারপর বর্তমানে চলমান app-গুলোর list save করতে হবে:

```bash
pm2 save
```

এরপর থেকে সার্ভার reboot নিলেও PM2 এবং তোমার app গুলো নিজে থেকেই আবার চালু হয়ে যাবে।