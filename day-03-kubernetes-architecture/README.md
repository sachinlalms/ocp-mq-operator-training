# Day 03: Kubernetes Architecture

**Phase 1: Cloud & Container Foundations** | Duration: ~90 minutes | Level: Beginner

## Objectives

By the end of this day you will be able to:

- Explain why we need Kubernetes after learning containers
- Name the control plane and node components and what each does
- Describe what happens, step by step, when you create a Pod
- Explain declarative configuration and the reconcile loop (desired vs actual state)
- Read the structure of a Kubernetes YAML (apiVersion, kind, metadata, spec, status)
- Use `oc` to inspect nodes, pods, events, and the API on your OCP cluster

## Prerequisites

- Days 01 and 02 completed
- Access to your team's OCP cluster and the `oc` CLI (see the lab)

---

## 1. Why Kubernetes?

Running one container is easy. In real life you need to:

- Run many containers across many machines
- Restart them when they crash
- Replace failed machines
- Scale up and down
- Roll out new versions without downtime
- Give containers stable networking and storage

**Kubernetes (K8s)** is a container orchestrator that does this automatically. You describe *what you want*; Kubernetes keeps making it true.

> OpenShift is Kubernetes plus extra features and opinionated defaults (Phase 2). Everything today applies to OCP.

## 2. The big idea: declarative and reconciling

| Imperative (do this now) | Declarative (this is what I want) |
|---|---|
| "Start 3 containers" | "I want 3 replicas of this app" |
| You handle failures | Kubernetes handles failures |

Kubernetes constantly compares:

- **Desired state**: what you wrote in YAML (the `spec`)
- **Actual state**: what is really running (the `status`)

and acts to close the gap. This is the **reconcile loop**, and it is the idea behind controllers and Operators (Day 11).

```
   you write desired state
            |
            v
   +------------------+      observe      +-----------------+
   |    Controller    | <---------------- |  Actual state   |
   +------------------+                   +-----------------+
            | act (create / delete / fix)
            +-----------------------------> cluster
```

## 3. Cluster architecture

```
                      CONTROL PLANE
 +-----------------------------------------------------------+
 |  kube-apiserver  <-->  etcd (the database)                |
 |       ^                                                   |
 |  kube-scheduler     kube-controller-manager               |
 +-------|---------------------------------------------------+
         |
 +-------v---------+   +-----------------+   +---------------+
 | Worker node 1   |   | Worker node 2   |   | Worker node 3 |
 | kubelet         |   | kubelet         |   | kubelet       |
 | CRI-O runtime   |   | CRI-O runtime   |   | CRI-O runtime |
 | kube-proxy/SDN  |   | kube-proxy/SDN  |   | kube-proxy/SDN|
 | [Pod][Pod]      |   | [Pod][Pod]      |   | [Pod][Pod]    |
 +-----------------+   +-----------------+   +---------------+
```

### Control plane components

| Component | Role | Simple view |
|---|---|---|
| **kube-apiserver** | The front door. All reads and writes go through it (kubectl, oc, controllers, kubelets). Does authentication, authorization, admission | Reception desk |
| **etcd** | Distributed key-value store holding all cluster state | The cluster's memory |
| **kube-scheduler** | Chooses a node for each new unscheduled pod (resources, affinity, zones, taints) | Seat allocator |
| **kube-controller-manager** | Runs built-in controllers (Deployment, ReplicaSet, Node, Job, ...) | Supervisors who keep reality matching the plan |
| **cloud-controller-manager** | Talks to the cloud provider (load balancers, disks, nodes) where applicable | Cloud interpreter |

### Node components

| Component | Role |
|---|---|
| **kubelet** | Agent on each node: receives pod specs, starts containers via the runtime, reports health |
| **Container runtime (CRI-O on OpenShift)** | Pulls images and runs containers (Day 02) |
| **Network proxy / SDN** | Makes Services and pod networking work (Day 05 and Day 08) |

### OpenShift specifics (preview)

- Control plane nodes are typically 3 (for etcd quorum); workers run your apps
- On OCP the control plane components run as pods in namespaces such as `openshift-kube-apiserver` and `openshift-etcd`, and the node OS is RHCOS (Day 06)
- Plus extra API servers and operators: OpenShift API, OAuth, Cluster Operators (Day 10)

## 4. What happens when you create a Pod?

1. You run `oc apply -f pod.yaml`
2. **API server** authenticates you (who?), authorizes (RBAC: allowed?), runs admission checks (policy, SCC), validates
3. API server **stores** the Pod object in **etcd**
4. **Scheduler** sees an unscheduled pod, picks a node, writes `nodeName`
5. **Kubelet** on that node sees a pod assigned to it
6. Kubelet asks **CRI-O** to pull the image and start containers; sets up storage and networking
7. Kubelet reports status back to the API server, which stores it in etcd
8. `oc get pod` shows `Running`

> Key insight: components never call each other directly. They all *watch and update the API server*. This makes the system loosely coupled and resilient.

## 5. Anatomy of a Kubernetes object

```yaml
apiVersion: v1            # API group/version
kind: Pod                 # type of object
metadata:                 # identity
  name: hello-ubi
  namespace: my-project
  labels:
    app: hello
spec:                     # DESIRED state (you write this)
  containers:
    - name: main
      image: registry.access.redhat.com/ubi9/ubi-minimal
      command: ["sleep", "3600"]
status:                   # ACTUAL state (the system writes this)
  phase: Running
```

Remember: **you write `spec`, Kubernetes writes `status`.** Operators and the MQ `QueueManager` resource follow exactly this pattern.

Useful helpers:

- **Namespace** (a Project in OpenShift): a logical area for resources and access control
- **Labels**: key/value tags used to group and select objects
- `oc explain pod.spec` shows the built-in documentation of any field

## 6. The API and resource types

Everything is a REST resource: `GET /api/v1/namespaces/my-project/pods`.

```bash
oc api-resources | head       # list resource types
oc explain pod                # documentation
oc get pod -v=6               # show the API calls oc makes
```

**Custom Resource Definitions (CRDs)** add new types to the API. IBM MQ adds `QueueManager`. We cover this on Day 11.

## 7. Why this matters for MQ

| MQ on OpenShift step | Architecture link |
|---|---|
| You apply a `QueueManager` YAML | API server stores it in etcd |
| The MQ Operator notices it | A controller watching the API |
| Operator creates a StatefulSet, Services, Routes | More objects through the API |
| Scheduler places MQ pods | Spread across nodes and zones (Day 01) |
| Kubelet pulls the MQ image and starts it | CRI-O runs the container (Day 02) |
| Pod crashes and restarts | Reconcile loop |
| etcd lost | Cluster state lost, so etcd backup is vital |

## Key Takeaways

- Kubernetes orchestrates containers across machines
- Declarative: you set `spec`; controllers make `status` match
- Control plane: API server, etcd, scheduler, controller manager
- Nodes: kubelet, CRI-O, networking
- Everything talks through the API server
- The reconcile loop is the foundation for Operators

## Hands-on

Go to [labs/lab-03-explore-the-cluster.md](labs/lab-03-explore-the-cluster.md).

## Review

- [Quiz](quiz.md)
- [Common issues](troubleshooting.md)

## Further reading

- Kubernetes documentation: Concepts, Cluster Architecture (kubernetes.io)
- OpenShift documentation: Architecture
- *Kubernetes Components* diagram on kubernetes.io

---

**Previous:** [Day 02](../day-02-containers/) | **Next:** [Day 04: Kubernetes Workloads](../day-04-k8s-workloads/)
