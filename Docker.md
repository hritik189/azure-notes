# Docker: Core Architecture, From the Ground Up

A deep dive into how Docker actually works under the hood — not just the commands, but the Linux primitives, the daemon internals, and the full lifecycle. Written for someone who already uses Docker and wants to understand *why* it works.

---

## 1. The Core Insight: What a Container Actually Is

The single most important thing to internalize: **a container is just a normal Linux process that has been lied to.**

It is not a virtual machine. There is no guest OS, no hypervisor, no emulated hardware. A container is a process running directly on your host kernel, but the kernel has been configured to give that process a deliberately limited and isolated view of the system. It *thinks* it has its own filesystem, its own network interfaces, its own process tree, and its own set of users — but it's all an illusion constructed by three Linux kernel features working together.

### Containers vs. Virtual Machines

| Aspect | Virtual Machine | Container |
|---|---|---|
| Isolation boundary | Hypervisor + full guest OS | Kernel namespaces + cgroups |
| Kernel | Each VM has its own kernel | All containers share the host kernel |
| Boot time | Seconds to minutes | Milliseconds |
| Size | Gigabytes (full OS) | Megabytes (just app + deps) |
| Overhead | High (full OS per VM) | Near-zero (just a process) |

The trade-off: because containers share the host kernel, you cannot run a Windows container on a Linux kernel, or a Linux container natively on macOS/Windows. This is exactly why Docker Desktop runs a lightweight Linux VM underneath on Mac and Windows — there has to be a Linux kernel somewhere for Linux containers to share.

---

## 2. The Three Linux Primitives That Make Containers Possible

Everything Docker does for isolation comes down to three kernel features. Docker did not invent these — it packaged them into a usable workflow.

### 2.1 Namespaces — "What the process can SEE"

Namespaces partition kernel resources so that one set of processes sees one set of resources, and another set sees a different set. When you start a container, Docker creates a fresh set of namespaces for it. The key ones:

- **PID namespace** — the container gets its own process tree. The first process inside is PID 1, and it cannot see host processes. This is why `ps aux` inside a container shows only its own processes.
- **NET namespace** — its own network stack: interfaces, routing tables, iptables rules, ports. This is how two containers can both "listen on port 8080" without colliding.
- **MNT namespace** — its own filesystem mount points. This is what makes the container see *its* root filesystem instead of the host's.
- **UTS namespace** — its own hostname and domain name.
- **IPC namespace** — its own inter-process communication (shared memory, semaphores).
- **USER namespace** — its own user/group ID mappings. Root inside the container (UID 0) can be mapped to an unprivileged UID on the host, which is a major security control.

When people say a container is "isolated," they almost always mean namespaces.

### 2.2 Control Groups (cgroups) — "What the process can USE"

If namespaces control what a process can *see*, cgroups control what it can *consume*. cgroups limit and account for resource usage:

- CPU shares and quotas (`--cpus`, `--cpu-shares`)
- Memory limits (`--memory`) — exceed it and the OOM killer terminates the process
- Block I/O bandwidth
- Number of PIDs (prevents fork bombs)

This is what stops one container from starving every other container on the host. In Kubernetes, your `resources.requests` and `resources.limits` map directly onto cgroup settings — worth knowing given your AKS work.

### 2.3 Union Filesystems — "How the filesystem is layered"

This is how Docker images are built efficiently. A union filesystem (OverlayFS on modern Linux) lets you stack multiple read-only directories and present them as a single merged filesystem, with one writable layer on top. We'll come back to this in the images section because it's central to how images work.

**Summary:** Namespaces (isolation) + cgroups (resource limits) + union FS (layered storage) = a container. Docker is the tooling that orchestrates all three.

---

## 3. The Docker Architecture: Client, Daemon, Registry

Docker uses a **client-server architecture**. There are three top-level players.

```
   ┌──────────────┐         REST API          ┌─────────────────────┐
   │ Docker Client│ ───────over socket──────▶ │   Docker Daemon     │
   │ (docker CLI) │                            │   (dockerd)         │
   └──────────────┘                            │                     │
                                               │  builds, runs,      │
   ┌──────────────┐         pull / push        │  manages containers │
   │   Registry   │ ◀──────────────────────────│  images, volumes,   │
   │ (Docker Hub, │ ───────────────────────────▶  networks          │
   │  ACR, etc.)  │                            └─────────────────────┘
   └──────────────┘
```

