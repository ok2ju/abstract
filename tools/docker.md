# Distroless Images — Summary

## Definition

A **distroless** image is a minimal Docker image that contains **only the application and its runtime dependencies**, without a full Linux distribution (no shell, no package manager, no system utilities).

A Google project: `gcr.io/distroless/*`. "Distroless" = "without a distribution".
## What's inside / what's not

| Included                       | Not included                          |
| ------------------------------ | ------------------------------------- |
| Application binary / code      | shell (`sh`, `bash`)                  |
| Runtime (`libc`, Node.js, JVM) | package manager (`apt`, `apk`)        |
| CA certificates, tzdata        | system utilities (`ls`, `curl`, `ps`) |
| Unprivileged `nonroot` user    | process manager                       |
## Comparison with neighbors

- **scratch** — a completely empty image (0 bytes), holding only a statically linked binary. Unlike scratch, distroless ships the required runtime (libc, certificates, tzdata).
- **Alpine** — still a mini-distribution with `musl libc`, `apk`, and `/bin/sh` (~5 MB). Distroless removes even this, closing off a whole class of in-container shell attacks.

## Advantages

1. **Security (the main one)** — no shell, so an attacker can't run arbitrary commands after a compromise; smaller attack surface and fewer CVEs in system packages.
2. **Size** — fewer layers and packages → faster pull and deploy.
3. **Cleaner vulnerability audits** — no dozens of OS packages to patch.

## Drawbacks / trade-offs

- **Harder to debug** — you can't shell into the container. Solutions: `:debug` image variants (with BusyBox) or ephemeral debug containers (`kubectl debug`).
- **Multi-stage build required** — build in a "fat" builder image, then copy the artifact into the distroless image.

## Example (Node.js, multi-stage)

```dockerfile
# Stage 1: build in a full-featured image
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: final distroless image — runtime + artifact only
FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["dist/index.js"]
```

**Note:** in distroless the `ENTRYPOINT` already points to the runtime (e.g. `node`), so `CMD` only passes the path to the script, without `node` in front. The exec form of `CMD` is mandatory (there is no shell for `sh -c`).