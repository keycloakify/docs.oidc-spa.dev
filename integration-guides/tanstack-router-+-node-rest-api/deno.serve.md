---
icon: dinosaur
---

# Deno.serve

TODO: Note import { ... } from "npm:oidc-spa@latest/server" (the package is only published on npm)<br>

We want an example in this style

```
Deno.serve((request: Request) => {
  const url = new URL(request.url);

  if (request.method === "GET" && url.pathname === "/hello") {
    return new Response("hello");
  }

  return new Response("not found", { status: 404 });
});
```