### 3.1 Docker Client

The `docker` command you type. It is *just a frontend*. When you run `docker run nginx`, the client translates that into REST API calls and sends them to the daemon. The client does almost no real work itself — it can even talk to a daemon running on a completely different machine (via the `DOCKER_HOST` environment variable). "Docker client" and "docker CLI" are the same thing.

### 3.2 Docker Daemon (dockerd) — also called the "Docker server"

This is the long-running background process that does all the actual work: building images, creating and running containers, managing networks, volumes, and the local image cache. It listens on a Unix socket (`/var/run/docker.sock`) by default, or optionally a TCP port. When you saw "Docker daemon" and "Docker server" as separate terms earlier — they're the same thing. The daemon *is* the server.

A critical detail: because the daemon runs as root and exposes a socket, anyone who can talk to that socket effectively has root on the host. This is the classic Docker security concern and the reason rootless Docker and alternatives like Podman exist.

### 3.3 Docker Host

The physical or virtual machine where the daemon runs. It holds the kernel that containers share, the local image store, and the running containers themselves.

### 3.4 Registry

Remote storage for images. The daemon `pull`s images from it and `push`es images to it. More on this in section 8.

---

## 4. Inside the Daemon: It's Not One Monolith Anymore

This is the part most people skip, and it's what separates surface knowledge from real understanding. Modern Docker is *not* a single program that does everything. Over the years it was broken apart into layered components, several of which are now CNCF/industry standards.

```
dockerd  (high-level: API, networking, volumes, build)
   │
   ▼
containerd  (container lifecycle manager: pull, storage, supervise)
   │
   ▼
containerd-shim  (one per container; keeps container alive if containerd restarts)
   │
   ▼
runc  (the low-level OCI runtime: actually creates namespaces + cgroups)
   │
   ▼
[ your container process ]
```

- **dockerd** — high-level engine. Handles the API, image building, networking, volumes. Delegates the actual running of containers downward.
- **containerd** — the core container runtime. It manages the complete container lifecycle: pulling images, managing storage, and supervising execution. Notably, **Kubernetes talks directly to containerd** (via the CRI interface), skipping dockerd entirely. This is why "Docker" was deprecated as a Kubernetes runtime — K8s never needed the dockerd layer, just containerd underneath. Your AKS nodes run containerd, not the full Docker engine.
- **containerd-shim** — a tiny process that sits between containerd and each container. It lets the container keep running even if containerd or dockerd is restarted or upgraded.
- **runc** — the low-level reference implementation of the **OCI (Open Container Initiative) runtime spec**. This is the piece that actually makes the syscalls to create namespaces and cgroups and then `exec`s your process inside them. runc does its job and exits; the shim takes over supervision.

The **OCI** matters here: it's the open standard (image spec + runtime spec) that means a Docker-built image can run on containerd, CRI-O, Podman, or any compliant runtime. Docker images aren't a proprietary format — they're OCI images.

---

## 5. Images: The Layered Blueprint

An **image** is a read-only template — the blueprint. A **container** is a running (or stopped) instance of an image. One image, many containers, all sharing the same underlying read-only layers.

### 5.1 Layers and the Union Filesystem

An image is built from a stack of **read-only layers**. Each instruction in a Dockerfile that changes the filesystem creates a new layer:

```dockerfile
FROM node:20-alpine        # Layer 1: base OS + Node runtime
WORKDIR /app               # (metadata, no real fs layer)
COPY package*.json ./      # Layer 2: dependency manifests
RUN npm ci                 # Layer 3: installed node_modules
COPY . .                   # Layer 4: your application source
CMD ["node", "server.js"]  # (metadata: default command)
```

Each layer stores only the *diff* from the layer below it. OverlayFS stacks them and presents a single unified view. When a container runs, Docker adds one thin **writable layer** on top (the "container layer"). All changes the running container makes — writing files, modifying data — go into that writable layer using **copy-on-write**: to modify a file from a lower read-only layer, it's first copied up into the writable layer, then changed.

Two big consequences:

1. **Sharing.** Ten containers from the same image share the exact same read-only layers on disk. Only their writable layers differ. This is why containers are so cheap to spin up.
2. **Ephemerality.** When you delete a container, its writable layer is destroyed. Any data written inside the container that wasn't in a volume is gone. This is *the* reason volumes exist (section 7).

