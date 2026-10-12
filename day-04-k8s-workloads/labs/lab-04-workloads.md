# Lab 04: Workloads on OCP

**Time:** 60 to 75 minutes | **Environment:** your team's OCP cluster

The YAML files are in [`../yaml/`](../yaml/): `deployment-web.yaml` and `statefulset-demo.yaml`. Run the commands below from the `day-04-k8s-workloads/` folder so the `yaml/...` paths work.

> **Safety:** use your own project. Never commit tokens or real cluster URLs.

```bash
oc new-project day04-<yourname>
```

## Exercise 1: Deployment, scaling, and self-healing (10 min)

```bash
oc apply -f yaml/deployment-web.yaml
oc get deployment,rs,pods -l app=web
oc scale deployment web --replicas=5
oc get pods -l app=web
oc scale deployment web --replicas=3
```

Delete one pod and watch it be replaced:
```bash
oc delete pod <one-pod-name>
oc get pods -l app=web -w
```

Questions:
1. What is the relationship between the Deployment, ReplicaSet, and Pods? (`oc get rs`)
2. Why does the new pod have a different name and IP?

## Exercise 2: Rolling update and rollback (15 min)

Check your current image, then update to a deliberately **wrong** tag to simulate a bad release:

```bash
oc set image deployment/web httpd=registry.access.redhat.com/ubi9/httpd-24:does-not-exist
oc rollout status deployment/web --timeout=60s
oc get pods -l app=web
oc describe pod <a-failing-pod> | tail -15
```

Observe: new pods show `ErrImagePull` or `ImagePullBackOff`, but the old pods keep serving because the rolling update protects availability.

Roll back:
```bash
oc rollout history deployment/web
oc rollout undo deployment/web
oc rollout status deployment/web
oc get pods -l app=web
```

Questions:
1. Why did the application stay up during the bad rollout?
2. What does `maxUnavailable: 1, maxSurge: 1` mean?

## Exercise 3: Probes and resources (15 min)

```bash
oc describe deployment web | grep -A6 -i "limits\|readiness\|liveness"
```

Break the readiness probe by pointing it at a wrong port:
```bash
oc patch deployment web --type=json -p='[{"op":"replace","path":"/spec/template/spec/containers/0/readinessProbe/tcpSocket/port","value":9999}]'
oc get pods -l app=web -w
```
Pods show `0/1 Running` (not Ready). Check `oc describe pod` for `Readiness probe failed`. Restore with `oc rollout undo deployment/web`.

Questions:
1. The pod is `Running` but not `Ready`. What does this mean for traffic?
2. Would a failing readiness probe restart the container? What about liveness?

Optional: set a very low memory limit (for example `16Mi`) and observe `OOMKilled`:
```bash
oc set resources deployment web --limits=memory=16Mi
oc get pods -l app=web
oc describe pod <pod> | grep -i -A3 "last state"
oc rollout undo deployment/web
```

## Exercise 4: StatefulSet and stable identity (20 min)

Check your storage classes first:
```bash
oc get storageclass
```
If none is marked `(default)`, uncomment and set `storageClassName` in the YAML.

```bash
oc apply -f yaml/statefulset-demo.yaml
oc get pods -l app=demo -w      # note the order: demo-0, then demo-1, then demo-2
oc get pvc
```

Look at the data written by each pod:
```bash
oc exec demo-1 -- cat /data/log.txt
```

Now delete pod `demo-1`:
```bash
oc delete pod demo-1
oc get pods -l app=demo -w
oc exec demo-1 -- cat /data/log.txt
```

Observe: the pod comes back with the **same name** and the **same data** (the log file now has two lines).

Compare with a Deployment: names were random and there was no per-pod storage.

Scale down and check the PVCs:
```bash
oc scale statefulset demo --replicas=1
oc get pods -l app=demo
oc get pvc
```

Questions:
1. Which PVCs still exist after scaling down? Why does Kubernetes keep them?
2. What would happen to an MQ queue manager if its PVC were deleted?
3. Which part of this lab is the same behaviour the MQ Operator relies on?

## Exercise 5: Job and CronJob (10 min)

```bash
oc create job hello --image=registry.access.redhat.com/ubi9/ubi-minimal -- echo "Hello from a Job"
oc get job,pods
oc logs job/hello

oc create cronjob every-minute --image=registry.access.redhat.com/ubi9/ubi-minimal --schedule="*/1 * * * *" -- date
oc get cronjob
# wait 1-2 minutes
oc get jobs
oc delete cronjob every-minute
```

## Exercise 6: DaemonSet (demo only, admin) (5 min)

```bash
oc get daemonset -A | head -20
oc get pods -n openshift-dns -o wide
```
Observe one DNS pod per node.

## Clean up

```bash
oc delete project day04-<yourname>
```
(Deleting the project removes the PVCs too.)

## Wrap-up

Fill in the table from memory:

| Need | Workload |
|---|---|
| Stateless web app |  |
| MQ queue manager |  |
| Log collector on every node |  |
| Nightly backup |  |
