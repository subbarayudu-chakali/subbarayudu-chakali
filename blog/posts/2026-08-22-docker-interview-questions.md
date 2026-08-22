# Docker Interview Questions & Answers

A practical, interview-ready reference for **Docker** — images, containers, Dockerfiles,
networking, volumes, Compose, and the operational and security topics that come up in
real interviews. Grouped by theme, with answers concise enough to say aloud.

---

## Fundamentals

**1. What is Docker?**

Docker is an open-source platform for building, shipping, and running applications in
**containers** — lightweight, isolated, portable units that package an application with
its dependencies so it runs consistently across environments.

**2. What is a container?**

A container is a running instance of an image: an isolated process (or set of processes)
with its own filesystem, network, and resource limits, sharing the host OS kernel. It's
the runtime unit.

**3. Container vs. virtual machine?**

A **VM** virtualizes hardware and runs a full guest OS (heavy, slow to boot, GBs). A
**container** virtualizes the OS and shares the host kernel (lightweight, starts in
milliseconds, MBs). Containers give higher density; VMs give stronger isolation.

**4. What is a Docker image?**

A read-only template — a layered, immutable snapshot of a filesystem plus metadata —
used to create containers. Images are built from a Dockerfile and stored in registries.

**5. Image vs. container — the relationship?**

An image is the blueprint (class); a container is a running instance of it (object). One
image can spawn many containers.

**6. What is Docker Hub / a registry?**

A registry is a store for images. **Docker Hub** is the default public registry; others
include GHCR, Amazon ECR, Google Artifact Registry, and private/self-hosted registries.
`docker push`/`pull` move images to/from a registry.

**7. What is the Docker architecture?**

A client-server model: the **Docker CLI** talks to the **Docker daemon (`dockerd`)** over
a REST API. The daemon builds/runs/manages images, containers, networks, and volumes.
Underneath, `containerd` and `runc` handle the container lifecycle.

**8. What is `containerd` and `runc`?**

`containerd` is the high-level container runtime that manages the container lifecycle
(pull, run, supervise). `runc` is the low-level OCI runtime that actually creates
containers using Linux namespaces and cgroups.

**9. What Linux features make containers possible?**

**Namespaces** (isolation of PID, network, mount, user, IPC, UTS) and **cgroups**
(resource limits: CPU, memory, I/O), plus union filesystems for layering.

---

## Images & Dockerfiles

**10. What is a Dockerfile?**

A text file with instructions to build an image — a reproducible recipe. Each
instruction typically creates a layer.

**11. Explain the common Dockerfile instructions.**

