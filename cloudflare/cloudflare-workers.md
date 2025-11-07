# Cloudflare Workers

Unlike Cloudflare Pages, for Cloudflare Workers project, you can edit settings via:

- Wrangler configuration file.
- Cloudflare dashboard.

[Cloudflare recommended the config file is **the SSOT**](https://developers.cloudflare.com/workers/wrangler/configuration/#source-of-truth). Avoid making changes to the project via Dashboard since Wrangler will override them in the next deployment. If you must:

- Generate a TOML/JSONC snippet and copy into the config file.
- Disable overriding behavior by setting `"workers_dev": true` and `"keep_vars": true`.

```jsonc
/**
 * Cloudflare Workers Wrangler configuration file references:
 * https://developers.cloudflare.com/workers/wrangler/configuration/
 */
{
  "name": "my-worker",
  "main": "src/index.js",
  "compatibility_date": "2022-07-12",
  // Prevent overriding env var set in Dashboard
  "keep_vars": true,
  // Prevent overriding routes set in Dashboard
  "workers_dev": false,
  "route": {
    "pattern": "example.org/*",
    "zone_name": "example.org"
  },
  "kv_namespaces": [
    {
      "binding": "<MY_NAMESPACE>",
      "id": "<KV_ID>"
    }
  ],
  "env": {
    "staging": {
      "name": "my-worker-staging",
      "route": {
        "pattern": "staging.example.org/*",
        "zone_name": "example.org"
      },
      "kv_namespaces": [
        {
          "binding": "<MY_NAMESPACE>",
          "id": "<STAGING_KV_ID>"
        }
      ]
    }
  }
}
```

## Runtime

Workers use V8 engine runtime.

## Security

Every Cloudflare's data center has Workers runtime running within its own "isolates", which provides security yet performant.

**Isolate** is a lightweight "context" that provide THREE elements:

- Code.
- Variables that code can access.
- A safe environment for code to be executed within.

One instance of the Worker runtime can run >100K+ of "isolates", simultaneously.

"Isolate" memory is completely "isolated", so each piece of code can be protected from other untrusted user-written code in the same runtime instance.

**Isolates** depend on Linux kernel containerization technology, instead of waiting for VM spinning up for each function.

The overhead of a JS runtime is paid once, on the start of a container.

Workers processes are able to run limitless scripts with no individual overhead.

An isolate can start **~100x** faster than and consume an order of magnitude less memory than a Node process inside a container or VM.

## Workflow

```ts
export default {
  async fetch(request, env, ctx): Promise<Response> {
    return new Response("Hello World!");
  },
} satisfies ExportedHandler<Env>;
```

1. An HTTP Request is sent to `*.workers.dev` subdomain or to your Cloudflare-managed domain and received by Cloudflare's data centers.

1. A specific `fetch()` handler matching the given request is invoked, with request data being passed into as parameter.

1. A HTTP Response is sent back when `fetch()` handler return a `Response` object.
