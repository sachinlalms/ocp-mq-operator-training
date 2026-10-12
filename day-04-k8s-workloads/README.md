# Day 04: Kubernetes Workloads

**Phase 1: Cloud & Container Foundations** | Duration: ~90 minutes | Level: Beginner to Intermediate

## Objectives

By the end of this day you will be able to:

- Explain what a Pod is and when it has more than one container
- Choose the right workload type: Deployment, StatefulSet, DaemonSet, Job, or CronJob
- Explain rolling updates and rollbacks
- Use probes (liveness, readiness, startup) and resource requests and limits
- Explain why IBM MQ runs as a StatefulSet and what "stable identity" means
- Create, scale, update, and break workloads on OCP with `oc`

## Prerequisites

- Days 01 to 03 completed
- OCP access and your own project (see Day 03 lab)

---

## 1. The Pod

A **Pod** is the smallest deployable unit. It wraps one or more containers that share:

- The same network (one IP, containers talk via `localhost`)
- The same storage volumes
- The same lifecycle (scheduled together on one node)

```
Pod
+----------------------------------+
|  main container   sidecar        |
|  (app)            (helper)       |
|     shared network + volumes     |
+----------------------------------+
```

| Pattern | Meaning | Example |
|---|---|---|
| **Single container** | Most common | An MQ queue manager container |
| **Sidecar** | Helper running alongside | Log shipper, metrics exporter |
| **Init container** | Runs to completion *before* the main container | Wait for a dependency, prepare a volume |

Rules to remember:

- Pods are **ephemeral**: when one dies, it is not "repaired", it is **replaced** (with a new IP)
- A bare Pod is not rescheduled if its node fails, so we use controllers

### Pod lifecycle phases

`Pending` (waiting to schedule/pull) → `Running` → `Succeeded` / `Failed`. Container states: `Waiting`, `Running`, `Terminated`. Restart behaviour is controlled by `restartPolicy`.

## 2. Resources: requests and limits

```yaml
resources:
  requests:        # used by the scheduler to choose a node (guaranteed minimum)
    cpu: 250m
    memory: 512Mi
  limits:          # enforced by cgroups (Day 02)
    cpu: "1"
    memory: 1Gi
```

| | CPU | Memory |
|---|---|---|
| Over the limit | Throttled (slowed) | Container killed: `OOMKilled` |
| Request too high | Pod stays `Pending` (no node fits) | Same |

`250m` = 250 millicores = a quarter of a CPU core.

## 3. Health probes

| Probe | Question | If it fails |
|---|---|---|
| **Startup** | Has the app finished starting? | Keeps waiting, then restarts |
| **Readiness** | Can it receive traffic now? | Removed from Service endpoints (not restarted) |
| **Liveness** | Is it still alive? | Container is restarted |

Probe types: `httpGet`, `tcpSocket`, `exec`. For MQ, a TCP probe on the listener port or a command check is common; the Operator configures this for you.

## 4. ReplicaSet and Deployment

- **ReplicaSet**: keeps N identical pods running (the reconcile loop from Day 03)
- **Deployment**: manages ReplicaSets and adds **rolling updates** and **rollbacks**. You almost never create a ReplicaSet directly

```
Deployment  ->  ReplicaSet (v2)  ->  Pods
                ReplicaSet (v1)  ->  (scaled to 0, kept for rollback)
```

Rolling update: new pods are started and old ones stopped gradually, so the app stays available.

```bash
oc set image deployment/web httpd-24=<new-image>
oc rollout status deployment/web
oc rollout history deployment/web
oc rollout undo deployment/web
```

Use a Deployment for **stateless** apps (web front ends, APIs): pods are interchangeable.

> OpenShift also has the older `DeploymentConfig`. It is deprecated; use standard `Deployment` for new work.

## 5. StatefulSet (this is the one MQ uses)

For apps that need **identity and their own storage**.

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Random (`web-7d9f-abcde`) | Stable, ordered (`qm-0`, `qm-1`, `qm-2`) |
| Storage | Shared or none | **Own PVC per pod** (`volumeClaimTemplates`) |
| Start/stop order | All at once | Ordered (0, 1, 2); reverse on scale down |
| Network identity | Via Service only | Stable DNS per pod via a **headless Service** |
| Replacement | New random pod | Same name, **same volume** re-attached |

```
StatefulSet "qm"            Headless Service
 qm-0 --- PVC data-qm-0     qm-0.qm-svc ...
 qm-1 --- PVC data-qm-1     qm-1.qm-svc ...
 qm-2 --- PVC data-qm-2     qm-2.qm-svc ...
```

If `qm-1` dies, a new pod named `qm-1` starts and mounts the **same** PVC, so the data is still there.

Important: **deleting a StatefulSet or scaling down does not delete the PVCs** by default (protects your data). Clean them up deliberately.

## 6. DaemonSet

One pod on **every node** (or every matching node). Use for node-level agents: log collectors, monitoring agents, network components. OpenShift itself uses many DaemonSets (DNS, SDN, node tuning).

## 7. Job and CronJob

| Type | Use |
|---|---|
| **Job** | Run a task to completion (and retry on failure), e.g. a migration, a one-off MQSC script |
| **CronJob** | A Job on a schedule, e.g. nightly cleanup or backup. Uses cron syntax: `*/5 * * * *` |

## 8. Choosing the workload

| Need | Use |
|---|---|
| Stateless app, many identical replicas | Deployment |
| Stable name + own storage per pod (databases, MQ, Kafka) | StatefulSet |
| One pod per node | DaemonSet |
| Run once to completion | Job |
| Run on a schedule | CronJob |

## 9. Why this matters for MQ

| MQ need | Workload feature |
|---|---|
| Queue manager has persistent data (queues, logs) | StatefulSet with PVCs |
| Same identity after restart | Stable pod name and re-attached volume |
| Native HA: 3 replicas, one active and two standby | StatefulSet with 3 pods (spread across zones) |
| Memory limit matters | Requests/limits and `OOMKilled` (Day 02) |
| Don't send traffic to a not-ready queue manager | Readiness probe |
| Stuck queue manager gets restarted | Liveness probe |

When you apply a `QueueManager` YAML (Day 16), the MQ Operator creates the StatefulSet, Services, and PVCs for you. Today you learn what it is creating.

## Key Takeaways

- A Pod wraps containers sharing network and storage; pods are replaced, not repaired
- Deployment = stateless + rolling updates; StatefulSet = stable identity + own storage
- DaemonSet = one per node; Job/CronJob = run to completion/on a schedule
- Requests drive scheduling; limits are enforced (CPU throttled, memory OOMKilled)
- Readiness controls traffic; liveness controls restarts
- StatefulSet PVCs outlive the pods, so delete them on purpose

## Hands-on

Go to [labs/lab-04-workloads.md](labs/lab-04-workloads.md).

## Review

- [Quiz](quiz.md)
- [Common issues](troubleshooting.md)

---

**Previous:** [Day 03](../day-03-kubernetes-architecture/) | **Next:** [Day 05: Kubernetes Networking & Storage](../day-05-k8s-networking-storage/)
