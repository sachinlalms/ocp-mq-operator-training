# Day 03 Quiz

Answers at the bottom.

**1.** Which component is the only one that components and users talk to directly for reading and writing cluster state?
- A) etcd  B) kube-apiserver  C) kubelet  D) scheduler

**2.** Where is cluster state stored?
- A) In the scheduler  B) In etcd  C) On worker disks only  D) In the registry

**3.** Which component decides *which node* a new pod runs on?
- A) kubelet  B) kube-proxy  C) kube-scheduler  D) CRI-O

**4.** Which component starts the containers on a node?
- A) Scheduler  B) API server  C) kubelet (using CRI-O)  D) etcd

**5.** In a Kubernetes object, who writes the `status` section?
- A) You  B) The system  C) The registry  D) The scheduler only

**6.** What is the reconcile loop?
- A) A backup job
- B) Comparing desired state to actual state and acting to close the gap
- C) A network protocol
- D) A type of volume

**7.** You delete one pod of a 3-replica Deployment. What happens?
- A) Nothing  B) A controller creates a replacement  C) The cluster stops  D) etcd is wiped

**8.** Which command shows the Events for a pod?
- A) `oc get pod -o name`  B) `oc describe pod <name>`  C) `oc login`  D) `oc whoami`

**9.** True or false: you write the `spec`, Kubernetes writes the `status`.

**10.** A pod stays `Pending` with a `FailedScheduling` event. Which component could not do its job?

**11.** In your own words: why does the MQ Operator rely on the reconcile loop?

**12.** Why do we normally run three control plane nodes?

---

## Answers

1. B
2. B
3. C
4. C
5. B
6. B
7. B
8. B
9. True
10. The scheduler: no node satisfied the pod's needs (resources, taints, affinity, or unbound storage).
11. The Operator watches `QueueManager` resources and keeps comparing the desired configuration with what exists, creating or fixing StatefulSets, Services, and Routes until they match.
12. etcd needs a majority (quorum). With three members, the cluster survives the loss of one.
