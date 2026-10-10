# Day 02: Containers

**Phase 1: Cloud & Container Foundations** | Duration: ~90 minutes | Level: Beginner

## Objectives

By the end of this day you will be able to:

- Explain what a container is and how it differs from a virtual machine
- Describe the two Linux features behind containers: namespaces and cgroups
- Explain images, layers, containers, registries, and tags
- Use Podman (or Docker) to pull, run, inspect, and stop containers
- Explain why containers need persistent storage for stateful apps like MQ
- Run an IBM MQ developer container locally

## Prerequisites

- Day 01 completed
- Podman or Docker installed (see the lab)
- ~4 GB free RAM and ~5 GB free disk

---

## 1. The problem containers solve

"It works on my machine" happens because apps depend on libraries, runtimes, and settings that differ between machines.

A **container** packages an application together with everything it needs to run (libraries, runtime, config), so it runs the same way on a laptop, a test server, and an OpenShift cluster.

## 2. Containers vs virtual machines

| | Virtual machine | Container |
|---|---|---|
| Includes | Full guest OS + app | App + libraries only |
| Kernel | Own kernel per VM | Shares the host kernel |
| Size | GBs | MBs to a few hundred MB |
| Start time | Minutes | Seconds |
| Isolation | Strong (hardware level) | Process level (namespaces, cgroups) |
| Density | Few per host | Many per host |

```
VM stack:                 Container stack:
App A | App B             App A | App B
Guest OS | Guest OS       Libs  | Libs
Hypervisor                Container runtime
Host OS                   Host OS (shared kernel)
Hardware                  Hardware
```

> A container is not a lightweight VM. It is an ordinary Linux process with limited visibility and limited resources.

## 3. What makes a container: namespaces and cgroups

| Feature | What it does | Simple view |
|---|---|---|
| **Namespaces** | Limit what a process can *see* (processes, network, filesystem mounts, hostname, users) | Blinkers |
| **cgroups** | Limit what a process can *use* (CPU, memory, I/O) | A usage quota |

Why this matters later: Kubernetes CPU and memory *requests and limits* are cgroups. A pod killed with `OOMKilled` hit its cgroup memory limit.

## 4. Images, layers, containers

| Term | Meaning |
|---|---|
| **Image** | Read-only template: files + metadata (a recipe plus ingredients) |
| **Layer** | Each build step adds a layer; layers are cached and shared |
| **Container** | A running instance of an image (a process) with a thin writable layer |
| **Containerfile / Dockerfile** | Text file with the build instructions |

Key rules:

- One image can start many containers
- Deleting a container deletes its writable layer, so **data written inside is lost** unless stored on a volume
- Images are **immutable**: to change an app, build a new image

## 5. Registries and image names

A **registry** stores and distributes images (Docker Hub, Quay.io, IBM Container Registry, Red Hat registry, OpenShift internal registry).

Image name anatomy:

```
icr.io/ibm-messaging/mq:9.4.0.0-r1
 |          |          |      |
registry  namespace  image   tag
```

- **Tag** = a label for a version (`latest` can change; pin a version for production)
- **Digest** (`@sha256:...`) = an exact, immutable reference
- Public pulls are free; private images need authentication, which is exactly why IBM entitlement keys appear in Day 13

## 6. Docker vs Podman vs CRI-O

| Tool | Role | Notes |
|---|---|---|
| Docker | Popular all-in-one tool with a background daemon | Licensing limits for large companies on Docker Desktop |
| **Podman** | Daemonless, rootless-capable, Docker-compatible CLI (`alias docker=podman`) | Red Hat's tool; used in this series |
| **CRI-O** | Lightweight runtime used *by Kubernetes/OpenShift nodes* | You rarely type CRI-O commands |
| Buildah / Skopeo | Build images / copy and inspect images | Related tools |

On OpenShift nodes, **CRI-O** runs your containers. Podman is for your laptop and for learning.

## 7. Key commands

```bash
podman pull <image>            # download an image
podman images                  # list local images
podman run -d --name web -p 8080:80 nginx   # start a container in background
podman ps                      # running containers (add -a for all)
podman logs web                # container output
podman exec -it web sh         # open a shell inside
podman inspect web             # full JSON details
podman stop web && podman rm web
podman rmi nginx               # remove image
podman volume create data      # create persistent volume
```

Flag cheat sheet: `-d` detach, `-p host:container` publish a port, `-e KEY=VALUE` environment variable, `-v vol:/path` mount a volume, `--name` give it a name, `--rm` delete when stopped.

## 8. Persistent storage and ports

- **Volume:** storage that outlives the container (essential for MQ queue and log data)
- **Port mapping:** `-p 1414:1414` forwards a host port to the container port

Later, these become Kubernetes **PersistentVolumeClaims** (Day 05) and **Services/Routes** (Days 05 and 08).

## 9. Why this matters for MQ

IBM publishes an MQ container image. The same image runs:

- On your laptop with Podman (today's lab)
- In a pod on OpenShift, started by the MQ Operator (Day 16)

Understanding the image, its environment variables, ports (1414 for MQ, 9443 for the web console), and its data volume (`/mnt/mqm`) makes Operator behaviour much easier to understand.

## Key Takeaways

- A container is an isolated Linux process, not a small VM
- Namespaces limit visibility; cgroups limit resources
- Images are immutable templates; containers are running instances
- Data inside a container is lost unless you use a volume
- Registries store images; names are `registry/namespace/image:tag`
- Podman for the laptop, CRI-O on OpenShift nodes

## Hands-on

Go to [labs/lab-02-podman-and-mq.md](labs/lab-02-podman-and-mq.md).

## Review

- [Quiz](quiz.md)
- [Common issues](troubleshooting.md)

## Further reading

- Podman documentation (podman.io)
- IBM MQ container image documentation (search "IBM MQ container image" on IBM Docs)
- Open Container Initiative (opencontainers.org)

---

**Previous:** [Day 01: Cloud Basics](../day-01-cloud-basics/) | **Next:** [Day 03: Kubernetes Architecture](../day-03-kubernetes-architecture/)