- `FROM` — base image.
- `RUN` — execute a command at build time (creates a layer).
- `COPY` / `ADD` — copy files into the image (`ADD` also handles URLs/tar extraction).
- `WORKDIR` — set the working directory.
- `ENV` — set environment variables.
- `EXPOSE` — document a listening port (doesn't publish it).
- `CMD` — default command/args when the container starts.
- `ENTRYPOINT` — the executable that always runs.
- `ARG` — build-time variable.
- `VOLUME` — declare a mount point.
- `USER` — set the user to run as.
- `HEALTHCHECK` — how to test container health.

**12. Difference between `CMD` and `ENTRYPOINT`?**

`ENTRYPOINT` sets the executable that always runs; `CMD` provides default arguments (or a
default command). If both exist, `CMD` supplies default args to `ENTRYPOINT`. `CMD` is
easily overridden at `docker run`; `ENTRYPOINT` is not (unless `--entrypoint`).

**13. Difference between `COPY` and `ADD`?**

`COPY` simply copies local files/directories. `ADD` does that plus can fetch remote URLs
and auto-extract local tar archives. Best practice: prefer `COPY` unless you need `ADD`'s
extra behavior.

**14. What are image layers?**

Each Dockerfile instruction creates a read-only layer stacked via a union filesystem.
Layers are cached and shared between images, making builds and pulls efficient. The
container adds a thin writable top layer.

**15. What is the build cache and how do you leverage it?**

Docker reuses cached layers when an instruction and its inputs are unchanged. Order the
Dockerfile so rarely-changing steps (installing dependencies) come **before**
frequently-changing steps (copying source), so code changes don't bust the dependency cache.

**16. What is a multi-stage build?**

A Dockerfile with multiple `FROM` stages where you build in one stage and copy only the
needed artifacts into a small final image — dramatically shrinking image size and
excluding build tools:

```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN go build -o /app

FROM gcr.io/distroless/base
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

**17. How do you reduce Docker image size?**

Use small base images (alpine, distroless, slim), multi-stage builds, combine `RUN`
commands and clean up in the same layer, use `.dockerignore`, avoid installing unneeded
packages, and remove caches (`apt-get clean`, `--no-cache`).

**18. What is `.dockerignore`?**

A file listing paths excluded from the build context sent to the daemon (like `.git`,
`node_modules`, secrets), speeding builds and avoiding leaking files into the image.

**19. Difference between `ARG` and `ENV`?**

`ARG` is a **build-time** variable available only during `docker build` (via
`--build-arg`). `ENV` sets an **environment variable** present at build time and
persisted in the running container.

**20. What does image tagging mean, and what's wrong with `latest`?**

Tags label image versions (`myapp:1.2.0`). `latest` is just a default tag, not
"newest" — it's mutable and can point anywhere, causing non-reproducible deploys. Pin
explicit version tags (or digests) in production.

**21. What is an image digest?**

An immutable SHA256 content hash (`myimage@sha256:...`) that uniquely identifies exact
image content, unlike mutable tags. Use digests for reproducibility/security.

**22. What is a distroless / scratch image?**

`scratch` is an empty base (for fully static binaries). **Distroless** images contain
only your app and runtime dependencies — no shell or package manager — reducing attack
surface and size.

---

## Running containers

**23. What does `docker run` do, and name key flags.**

Creates and starts a container from an image. Common flags: `-d` (detached), `-p host:container` (publish port), `-e` (env var), `-v` (volume/mount), `--name`, `--rm`
(remove on exit), `--network`, `--restart`, `-it` (interactive TTY).

**24. Difference between `docker run`, `docker start`, and `docker exec`?**

`docker run` creates and starts a **new** container. `docker start` restarts an existing
stopped container. `docker exec` runs a command **inside a running** container (e.g.
`docker exec -it web bash`).

**25. How do you view running/all containers?**

`docker ps` shows running containers; `docker ps -a` shows all (including stopped).

**26. How do you see container logs?**

`docker logs <container>` (add `-f` to follow, `--tail N` for last N lines).

**27. What's the difference between stopping and killing a container?**

`docker stop` sends SIGTERM then SIGKILL after a grace period (graceful). `docker kill`
sends SIGKILL immediately (forceful).

**28. What happens to data when a container is removed?**

The container's writable layer is deleted — any data not stored in a volume or bind
mount is lost. That's why persistent data belongs in volumes.

**29. What are container restart policies?**

`--restart` options: `no` (default), `on-failure[:max]`, `always`, `unless-stopped` —
they control whether the daemon restarts a container after exit or on daemon restart.

**30. How do you limit container resources?**

Flags like `--memory` (`-m`), `--cpus`, `--cpu-shares`, and `--pids-limit` apply cgroup
limits to cap memory, CPU, and process count.

**31. What is a HEALTHCHECK?**

An instruction/flag defining a command Docker runs periodically to determine if a
container is healthy; the status shows in `docker ps` and can gate orchestrators/dependencies.

---

## Storage & data

**32. How does Docker persist data?**

With **volumes**, **bind mounts**, and **tmpfs mounts**. Volumes are the preferred
mechanism for persistent data because Docker manages them and they're decoupled from the
container lifecycle.

**33. Volume vs. bind mount?**

A **volume** is managed by Docker (stored under Docker's area, portable, easy to back
up). A **bind mount** maps a specific host path into the container (host-dependent, good
for development/live code). A **tmpfs** mount lives in memory only.

**34. How do you create and use a named volume?**

```bash
docker volume create data
docker run -v data:/var/lib/app myimage
```

The volume persists independently of the container.

**35. What is a tmpfs mount?**

An in-memory mount that isn't written to disk — used for sensitive or ephemeral data
that shouldn't persist.

**36. What is the copy-on-write (CoW) filesystem?**

Containers share read-only image layers; when a file is modified, it's copied to the
container's writable layer first. This makes container startup fast and storage efficient.

---

## Networking

**37. What Docker network drivers exist?**

- **bridge** — default; containers on a private network on a single host.
- **host** — container shares the host's network stack (no isolation).
- **none** — no networking.
- **overlay** — multi-host networking (Swarm).
- **macvlan** — assign a MAC/IP so the container appears as a physical device.

**38. What is the default bridge network vs. a user-defined bridge?**

On the default `bridge`, containers reach each other only by IP. On a **user-defined
bridge**, Docker provides automatic **DNS resolution by container name** and better
isolation — the recommended approach for multi-container apps.

**39. How do containers communicate?**

On the same user-defined network, by container name (built-in DNS). Across the host
boundary, via **published ports** (`-p`). Across hosts, via overlay networks or an
external service mesh/load balancer.

**40. Difference between EXPOSE and publishing a port (`-p`)?**

`EXPOSE` only documents which port the app listens on (metadata). Publishing with
`-p host:container` (or `-P`) actually maps and makes it reachable from the host/outside.

**41. How do you inspect a container's network?**

`docker network ls`, `docker network inspect <net>`, and `docker inspect <container>` to
see IPs, ports, and attached networks.

---

## Docker Compose

**42. What is Docker Compose?**

A tool to define and run **multi-container** applications with a single YAML file
(`compose.yaml`/`docker-compose.yml`), describing services, networks, and volumes. One
command (`docker compose up`) brings the whole stack up.

**43. Show a minimal Compose file.**

```yaml
services:
  web:
    build: .
    ports: ["8080:80"]
    depends_on: [db]
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```

**44. Common Compose commands?**

`docker compose up -d` (start), `down` (stop and remove), `ps`, `logs -f`, `build`,
`exec`, `pull`. `down -v` also removes named volumes.

**45. What does `depends_on` do — and not do?**

It controls **start order**, ensuring `db` starts before `web`. It does **not** wait for
the dependency to be *ready* — use healthchecks (`condition: service_healthy`) or
retry logic in the app for true readiness.

**46. How do you scale a service with Compose?**

`docker compose up --scale web=3` runs multiple instances of a service (best paired with
a load balancer/reverse proxy in front).

**47. Compose vs. Kubernetes/Swarm?**

Compose is ideal for **local development** and single-host multi-container setups.
Kubernetes/Swarm are **orchestrators** for production clustering, scaling, self-healing,
and rolling updates across many hosts.

---

## Orchestration & registries

**48. What is Docker Swarm?**

Docker's native clustering/orchestration mode that turns multiple Docker hosts into a
single virtual host, offering services, scaling, rolling updates, and overlay networking.
Kubernetes is the more widely adopted orchestrator today.

**49. How do you push an image to a registry?**

Tag it with the registry path and push:

```bash
docker tag myapp:1.0 ghcr.io/owner/myapp:1.0
docker login ghcr.io
docker push ghcr.io/owner/myapp:1.0
```

**50. What is a private registry and why use one?**

A registry you control (Harbor, ECR, GAR, self-hosted) for private images, access
control, vulnerability scanning, and keeping images inside your network.

---

## Security

**51. What are Docker security best practices?**

Run as a **non-root user**, use minimal/trusted base images, scan for vulnerabilities,
don't bake secrets into images, drop unnecessary Linux capabilities, use read-only
filesystems where possible, keep the daemon patched, and pin image digests.

**52. Why avoid running containers as root?**

A root process inside a container can be more dangerous if it escapes isolation or a
volume is mounted, and it violates least privilege. Use `USER` in the Dockerfile or
`--user` at runtime.

**53. How do you handle secrets in Docker?**

Don't put them in images or `ENV` baked at build. Use Docker/Swarm **secrets**, mount
them at runtime, use a secrets manager, or inject via the orchestrator. Build secrets
can use BuildKit's `--secret` mount so they don't persist in layers.

**54. How do you scan images for vulnerabilities?**

Tools like `docker scout`, Trivy, Grype, or registry-integrated scanning inspect image
layers/packages for known CVEs. Integrate scanning into CI.

**55. What is Docker Content Trust (DCT)?**

Image signing/verification using Notary so you only pull/run signed, trusted images
(`DOCKER_CONTENT_TRUST=1`).

**56. What are Linux capabilities in the Docker context?**

Fine-grained root privileges. Docker drops many by default; you can drop more with
`--cap-drop` and add specific ones with `--cap-add`, following least privilege.

**57. What does `--privileged` do and why is it risky?**

It gives the container nearly all host capabilities and device access, effectively
removing isolation — a major security risk; avoid unless absolutely required.

---

## Operations & troubleshooting

**58. How do you clean up unused Docker resources?**

`docker system prune` removes stopped containers, unused networks, dangling images (add
`-a` for all unused images, `--volumes` for volumes). Also `docker image/container/volume prune`.

**59. How do you debug a container that keeps crashing?**

Check `docker logs`, inspect exit code with `docker ps -a`, run interactively
(`docker run -it --entrypoint sh image`), verify env/volumes/ports, and use
`docker inspect` for config. Look for missing files, bad CMD, or dependency readiness.

**60. What are dangling images?**

Untagged images (`<none>`) left behind by rebuilds. Remove with `docker image prune`.

**61. How do you copy files between host and container?**

`docker cp <container>:/path/in/container ./host/path` (and vice versa).

**62. How do you inspect image/container details?**

`docker inspect <name>` returns detailed JSON (config, mounts, networks, env). `docker history <image>` shows layers and how they were built.

**63. What is BuildKit?**

The modern Docker build engine (default now) offering faster parallel builds, better
caching, build secrets/SSH mounts, and cache export/import. Enabled via `DOCKER_BUILDKIT=1` or `docker buildx`.

**64. What is `docker buildx`?**

An extended build CLI (built on BuildKit) supporting multi-platform builds (e.g.
`linux/amd64` and `linux/arm64`), advanced caching, and building for multiple
architectures from one command.

**65. How do you build a multi-architecture image?**

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t owner/app:1.0 --push .
```

BuildKit builds and pushes a manifest list covering all platforms.

---

## Quick-fire round

- **Build an image?** `docker build -t name:tag .`
- **List images?** `docker images`.
- **Default registry?** Docker Hub.
- **Persist data?** Volumes.
- **Multi-container local dev?** Docker Compose.
- **Shrink an image?** Multi-stage build + small base.
- **Run a command in a live container?** `docker exec -it`.
- **Ignore files from build context?** `.dockerignore`.
- **Immutable image reference?** A digest (`@sha256:...`).
- **See resource usage live?** `docker stats`.

---

These questions cover most Docker interviews end to end — from container-vs-VM through
multi-stage builds, networking, Compose, and security hardening. The best follow-up
prep is to containerize a real app yourself: write a multi-stage Dockerfile, run it as a
non-root user, wire it up with a database in Compose, and push a multi-arch image to a
registry. Doing it once makes every answer above concrete.
