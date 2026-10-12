# Day 04 Quiz

Answers at the bottom.

**1.** What do containers in the same Pod share?
- A) Nothing  B) Network and volumes  C) A kernel only  D) A registry

**2.** Which workload gives pods stable names like `qm-0`, `qm-1`, and a separate PVC for each?
- A) Deployment  B) DaemonSet  C) StatefulSet  D) Job

**3.** Which workload runs one pod on every node?
- A) Deployment  B) DaemonSet  C) CronJob  D) ReplicaSet

**4.** Which value does the scheduler use to decide where to place a pod?
- A) Limits  B) Requests  C) Labels only  D) Image tag

**5.** A container exceeds its memory limit. What happens?
- A) It is throttled  B) It is `OOMKilled`  C) Nothing  D) It moves node

**6.** Which probe decides whether a pod receives traffic?
- A) Liveness  B) Readiness  C) Startup  D) None

**7.** Which probe failure restarts the container?
- A) Readiness  B) Liveness  C) Both  D) Neither

**8.** You update a Deployment to a bad image. What protects availability?
- A) Rolling update keeps old pods until new ones are Ready
- B) The scheduler
- C) etcd
- D) Nothing

**9.** Which command rolls a Deployment back?
- A) `oc rollout undo deployment/web`  B) `oc delete deployment/web`  C) `oc rollback node`  D) `oc restart`

**10.** After you scale a StatefulSet down, what happens to its PVCs?
- A) Deleted  B) Kept  C) Moved to another project  D) Encrypted

**11.** What does an init container do?

**12.** Why does the MQ Operator use a StatefulSet rather than a Deployment?

**13.** A pod is `Running` but shows `0/1` Ready. What is wrong, and will it receive traffic?

---

## Answers

1. B
2. C
3. B
4. B
5. B
6. B
7. B
8. A
9. A
10. B
11. It runs to completion before the main containers start, for setup tasks such as waiting for a dependency or preparing a volume.
12. A queue manager needs stable identity and its own persistent storage that follows the pod when it is replaced. StatefulSets provide stable names and per-pod PVCs.
13. The readiness probe is failing; the pod will not receive Service traffic (it is not restarted).
