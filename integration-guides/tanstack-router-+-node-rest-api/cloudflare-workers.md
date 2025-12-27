---
icon: cloudflare
---

# Cloudflare Workers

TODO an example with this style

```
export default {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);

    if (request.method === "GET" && url.pathname === "/hello") {
      return new Response("hello", { headers: { "content-type": "text/plain" } });
    }

    return new Response("not found", { status: 404 });
  }
};
```
