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

## 20.5 Conclusions

### 이번에 할 것

- Visitor를 JSON에서 SVG로 꾸미기

### 인상적인 내용

- 이미지 요청 → 데이터 처리 → Visitor결과를 svg로 만들고 응답
- 이메일을 읽었는지 확인하는 것도 이렇게 확인한다.

### 코드

- workers-visitors/src/home.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Visit Counter</title>
  </head>
  <body>
    <h1>Visit Counter</h1>
    <h3>How to use?</h3>
    <ul>
      <li>
        All you have to do is call (GET) this URL:
        <code
          >https://workers-visitors.myungjinki91.workers.dev/visit?page=$URL
        </code>
      </li>
      <li>Replace $URL for your website URL.</li>
    </ul>
    <h4>Live Demo:</h4>
    <img src="/visit?page=https://workers-visitors.myungjinki91.workers.dev/" />
  </body>
</html>
```

- workers-visitors/src/index.ts

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
import { makeBadge } from "./utils";

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
  return new Response(makeBadge(value), {
    headers: {
      "Content-Type": "image/svg+xml;charset=utf-8",
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

- workers-visitors/src/utils.ts

```tsx
export const makeBadge = (hits: number) => {
  const formatted = new Intl.NumberFormat("kr-KO").format(hits);
  const width = `Views:  ${formatted}`.length * 6.5;
  const svg = `
      <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="${
        width + 10
      }" height="20" role="img" aria-label="Views ${formatted}">
        <title>Views: ${formatted}</title>
        <g shape-rendering="crispEdges">
        <rect x="0" width="${width + 3}" height="20" fill="#000"/>
        </g>
        <g fill="#fff" text-anchor="middle" font-family="Verdana,Geneva,DejaVu Sans,sans-serif" text-rendering="geometricPrecision" font-size="110">
          <text x="${
            width * 5
          }" y="140" transform="scale(.1)" fill="#fff" textLength="${
    width * 9
  }">Views:  ${formatted}</text>
        </g>
      </svg>
      `;
  return svg;
};
```

## 20.7 Our First Durable Object

### 이번에 할 것

- Durable Object 사용해보기

### 인상적인 내용

- durable object는 유저가 직접 호출할 수 없음!

```tsx
export default {
  async fetch(
    request: Request,
    env: Env,
    ctx: ExecutionContext
  ): Promise<Response> {
    const id = env.COUNTER.idFromName("counter");
    const durableObject = env.COUNTER.get(id);
    const response = await durableObject.fetch(request);
    return response;
  },
};
```

고유한 id 생성한 후,

```tsx
const id = env.COUNTER.idFromName("counter");
```

Cloudflare Workers가 인스턴스를 생성하고,

```tsx
const durableObject = env.COUNTER.get(id);
```

인스턴스의 fetch 메소드 호출

```tsx
const response = await durableObject.fetch(request);
```

### 코드

- workers-chat/wrangler.toml

```tsx
#:schema node_modules/wrangler/config-schema.json
name = "workers-visitors"
main = "src/index.ts"
compatibility_date = "2024-07-29"
compatibility_flags = ["nodejs_compat"]

kv_namespaces = [
  { binding = "DB", id = "3d04e05e4b864148b2950aad45851473", preview_id = "a7573ca76396474c811223313b0ad5a6" }
]

[[migrations]]
tag = "v1" # Should be unique for each entry
new_classes = ["CounterObject"]

[durable_objects]
bindings = [
    {name = "COUNTER", class_name = "CounterObject"}
]
```

- workers-chat/src/index.ts

```tsx
export interface Env {
  DB: KVNamespace;
  COUNTER: DurableObjectNamespace;
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

export class CounterObject {
  counter: number;
  constructor() {
    this.counter = 0;
  }
  async fetch(request: Request) {
    const { pathname } = new URL(request.url);
    switch (pathname) {
      case "/":
        return new Response(this.counter);
      case "/+":
        this.counter++;
        return new Response(this.counter);
      case "/-":
        this.counter--;
        return new Response(this.counter);
      default:
        return handleNotFound();
    }
  }
}

export default {
  async fetch(
    request: Request,
    env: Env,
    ctx: ExecutionContext
  ): Promise<Response> {
    const id = env.COUNTER.idFromName("counter");
    const durableObject = env.COUNTER.get(id);
    const response = await durableObject.fetch(request);
    return response;
  },
};
```

## 20.8 Serverless WebSockets

### 이번에 할 것

- Websocket 기본 연결

### 인상적인 내용

window.location.host

어렵다!!!!

### 코드

- workers-chat/wrangler.toml

```tsx
#:schema node_modules/wrangler/config-schema.json
name = "workers-visitors"
main = "src/index.ts"
compatibility_date = "2024-07-29"
compatibility_flags = ["nodejs_compat"]

kv_namespaces = [
  { binding = "DB", id = "3d04e05e4b864148b2950aad45851473", preview_id = "a7573ca76396474c811223313b0ad5a6" }
]

[[migrations]]
tag = "v1" # Should be unique for each entry
new_classes = ["CounterObject"]

[durable_objects]
bindings = [
    {name = "CHAT", class_name = "ChatRoom"}
]
```

- workers-chat/src/home.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Visit Counter</title>
  </head>
  <body>
    <h1>Visit Counter</h1>
    <button>Connect</button>
    <script>
      const button = document.querySelector("button");
      button.addEventListener("click", () => {
        const socket = new WebSocket(`ws://${window.location.host}/connect`);

        socket.addEventListener("open", () => {
          console.log("connected");
        });

        socket.addEventListener("message", (event) => {
          console.log(event.data);
        });
      });
    </script>
  </body>
</html>
```

- workers-chat/src/index.ts

```tsx
export interface Env {
  DB: KVNamespace;
  CHAT: DurableObjectNamespace;
}

// @ts-ignore
import home from "./home.html";

export class ChatRoom {
  state: DurableObjectState;
  constructor(state: DurableObjectState, env: Env) {
    this.state = state;
  }
  handleHome() {
    return new Response(home, {
      headers: {
        "Content-Type": "text/html;chartset=utf-8",
      },
    });
  }
  handleNotFound() {
    return new Response(null, {
      status: 404,
    });
  }
  handleConnect(request: Request) {
    const pairs = new WebSocketPair();
    this.handleWebSocket(pairs[1]);
    return new Response(null, { status: 101, webSocket: pairs[0] });
  }
  handleWebSocket(webSocket: WebSocket) {
    webSocket.accept();
    webSocket.send(JSON.stringify({ message: "hello from backend!" }));
  }
  async fetch(request: Request) {
    const { pathname } = new URL(request.url);
    switch (pathname) {
      case "/":
        return this.handleHome();
      case "/connect":
        return this.handleConnect(request);
      default:
        return this.handleNotFound();
    }
  }
}
export default {
  async fetch(
    request: Request,
    env: Env,
    ctx: ExecutionContext
  ): Promise<Response> {
    const id = env.CHAT.idFromName("CHAT");
    const durableObject = env.CHAT.get(id);
    const response = await durableObject.fetch(request);
    return response;
  },
};
```

### 팁

https://developers.cloudflare.com/durable-objects/api/websockets/

https://websockets.spec.whatwg.org/
