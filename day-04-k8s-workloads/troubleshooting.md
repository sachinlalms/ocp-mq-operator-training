# Day 04: Common Issues and Misconceptions

## Misconceptions

| Misconception | Reality |
|---|---|
| "A Pod is a container" | A Pod wraps one or more containers that share network and storage |
| "Pods get repaired" | Pods are replaced; the new one has a new IP (and, in a Deployment, a new name) |
| "A failing readiness probe restarts the container" | It only removes the pod from traffic. Liveness restarts |
| "Deleting a StatefulSet deletes the data" | PVCs remain by default; you must delete them deliberately |
| "Limits = what the scheduler uses" | The scheduler uses **requests**; limits are enforced at runtime |
| "Use a Deployment for MQ with a PVC, it's the same" | StatefulSet gives stable identity and per-pod volumes; the Operator uses it for good reason |
| "DeploymentConfig is the OpenShift way" | Deprecated; use Deployment |

## Practical issues

| Symptom | Likely cause | Fix |
|---|---|---|
| Pod `Pending`, `Insufficient cpu/memory` | Requests larger than free node capacity | Lower requests, add capacity, check quotas |
| Pod `Pending`, `unbound PersistentVolumeClaim` | No default StorageClass or no capacity | `oc get pvc`, `oc get storageclass`, set `storageClassName` |
| Rollout stuck at "Waiting for rollout to finish" | New pods not Ready (bad image, failing probe) | `oc get pods`, `oc describe pod`, then `oc rollout undo` |
| `ImagePullBackOff` | Bad name/tag or registry auth | Check image in `oc describe pod`; check pull secret |
| `CrashLoopBackOff` | App exits or fails liveness | `oc logs <pod> --previous`; check command, config, probes |
| `Running` but `0/1 Ready` | Readiness probe failing | `oc describe pod`, check probe port/path/delay |
| Restart counter keeps increasing | Liveness failing or `OOMKilled` | `oc describe pod`, check Last State and Reason |
| `OOMKilled` | Memory limit too low | Raise limit, find the leak |
| StatefulSet pod `demo-1` won't start while `demo-0` is unhealthy | Ordered startup waits for the previous pod to be Ready | Fix pod 0 first |
| StatefulSet pod stuck `Terminating` | Node down or volume detach issue | `oc describe pod`; storage and node checks (Day 20) |
| Old PVCs left after tests | StatefulSet PVC retention | `oc delete pvc -l app=demo` |
| `exceeded quota` | Project quota reached | `oc describe quota`; clean up |
| Container fails with permission denied on port 80 or filesystem | Non-root random UID on OCP | Use non-root images and ports above 1024 (Day 07, SCC) |

## Debug flow for workloads

```bash
oc get deploy,sts,ds,job        # which controller?
oc get pods -o wide             # state, node, restarts
oc describe pod <name>          # Events, Last State, probe failures
oc logs <name> --previous       # logs of the crashed container
oc rollout status / history / undo
oc get events --sort-by=.lastTimestamp
```
