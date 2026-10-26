# Liveness and Readiness Probes

A probe is a small check that the platform (Kubernetes, Docker, a load balancer) runs against your service again and again. Its job is to answer one question. There are two common questions, and mixing them up causes most probe problems.

| | Liveness | Readiness |
|---|---|---|
| Question | Is the process alive, or is it stuck? | Can it serve requests right now? |
| Typical endpoint | `/healthz`, `/livez` | `/readyz` |
| What it checks | Only the process itself | The process plus what it needs (DB, other services, warm cache) |
| On failure | The container is **restarted** | Traffic **stops** going to it. It is not restarted |
| Good for | Deadlocks, hung event loops | Startup, losing a dependency, graceful shutdown |

## Simple analogy

A shop assistant.

- **Liveness:** is the person awake? If not, replace them.
- **Readiness:** is the person able to serve customers? Maybe they are still opening the till. Then customers go to another counter. Nobody gets fired.

## Liveness

Liveness answers "is it broken in a way only a restart can fix?"

- Keep it cheap and local: return `200` if the process can handle a request at all.
- **Do not check dependencies here.** If the database goes down and liveness checks the database, every pod restarts at once. The restarts don't fix the database, and they add load when it comes back.

## Readiness

Readiness answers "should I get traffic right now?"

- It checks what the service needs in order to do its job. Example: a gateway returns `503` from `/readyz` while `users` or `orders` cannot be reached.
- It runs for the whole life of the service, not only at startup. If a dependency fails later, the instance leaves rotation. When the dependency recovers, the instance comes back on its own.
- During shutdown, the service can fail readiness first. Traffic drains away, then the process exits cleanly.

## Startup probe (Kubernetes)

Some apps start slowly. A startup probe holds off the liveness and readiness probes until it succeeds once. A slow start then doesn't count as "dead".

## Example: Kubernetes

```yaml
livenessProbe:
  httpGet: { path: /healthz, port: 8080 }
  periodSeconds: 10
  failureThreshold: 3      # restart after ~30s of failures
readinessProbe:
  httpGet: { path: /readyz, port: 8080 }
  periodSeconds: 5
  failureThreshold: 2      # removed from Service endpoints after ~10s
startupProbe:
  httpGet: { path: /healthz, port: 8080 }
  periodSeconds: 5
  failureThreshold: 30     # allow up to ~150s to start
```

## Example: Docker Compose

Docker has a single `HEALTHCHECK`, with no separate liveness and readiness. Its result is a status: `starting`, `healthy` or `unhealthy`.

```yaml
services:
  gateway:
    depends_on:
      users: { condition: service_healthy }   # start gateway only after users is healthy
```

- `service_healthy` only controls **startup order**. After startup, nothing watches the dependency.
- Plain Docker does **not** restart an `unhealthy` container. Swarm and Kubernetes act on probe results; Compose does not.
- If you remove `service_healthy`, the gateway starts too early. `/readyz` returns `503` until its dependencies are up. That window is exactly what readiness protects against.

## Rules of thumb

- Liveness checks the process only. Readiness checks the process and its dependencies.
- Keep probes fast, with a short timeout. Make them unauthenticated, and leave them out of business metrics.
- Fail readiness with `503`, not a `200` that has an error in the body. Platforms only look at the status code.
- Set thresholds so that a single slow response does not trigger a restart.

## Sources

- [Kubernetes: Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)
- [Docker: HEALTHCHECK](https://docs.docker.com/reference/dockerfile/#healthcheck)
- [Compose: depends_on](https://docs.docker.com/reference/compose-file/services/#depends_on)
