# Day 02: Common Issues and Misconceptions

## Misconceptions

| Misconception | Reality |
|---|---|
| "A container is a small VM" | It is a process sharing the host kernel, isolated by namespaces and limited by cgroups |
| "Data in a container is saved" | The writable layer disappears with the container; use volumes |
| "`latest` always means newest and safest" | `latest` is just a tag that can change; pin versions for production |
| "Docker is required for Kubernetes" | Kubernetes and OpenShift use CRI-O or containerd |
| "Podman is a different thing from Docker" | The CLI is nearly identical; Podman is daemonless |

## Practical issues

| Symptom | Likely cause | Fix |
|---|---|---|
| `Cannot connect to Podman` (macOS/Windows) | Podman machine not started | `podman machine start` |
| `port is already allocated` / `address already in use` | Another process uses the host port | Use a different host port, e.g. `-p 8081:80`, or stop the other process |
| `Error: short-name resolution` / image not found | Registry not specified | Use the full name, e.g. `docker.io/library/nginx` |
| `unauthorized` or `denied` on pull | Private image needs login | `podman login <registry>` (IBM entitled images need an entitlement key later) |
| MQ container exits immediately | `LICENSE=accept` missing, or bad env var, or architecture mismatch | Check `podman logs qm1`; add `-e LICENSE=accept`; try a matching tag for your CPU |
| MQ web console not reachable | Port 9443 not published, or startup not finished | Check `podman ps` ports; wait for logs to show the web server started |
| Browser certificate warning on 9443 | Self-signed certificate | Expected in the lab; proceed. Real systems use CA-signed certs (Day 17) |
| Permission denied on mounted folder | SELinux or user mapping | Add `:Z` to the volume mount on SELinux systems, or use a named volume |
| Out of disk space | Old images and containers | `podman system df`, then `podman system prune` |
| Container name already in use | Old container still exists | `podman rm -f <name>` |

## Debug checklist (use this habit for every container problem)

1. `podman ps -a`: is it running, exited, or restarting?
2. `podman logs <name>`: what did it say before it stopped?
3. `podman inspect <name>`: ports, environment, mounts, exit code
4. `podman exec -it <name> sh`: look inside (if still running)
5. Check the basics: image name and tag, ports, environment variables, volumes

> This exact checklist maps to `oc get pods`, `oc logs`, `oc describe pod`, and `oc rsh` on OpenShift.
