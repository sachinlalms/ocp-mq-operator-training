# Day 03: Common Issues and Misconceptions

## Misconceptions

| Misconception | Reality |
|---|---|
| "The scheduler starts the containers" | It only picks a node. The kubelet starts containers |
| "etcd stores my application data" | etcd stores cluster *state* (objects), not your MQ messages |
| "Kubernetes and OpenShift are separate things" | OpenShift is Kubernetes plus extra components and defaults |
| "Components call each other directly" | They all watch and update the API server |
| "A bare Pod is restarted on another node if its node dies" | No. Use a controller (Deployment, StatefulSet) |
| "Control plane nodes run my apps" | By default, app pods run on workers |

## Practical issues with `oc` and pods

| Symptom | Likely cause | Fix |
|---|---|---|
| `Unauthorized` / token expired | Login token expired | Copy a fresh login command from the console |
| `x509: certificate signed by unknown authority` | Cluster uses a private CA | Install/trust the CA. For a lab only: `--insecure-skip-tls-verify=true` |
| `Unable to connect to the server` | Wrong URL, VPN off, or proxy | Check `oc whoami --show-server`, VPN, proxy variables |
| `Error from server (Forbidden)` | RBAC: your user lacks permission | Check the project (`oc project`); ask for access; cluster-wide commands need admin |
| Pod `Pending` | No node fits (resources, taints, quota) or PVC unbound | `oc describe pod`, read Events (`FailedScheduling`) |
| Pod `ImagePullBackOff` / `ErrImagePull` | Wrong image name/tag, registry auth, network | Check name in `oc describe pod`; check pull secret; test pull from a node/laptop |
| Pod `CrashLoopBackOff` | Container starts then exits | `oc logs <pod> --previous`, check command and config |
| Pod `CreateContainerConfigError` | Missing ConfigMap/Secret or bad config | `oc describe pod` Events |
| Pod `OOMKilled` | Exceeded memory limit (cgroup, Day 02) | Raise the limit or fix the app |
| Image runs on Docker but fails on OCP | OCP runs containers as a random non-root user; privileged ports (<1024) | Use images built for non-root, e.g. UBI httpd-24 on 8080 (Day 07 covers SCC) |
| `exceeded quota` | Project resource quota reached | `oc describe quota`; clean up or request more |

## The universal debug flow

```bash
oc get pods                        # state?
oc describe pod <name>             # Events at the bottom
oc logs <name> [--previous]        # what did the app say?
oc get events --sort-by=.lastTimestamp
oc get pod <name> -o yaml          # full detail, spec and status
oc rsh <name>                      # shell inside, if running
```

Read order: **status, events, logs**. Almost every issue shows up in one of them.
