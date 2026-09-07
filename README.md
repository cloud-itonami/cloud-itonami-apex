# cloud-itonami-apex

The Cloudflare Workers that own real `itonami.cloud` hostnames — the public
edge of the itonami surface.

| Worker | Host | Source |
|---|---|---|
| `agent-edge` | `agent.itonami.cloud` | JavaScript |
| `itonami-cloud-webhooks` | `hooks.itonami.cloud` | TypeScript |
| `itonami-fleet-dispatch` | `app.itonami.cloud` | TypeScript |
| `mcp-edge` | `mcp.itonami.cloud` | JavaScript |

Each has its own `wrangler` config, its own dependencies, and rolls back
independently of the others. Nothing here imports from `cloud-itonami-app`.

## What "apex" means here, and how the line was drawn

Not by choosing which Workers felt like edge services. By measuring which ones
reach into the app:

    agent-edge               0 references into the app tree
    itonami-cloud-webhooks   0
    itonami-fleet-dispatch   0
    mcp-edge                 0
    app-edge                 4  — requires cloud.itonami.app.fleet-core
                                  and .kotoba-oracle in PRODUCTION code

`app-edge` stayed in `cloud-itonami-app`. Two things made that the answer
rather than a judgement call: its dependency is in the shipped Worker, not
only in a test, and `kotoba-oracle` is required by six other app namespaces,
so it is shared foundation rather than edge code. `fleet-core` also has a
`.kotoba` twin under an active parity test, and splitting a namespace in the
middle of that migration would fork it.

The line landed where the hostnames already were: **the four Workers that own
real hosts are exactly the four that are independent, and the one that is
coupled owns no host at all** — `app-edge` deploys to workers.dev only, and its
config says so deliberately, so that a slice in progress cannot take a live
`itonami.cloud` host with it.

## Deploying

Each Worker deploys from its own directory:

```bash
cd services/<name> && npx wrangler deploy
```

Verify without deploying — this is the check that a move did not break a build,
and it discriminates (a syntax error in an entry point returns 1):

```bash
cd services/<name> && npx wrangler deploy --dry-run --outdir /tmp/out
```

All four were measured building from this tree before the repository was
created.

## Where it came from

Split out of `cloud-itonami/cloud-itonami-app` on 2026-09-07, after
`cloud-itonami-cli`. See `90-docs/adr/2609076000-the-apex-workers-leave-the-app.edn`.