### 5.2 The Build Cache — and Why Layer Order Matters

When you `docker build`, the daemon caches each layer. On a rebuild, if an instruction and all its inputs are unchanged, Docker reuses the cached layer instead of re-running it. The moment one layer's input changes, that layer **and every layer after it** are rebuilt (cache busted).

This is why the Dockerfile above copies `package.json` and runs `npm ci` *before* copying the full source. Your source code changes constantly, but your dependencies rarely do. By putting dependency installation earlier, a code change only busts the final `COPY . .` layer — `npm ci` stays cached. Reverse the order and every code change reinstalls all dependencies. This ordering discipline is one of the highest-leverage Dockerfile optimizations.

### 5.3 Image Naming and Tags

```
registry.example.com/namespace/repository:tag
        │                  │          │      │
        │                  │          │      └─ version label (default: latest)
        │                  │          └──────── image name
        │                  └─────────────────── user/org/project
        └────────────────────────────────────── registry host (default: Docker Hub)
```

`nginx:1.25` is really `docker.io/library/nginx:1.25`. For ACR it'd look like `myregistry.azurecr.io/workflo-backend:v1.2.0`. Under the hood, images are also addressable by a content **digest** (`sha256:...`) which is immutable — tags can move, digests cannot. Pinning to digests is a supply-chain hardening practice.

### 5.4 Multi-Stage Builds

A pattern worth knowing: use one stage with the full build toolchain, then copy only the built artifacts into a clean, minimal final stage. The bulky build tools never ship in the final image.

```dockerfile
FROM node:20 AS build
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM node:20-alpine          # fresh, tiny final image
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

The final image contains none of the build-stage cruft — smaller images, smaller attack surface, faster pulls.

---

## 6. The Container Lifecycle

A container moves through distinct states:

```
created ──start──▶ running ──stop──▶ stopped ──start──▶ running
   │                  │                  │
   │                  ├─pause──▶ paused  │
   │                  │                  └──rm──▶ removed (gone)
   └──────────────────┴───────────────── rm ───▶ removed
```

What actually happens on `docker run nginx`:

1. **Client → daemon:** the CLI sends the run request to dockerd.
2. **Image resolution:** daemon checks the local store for `nginx`; if absent, it pulls each missing layer from the registry.
3. **Writable layer:** daemon creates the thin writable layer on top of the image's read-only layers.
4. **Namespaces + cgroups:** via containerd → runc, the kernel creates the isolated namespaces and applies cgroup limits.
5. **Network setup:** daemon attaches the container to a network (default bridge), creating a virtual interface and assigning an IP.
6. **Process exec:** runc execs the image's `CMD`/`ENTRYPOINT` as PID 1 inside the namespaces. The shim takes over supervision; runc exits.
7. **Running:** the container lives exactly as long as its PID 1 process lives. When PID 1 exits, the container stops. (This is why a container with no foreground process exits immediately — there's nothing to keep it alive.)

That last point trips people up constantly: a container is not a box you log into, it's a wrapper around *one* process. No process, no container.

---

## 7. Storage: Making Data Survive

Because the writable layer dies with the container, you need a way to persist data outside it. Docker offers three mechanisms:

### 7.1 Volumes (the recommended way)

Managed by Docker, stored in a part of the host filesystem the daemon controls (`/var/lib/docker/volumes/`). They exist independently of any container, survive container deletion, can be shared between containers, and can be backed by drivers (including cloud/network storage).

```bash
docker volume create appdata
docker run -v appdata:/var/lib/mysql mysql:8
```

### 7.2 Bind Mounts

Map a specific host directory straight into the container. Great for development (live-editing source code into a running container) but tightly couples the container to the host's directory layout.

```bash
docker run -v /home/sam/code:/app node:20
```

### 7.3 tmpfs Mounts

Stored in host memory only, never written to disk. Vanish when the container stops. Useful for secrets or scratch data you never want persisted.

In Kubernetes this concept generalizes into Volumes, PersistentVolumes, and PersistentVolumeClaims — same fundamental problem (decouple data lifetime from container lifetime), bigger abstraction.

---

## 8. Networking

Each container gets its own network namespace. Docker wires these together with several **network drivers**:

- **bridge** (default) — Docker creates a virtual bridge (`docker0`) on the host. Containers on the same bridge get private IPs and can reach each other; the outside world reaches them only through published ports (`-p 8080:80` maps host 8080 to container 80). On a **user-defined bridge** (as opposed to the default one), containers can resolve each other by *name* via Docker's built-in DNS — this is how Compose services find each other.
- **host** — the container shares the host's network namespace directly. No isolation, no port mapping needed, maximum performance. The container's port 80 *is* the host's port 80.
- **none** — no networking at all. Fully isolated.
- **overlay** — spans multiple Docker hosts, letting containers on different machines communicate as if on one network. This is the basis for multi-host orchestration (Swarm); Kubernetes solves the same problem with its own CNI plugins.

---

## 9. Registries: Docker Hub, ACR, and Private Registries

A **registry** is the storage and distribution system for images. A **repository** is a collection of related images (usually different versions of the same app) within a registry.

- **Docker Hub** (`docker.io`) — the default public registry, run by Docker Inc. Like GitHub for images: public repos are free, official images (`nginx`, `postgres`, etc.) live under the `library/` namespace. Note the pull rate limits for anonymous/free accounts — relevant in CI.
- **Azure Container Registry (ACR)** — a private, managed registry in Azure. Given your AKS/ACR pipeline work, this is your bread and butter: build in Azure Pipelines, push to ACR, and AKS pulls from ACR using managed identity. ACR also supports geo-replication, image scanning, and content trust.
- **Self-hosted** — you can run the open-source `registry` image yourself, or use Harbor, GitLab Container Registry, etc.

**Push/pull mechanics:** images transfer layer-by-layer. If a layer already exists on the destination (matched by digest), it's skipped. This is why pushing a new app version after only a code change is fast — the base OS and dependency layers are already there; only your changed top layers move.

---

## 10. Docker Compose

Compose solves a real pain: running a multi-container app by hand means juggling many long `docker run` commands plus manually creating networks and volumes. Compose declares the whole stack in one `compose.yaml`:

```yaml
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - MONGO_URL=mongodb://db:27017/workflo
    depends_on:
      - db
  db:
    image: mongo:7
    volumes:
      - dbdata:/data/db
