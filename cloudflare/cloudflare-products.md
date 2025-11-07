# Cloudflare Products

There are a lot of products on Cloudflare Developer Platform. Cloud engineers will need to consider and pick a subset of services for their use cases.

> [!IMPORTANT]
>
> Don't waste your time learning Cloudflare Pages or Workers, their Free Tier quota is nowhere near AWS.
>
> There are only THREE services that's worth to learn:
>
> - Cloudflare Zero Touch.
> - Cloudflare Workers AI.
> - Cloudflare Turnstile.

## Overview

Cloudflare, is a company that provides services on its global network of servers. It's one of the largest networks on the Internet.

Cloudflare's services are categorized into 4 product lines:

- SASE and SSE services.
- Application services.
- Infrastructure services.
- Developer Platform.

Three first services are for private and public organizations, businesses, governments, individual consumers. In general, non tech-savvy clients that need technological consultance from Cloudflare.

**Cloudflare Developer Platform** is the product line that we, cloud engineer, care about. It includes Cloudflare Workers, which allows engineers to deploy serverless code globally.

### Cloud Computing

In the early days of the web, we have on-premise infrastructure. Basically anyone who wanted to build a web application had to buy their own physical hardware and install and configure web server program into them. It requires painstakingly time and money.

Then came Cloud Computing. It is defined as hosting computing resources (virtual machines, storage, databases, networking services) on 3rd-party servers. There are several major vendors: AWS, MS Azure, Google Cloud Platform (GCP), Cloudflare.

Cloud computing allows organizations to rent a fixed number of servers or server space. To prepare for seasonal or unplanned traffic spikes, organizations have to overpurchase server space to ensure their applications do not go down because of high request volume from end users or customers.

### Serverless Computing (Serverless)

_Definition:_ It's a subset of cloud computing, a method of providing backend services on an pay-as-you-go basis. Cloud's customers aren't required to calculate how much server space or machines they need to rent.

Despite the name "serverless", physical servers are still existed, but engineers do not need to be aware of them.

_Characteristics:_

- **Abstracted:** Unlike traditional hosting providers, a serverless computing provider take card or server management, provisioning, allowing developers and organizations to focus on writing and deploying logic.
- **Pay-for-value:** or usage model paradigm. Instead of paying for fixed amount of computing resources that may be underutilized or exceeded, users pay as much as users are charged based on their computation usage. However, usage is defined differently per serverless computing provider so make sure you've done thorough research.
- **Auto-scaling:** The service is responsible for the scalability of your application. It scales automatically to handle both low points and surges in request traffic.
- **Event-driven execution model:** when an event (HTTP request, Worker Cron Trigger, ...) invokes a Worker, the Worker code will execute.

_Ideal for:_ Event-driven applications, Microservices, Workloads with unpredictable traffic patterns.

_Limitations:_

- **Cold starts:** There are initial startup time when a function hasn't been used recently. Not a problem for functions that are used often.
- **Resource constraints:** There might be time, memory, CPU constraints that are set up by cloud vendors:
  - The total amount of time from the start to end of an invocation of a Worker is known as duration.
  - The amount of time the CPU actually spends doing work during a given request is known as CPU time.
- **Statelessness:** Functions don't maintain state between executions.
- **Vendor Lock-in:** Code might be tied to specific provider services

## Products Overview

> Browse all Cloudflare's products here: https://developers.cloudflare.com/products/

Here are some of the most popular products:

<!-- prettier-ignore -->
| Groups | Product | Use Case | Ideal for |
| --- | --- | --- | --- |
| Consumer services | Registrar | Buy a new domain | Secure, performance DNS query from [Cloudflare DNS](https://developers.cloudflare.com/dns/) |
| Developer platform | Pages and Pages Functions | Configure and deploy a static sites with minimal dynamic functionality | Landing Pages, Blogs, Simple e-commerce website for local business, Pet projects, Online CVs, ... |
| Developer platform | Workers | Full-stack applications | Front-end applications, Back-end applications, Serverless AI inference, Background jobs, ... |
| Storage | Workers KV | Key-value storage | API Gateway Configuration (or Service Routing), Feature Flags, Personalization (A/B Testing) |
| Storage | R2 | No egress object storage | Web assets (images, videos, ...), Large object (Machine Learning model's datasets, Analytics datasets, Log and event data), Strong consistency per object application |
| Storage | D1 | SQLite-based database | Relational data (user profiles, e-commerce product listings and orders, ...), Ad-hoc query, High ratio of reads to writes workloads |
| Storage | Durable Objects | Special kind of Worker which uniquely combines compute and storage | Real-time Collaborative application (Chat application, Game server), Consistent, transactional storage, Data Locality |
| Storage | Hyperdrive | Service which accelerates queries users make to existing database | Scale web application that is built on top of on-premise databases |
| Media | Images | Store, transform, optimize, deliver images at scale | Image hosting platforms, e-commerce platforms, ... |

Built-in products: no manual set up, no configuration

- Application Performance CDN, Load Balancing, ...
- Application Security: DDoS protection, WAF, ...
- Artificial Intelligence: Workers AI, AI Gateway, Vectorize, ...

## Bindings

The following are cloud services that can work with Cloudflare Pages by specifying bindings inside Wrangler configuration file:

- D1: serverless relational database.
- Durable Objects: coordination and consistent storage.
- Hyperdrive: connect to existing database.
- Workers KV: serverless key-value store.
- R2: S3-compatible object storage
- Vectorize: build AI-powered applications.
- Services: the ability to call a Worker from Pages Function.
- Workers Analytics Engine: visualize metadata collected in Dashboard
- Workers AI: self-hosted, customizable SLM.

For Cloudflare Workers, there are a lot more. But Pages covered most of the services for basic use cases.

> [!CAUTION]
>
> According to a [Cloudflare Blog written in 2025-04-08](https://blog.cloudflare.com/full-stack-development-on-cloudflare-workers/), Pages is deemed to be **sunset** in the future. All of the resources are poured into Workers.

## Free-tier quotas

| Service         | Operation                  | Quota                 |
| --------------- | -------------------------- | --------------------- |
| Pages & Workers | # of inbound requests      | **100k** / day        |
|                 | Compute duration           | **10ms** / invocation |
|                 | I/O duration               | Unlimited             |
|                 | Subrequests                | Unlimited             |
|                 | Build duration             | 3000 minutes / month  |
|                 | Build concurrency          | 1                     |
|                 | Log events                 | 200k / day            |
|                 | Log retention interval     | 3 days                |
| D1              | # of read query            | 5M / day              |
|                 | # of write query           | 100k / day            |
|                 | Storage capaity            | 5GB total             |
| Durable Objects | # of requests              | 100k / day            |
|                 | Compute duration           | 13k GB-s / day        |
|                 | # of read rows             | 5M rows / day         |
|                 | # of write rows            | 100k rows / day       |
|                 | Storage capacity           | 5GB total             |
| Hyperdrive      | # of read + write query    | 100k / day            |
| Workers KV      | # of read query            | 100k / day            |
|                 | # of write query           | 1k / day              |
|                 | # of delete query          | 1k / day              |
|                 | # of list query            | 1k / day              |
|                 | Storage capacity           | 1GB total             |
| R2              | Class A (write) operations | 1M / month            |
|                 | Class B (read) operations  | 10M / month           |
|                 | Egress                     | Free                  |
| Vectorize       | # of dimension query       | 30M / month           |
|                 | Dimension storage capacity | 5M total              |
| Workers AI      | Neurons                    | **10k/day**           |

**Storage group:**

<!-- prettier-ignore -->
| Feature | Workers KV | R2 | Durable Objects | D1 |
| --- | --- | --- | --- | --- |
| Maximum storage per account | 1 GiB | 10 GiB | 5 GiB (SQLite-backed) + 50 GiB (KV-backed) | 5 GiB |
| Maximum size per value | 25 MiB/value | 5 TiB/object | 128 KiB/object | 500 MiB/database |
| Storage grouping name | Namespace | Bucket | Durable Object | Database |
| Consistency model | Eventual updates (~60s to be reflected) | Strong (read-after-write) | Serializable (with transactions) | Serializable (no replicas) / Causal (with replicas) |
| Supported APIs | Workers, HTTP/REST API | Workers, S3-compatible | Workers | Workers, HTTP/REST API |

## Built with Cloudflare

Here's how you can build one application using Cloudflare:

- Workers deployed the code.
- Storage group's products hosted the assets.
- Enhance the application performance by speeding up content delivery using CDN.
- Protect the application from malicious activity such as DDoS by configuring Web Application Firewall (WAF).
- Route traffic (Load Balancing, Waiting Room). These are paid products.

## Source of truth

If you're building your project, there are 2 ways to configure project settings:

- Web UI [Cloudflare dashboard](https://dash.cloudflare.com).
- Wrangler configuration files and Wrangler CLI.

```jsonc
/**
 * Cloudflare Pages Wrangler configuration file references:
 * - Pages Bindings API and Wrangler CLI: https://developers.cloudflare.com/pages/functions/bindings/
 * - Pages Wrangler Configuration: https://developers.cloudflare.com/workers/wrangler/configuration/
 */
{
  /* ======================================================================== */
  /* INHERITABLE KEYS                                                         */
  /* ======================================================================== */

  // alphanumeric + hyphen only
  "name": "my-project",

  // path to your project's build output folder
  // CAUTION: if added, when deploy, Pages will use local/non-production configuration
  // NOTE: omit to ensure the file is used for local developement only, even when `wrangler pages deploy` is run
  // NOTE: ignored in Workers
  "pages_build_output_dir": "./_site", // 11ty

  // determine which version of the Workers runtime is used
  // set compatibility dates is to "Latest" in Dashboard to avoid explicitly set later
  "compatibility_date": "2025-04-15", // NOTE: might be newer now

  // enable features from upcoming features of the Workers runtime
  // used together with `compatibility_date:`
  "compatibility_pages": ["nodejs_compat"],

  // DO NOT send usage data to CF
  "send_metrics": false,

  // impose limits on execution at runtime
  // docs: https://developers.cloudflare.com/pages/functions/wrangler-configuration/#limits
  "limits": {
    "cpu_ms": 5 // 5ms per invocation
  },

  // by default, Pages Functions are invoked in a data center closest to the user
  // but sometimes, in the case of heavy use back-end logic, it may be more performant for them to be run near the back-end infrastructure
  // docs: https://developers.cloudflare.com/workers/configuration/smart-placement/
  "placement": {
    "mode": "smart"
  },

  // upload server-side source maps to give correct stack traces in logs
  "upload_source_maps": true,

  /* ======================================================================== */
  /* NON-INHERITABLE KEYS                                                     */
  /* ======================================================================== */

  // Environment Variable
  // string value, can store serialized data such as JSON
  "vars": {
    "ENVIRONMENT": "development"
  },

  // Secrets
  // special `env:`-like bindings allow you to attach encrypted text values to Pages Function for use cases like API keys, auth tokens, etc.
  // not supported by config file (duh!)

  // D1
  // Cloudflare's Relational Database service
  "d1_databases": [
    {
      "binding": "DB",
      "database_name": "<DB_NAME>",
      "database_id": "<DB_ID>"
    }
  ],

  // Hyperdrive
  // Connect to existing databases from Workers/Pages Functions
  "compatibility_pages": ["nodejs_compat_v2"], // required!
  "hyperdrive": [
    {
      "binding": "HD1",
      "id": "<HD1_ID>",
      "localConnectionString": "postgres://user:password@localhost:5432/databasename"
    }
    // ...
  ],

  // Cloudflare's global, low-latency, key-value store
  // Data store in small # of centralized data centers then cache it in Cloudflare's data centers
  "kv_namespaces": [
    {
      "binding": "KV1",
      "id": "<NAMESPACE_ID1>",
      "preview_id": "<PREVIEW_ID1>"
      // required when develop remotely
      // optional when develop locally, also used by `wrangler dev`
    }
    // ...
  ],

  // Durable Objects -
  "durable_objects": {
    "bindings": [
      {
        "name": "DO1",
        "class_name": "MockDurableObjClass", // when changed, perform a migration
        "script_name": "", // name of Worker where Durable Object is defined
        "environment": "" // the env `script_name` bind to
      }
    ]
  },

  // R2 buckets
  // Store large amount of unstructured data
  // In Local environment, files are written to local storage, instead of preview/product bucket
  "r2_buckets": [
    {
      "binding": "<BINDING_NAME1>",
      "bucket_name": "<BUCKET_NAME1>"
    }
    // ...
  ]

  // "vectorize": [],
  // "services": [],
  // "analytics_engine_datasets": [],
  // "ai": []
  // and many more that supported by Workers runtime...
}
```

```js
interface Env {
  ENVIRONMENT: string;      // vars:
  API_KEY: string;          // secret
  TODO_LIST: KVNamespace;   // kv_namespaces:
  MY_DB: D1Database;        // d1_database:
  BUCKET: R2Bucket;         // r2_bucket:
}

export const onRequest: PagesFunction<Env> = async (context) => {
  console.log(context.env.ENVIRONMENT === "development");
  console.log(context.env.API_KEY);

  // D1
  const ps = context.env.MY_DB.prepare("SELECT * FROM users");
  const data = await ps.first();
  return Response.json(data);

  // KV
  const task = await context.env.TODO_LIST.get("Task:123"); // naming convention
  return new Response(task);

  // DO
  const id = context.env.DURABLE_OBJECT.newUniqueId();
  const stub = context.env.DURABLE_OBJECT.get(id);
  // pass the req down to the durable obj
  return stub.fetch(context.request);

  // Hyperdrive
  import postgre from "postgres";
  const db = postgres(context.env.HYPERDRIVE.connectionString);
  try {
    const result = await db("SELECT id, name, value FROM records;");

    return Response.json({
      result: result
    })
  } catch (e) {
    return Response.json({
      error: e.message, {
        status: 500
      }
    })
  }

  // R2
  const obj = await context.env.BUCKET.get("some-key"); // get() method
  if (obj === null) {
    return new Response("Not found", { status: 404 });
  }
  return new Response(obj.body);
}
```

```sh
# interact with bindings locally, test functionality without login to dashboard
npx wrangler dev
npx wrangler pages dev

# environment variable
npx wrangler pages dev --binding="${ENVIRONMENT_VARIABLE_NAME}"="${ENVIRONMENT_VARIABLE_VALUE}"

# secret
touch .dev.vars
touch .dev.vars.development
touch .dev.vars.production
touch ".dev.vars.${ENV_NAME}"

cat <<EOF > .dev.vars
SECRET_KEY="value"
API_TOKEN="abcdefghi0123456789"
EOF

npx wrangler "${COMMAND}" --env "${ENV_NAME}"

# KV namespaces
npx wrangler pages dev "${OUTPUT_DIR}" --kv=TODO_LIST

# Hyperdrive
export WRANGLER_HYPERDRIVE_LOCAL_CONNECTION_STRING_TEST_DB="postgres://user:password@localhost:5432/databasename"
npx wrangler dev $OUTPUT_DIR

# Durable Object
npx wranger pages dev --do="${BINDING_NAME}"="${CLASS_NAME}@${SCRIPT_NAME}"

# R2
npx wrangler pages dev "${OUTPUT_DIR}" --r2=BUCKET

# D1
npx wrangler pages dev "${OUTPUT_DIR}" --d1="${MY_DB}"

# download Dashboard config of an existing Pages project
# CAUTION: override existing local config file, make sure you backup it
npx wrangler pages download config "${PROJECT_NAME}"
```

## Reference

- [Cloudflare's Docs "Choose a data or storage product"](https://developers.cloudflare.com/workers/platform/storage-options/)
- [2025-04-08, Cloudflare Blog's "Your frontend, backend, and database — now in one Cloudflare Worker"](https://blog.cloudflare.com/full-stack-development-on-cloudflare-workers/)
- [Cloudflare Docs "Products > Pages > Functions > Bindings"](https://developers.cloudflare.com/pages/functions/bindings/)
- [Build applications with Cloudflare Workers (Learning Paths)](https://developers.cloudflare.com/learning-paths/workers/concepts/)
- [2025-04-08, Cloudflare Blog's "Your frontend, backend, and database — now in one Cloudflare Worker"](https://blog.cloudflare.com/full-stack-development-on-cloudflare-workers/)
