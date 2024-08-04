# 20 BONUS: CLOUDFLARE WORKERS

## 20.0 Welcome

### 인상적인 내용

- Cloudflare worker는 어마어마한 serverless platform
- KV도 배울것임
- Cloudflare는 serverless DB도 보유
- Cloudflare는 대부분의 제품을 serverless로 만듬
- durable Object는 실시간 serverless 채팅을 만들 때 사용
- visit counter도 만들것임

## 20.1 Our First Worker

### 이번에 할 것

- Cloudflare workers 튜토리얼

### 인상적인 내용

아래 두 명령으로 바로 배포 가능!

```bash
wrangler init workers-visitors
npm run deploy
```

### 코드

```bash
wrangler init workers-visitors

 ⛅️ wrangler 3.68.0
-------------------

Using npm as package manager.
▲ [WARNING] The `init` command is no longer supported. Please use `npm create cloudflare\@2.5.0 -- workers-visitors` instead.

  The `init` command will be removed in a future version.

Running `npm create cloudflare\@2.5.0 -- workers-visitors`...

> npx
> create-cloudflare workers-visitors

> npx
> create-cloudflare workers-visitors

using create-cloudflare version 2.23.0

╭ Create an application with Cloudflare Step 1 of 3
│
├ In which directory do you want to create your application?
│ dir ./workers-visitors
│
├ What would you like to start with?
│ category Hello World example
│
├ Which template would you like to use?
│ type Hello World Worker
│
├ Which language do you want to use?
│ lang TypeScript
│
├ Copying template files
│ files copied to project directory
│
├ Updating name in `package.json`
│ updated `package.json`
│
├ Installing dependencies
│ installed via `npm install`
│
╰ Application created

╭ Configuring your application for Cloudflare Step 2 of 3
│
├ Installing @cloudflare/workers-types
│ installed via npm
│
├ Adding latest types to `tsconfig.json`
│ added @cloudflare/workers-types/2023-07-01
│
├ Retrieving current workerd compatibility date
│ compatibility date 2024-07-29
│
╰ Application configured

╭ Deploy with Cloudflare Step 3 of 3
│
├ Do you want to deploy your application?
│ yes deploy via `npm run deploy`
│
├ Logging into Cloudflare checking authentication status
│ logged in
│
├ Selecting Cloudflare account retrieving accounts
│ account Myungjinki91@gmail.com's Account
│
├ Deploying your application
│ deployed via `npm run deploy`
│
├  SUCCESS  View your deployed application at https://workers-visitors.myungjinki91.workers.dev
│
│ Navigate to the new directory cd workers-visitors
│ Run the development server npm run start
│ Deploy your application npm run deploy
│ Read the documentation https://developers.cloudflare.com/workers
│ Stuck? Join us at https://discord.cloudflare.com
│
├ Waiting for DNS to propagate
│ DNS propagation complete.
│
├ Waiting for deployment to become available
│ deployment is ready at: https://workers-visitors.myungjinki91.workers.dev
│
├ Opening browser
│
╰ See you again soon!

```

```bash
npm run deploy
```
