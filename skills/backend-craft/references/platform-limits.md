# Platform limits and infrastructure choices

## When to check limits

Ordinary work (a new endpoint, a field, a query) fits the platform; do not audit limits for it. Check the documented limits only when the work involves what usually collides with them:

- file uploads or downloads and large payloads;
- concurrent or parallel work;
- long-running or background jobs;
- cron or scheduled tasks;
- long-lived connections (websockets, streaming);
- heavy processing (media, documents, AI);
- bulk data operations;
- third-party quotas and rate limits.

Limits change: read the provider's current documentation instead of relying on memory.

## Common collisions and alternatives

| The work needs | Usually collides with | Alternatives to present |
|---|---|---|
| Large uploads (hundreds of MB, GB) | Request body size, CPU time, memory | Direct-to-storage upload with a presigned URL; multipart upload; processing afterwards in a background job |
| Long-running work | Request timeouts, CPU time per request | A queue and a consumer; a durable workflow; a separate worker or container |
| Scheduled tasks | Execution time per run, overlapping runs | Platform cron that enqueues jobs; idempotent jobs with a lock |
| Long-lived connections | Connection duration, per-instance memory | A platform primitive for stateful connections (for example Durable Objects on Workers); a dedicated realtime service |
| Heavy processing | CPU, memory, native dependencies | A queue consumer on a runtime that supports it; a specialized service |
| Bulk operations | Statement or batch limits, timeouts | Chunked batches in a background job; resumable progress |
| Third-party quotas | Rate limits, monthly quotas | Caching, backoff with jitter, request coalescing, a queue to smooth bursts |

If the work does not fit, stop and present the alternatives with their trade-offs. Never ship a workaround that only breaks in production (for example a multi-GB upload through an edge function).

## Comparing runtimes and deploy targets

Only when the developer asks for a recommendation. The developer decides; never choose the cloud, hosting or deploy platform, create paid resources or deploy without approval.

1. **Start from the product's real requirements:** state, connections, background jobs, cron, file access, native dependencies, latency, data residency, scale, budget.
2. **Compare only options that actually satisfy them.**
3. **Keep categories honest:** a runtime or platform (Workers, Node on a VM) is not a packaging technology (Docker) is not a framework (Hono, NestJS) is not a hosting provider. Never offer them as equivalent options in one question.
4. **For each option:** capabilities, limits, local-development parity, operational burden, cost drivers and lock-in, with links to the official docs.
5. **Recommend one** with the reason, and list what would change the recommendation.

Once a runtime or deploy decision is made, do not reopen it in every task; raise a concrete problem with it instead.
