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

## 20.3 Workers KV

### 이번에 할 것

- KV생성
- KV와 Worker 연결

### 인상적인 내용

- Workers KV
- KV database
- Serverless DB

### 코드

```tsx
wrangler kv:namespace create "view_counter"

▲ [WARNING] The `wrangler kv:namespace` command is deprecated and will be removed in a future major version. Please use `wrangler kv namespace` instead which behaves the same.

 ⛅️ wrangler 3.68.0
-------------------

🌀 Creating namespace with title "workers-visitors-view_counter"
✨ Success!
Add the following to your configuration file in your kv_namespaces array:
[[kv_namespaces]]
binding = "view_counter"
id = "3d04e05e4b864148b2950aad45851473"
```

- workers-visitors/wrangler.toml

```tsx
#:schema node_modules/wrangler/config-schema.json
name = "workers-visitors"
main = "src/index.ts"
compatibility_date = "2024-07-29"
compatibility_flags = ["nodejs_compat"]

kv_namespaces = [
  { binding = "view_counter", id = "3d04e05e4b864148b2950aad45851473" }
]
```

- workers-visitors/src/index.ts

```tsx
export interface Env {
  // Example binding to KV. Learn more at https://developers.cloudflare.com/workers/runtime-apis/kv/
  DB: KVNamespace;
}

// @ts-ignore
import home from "./home.html";

export default {
  async fetch(request, env, ctx): Promise<Response> {
    const url = new URL(request.url);
    if (url.pathname === "/") {
      await env.DB.put("hello", "how are you?");
      return new Response(home, {
        headers: {
          "Content-Type": "text/html;chartset=utf-8",
        },
      });
    }
    return new Response(null, {
      status: 404,
    });
  },
} satisfies ExportedHandler<Env>;
```

### 팁

wrangler kv:namespace create "view_counter"

wrangler kv:namespace create --preview "view_counter"

VSC 'better toml' install

---

로컬에서 사용시

npm run start 사용시 KV-Cloudflare preview 데이터 저장 안됨

wrangler dev --remote 로 시작해야 KV-Cloudflare preview 에 데이터 저장이 됨

## 20.4 Visit Counter

### 이번에 할 것

- Visit Counter 만들기

### 인상적인 내용

- TypeScript: URLSearchParams
- JSON도 저장가능!
- eventually consistent: DB는 Cloudflare network에 존재해서 너무 빨리 업데이트하면 반영이 바로 안될 수도 있습니다. 그리고 결국 마지막 데이터가 저장됩니다.
- KV는 미쳤습니다.

### 코드

```tsx
/**
 * Welcome to Cloudflare Workers! This is your first worker.
 *
 * - Run `npm run dev` in your terminal to start a development server
 * - Open a browser tab at http://localhost:8787/ to see your worker in action
 * - Run `npm run deploy` to publish your worker
 *
 * Bind resources to your worker in `wrangler.toml`. After adding bindings, a type definition for the
 * `Env` object can be regenerated with `npm run cf-typegen`.
 *
 * Learn more at https://developers.cloudflare.com/workers/
 */

export interface Env {
  // Example binding to KV. Learn more at https://developers.cloudflare.com/workers/runtime-apis/kv/
  DB: KVNamespace;
}

// @ts-ignore
import home from "./home.html";

function handleHome() {
  return new Response(home, {
    headers: {
      "Content-Type": "text/html;chartset=utf-8",
    },
  });
}

function handleNotFound() {
  return new Response(null, {
    status: 404,
  });
}

function handleBadRequest() {
  return new Response(null, {
    status: 400,
  });
}

async function handleVisit(searchParams: URLSearchParams, env: Env) {
  const page = searchParams.get("page");
  if (!page) {
    return handleBadRequest();
  }
  const kvPage = await env.DB.get(page);
  let value = 1;
  if (!kvPage) {
    await env.DB.put(page, value + "");
  } else {
    value = parseInt(kvPage) + 1;
    await env.DB.put(page, value + "");
  }
  return new Response(JSON.stringify({ visits: value }), {
    headers: {
      "Content-Type": "application/json",
    },
  });
}

export default {
  async fetch(
    request: Request,
    env: Env,
    ctx: ExecutionContext
  ): Promise<Response> {
    const { pathname, searchParams } = new URL(request.url);
    switch (pathname) {
      case "/":
        return handleHome();
      case "/visit":
        return handleVisit(searchParams, env);
      default:
        return handleNotFound();
    }
  },
};
```