volumes:
  dbdata:
```

`docker compose up` then: creates a dedicated network (so `api` can reach `db` by the name `db` via DNS), creates the named volume, builds/pulls images, and starts everything in dependency order. It's declarative single-host orchestration. For production multi-host orchestration you graduate to Kubernetes — but the mental model (declare desired state, let the tool reconcile) is the same, which is why Compose is such good preparation for K8s.

---

## 11. Docker Desktop (and why Mac/Windows are different)

On Linux, the daemon runs natively because the host kernel is already Linux. On macOS and Windows there's no Linux kernel for containers to share, so **Docker Desktop** runs a lightweight Linux VM in the background and runs the daemon inside it. Docker Desktop bundles: the Linux VM, dockerd, the CLI, Compose, a Kubernetes option, and a GUI dashboard. It's a convenience package, not a different Docker — the architecture underneath is identical.

---

## 12. How It All Connects to Kubernetes (your world)

Tying it back to your stack, since this is where it matters for you:

- A K8s **Pod** wraps one or more containers that share network and storage namespaces — built on the same primitives described above.
- K8s talks to **containerd** directly via the CRI, bypassing dockerd. Your AKS nodes don't run the full Docker engine.
- Pod `resources.requests/limits` → **cgroups**.
- Pod **Volumes / PVC** → the same data-persistence problem as Docker volumes, generalized.
- K8s **Services + CNI** → the same networking problem as Docker networks, generalized across a cluster.
- Images still come from a **registry** (ACR in your case), still pulled layer-by-layer.

Docker taught the industry the *packaging* format (OCI images) and the *single-host* runtime. Kubernetes is the multi-host orchestration layer built on top of those same foundations. Understanding the Docker internals above means the Kubernetes abstractions stop feeling like magic — they're the same kernel primitives, scaled out and declared rather than commanded.

---

## Quick Reference: The Mental Model in One Paragraph

A container is a host process isolated by **namespaces** (what it sees) and constrained by **cgroups** (what it uses), running on a stack of **union-filesystem layers** (the image) with one writable layer on top. You drive it with the **client** (CLI), which sends API calls to the **daemon** (dockerd), which delegates through **containerd** and **runc** to actually create the container against the **host** kernel. Images live in **registries** and transfer layer-by-layer. **Volumes** outlive containers. **Compose** declares multi-container stacks on one host; **Kubernetes** does it across many.