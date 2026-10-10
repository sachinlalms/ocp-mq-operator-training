# Lab 03: Explore the Cluster with `oc`

**Time:** 45 to 60 minutes | **Environment:** your team's OCP cluster (one project per person)

> **Safety:** never commit tokens, kubeconfig files, or real cluster URLs to this repo. Use placeholders such as `<API_URL>` in anything you share.
> Parts A to C work with normal user access. Part D needs cluster-admin; if you don't have it, your trainer will demo it.

## Part A: Connect and look around (10 min)

Get your login command from the OpenShift web console (top-right menu, **Copy login command**), or:

```bash
oc login --server=<API_URL> --token=<YOUR_TOKEN>
oc whoami
oc whoami --show-server
oc version
```

Create your own project (namespace). Use a unique name, for example with your name or initials:

```bash
oc new-project day03-<yourname>
oc project
```

Questions:
1. Which user are you?
2. What Kubernetes version is the cluster built on?

## Part B: Create a Pod and follow its life (15 min)

Create `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-ubi
  labels:
    app: hello
spec:
  containers:
    - name: main
      image: registry.access.redhat.com/ubi9/ubi-minimal
      command: ["sleep", "3600"]
```

```bash
oc apply -f pod.yaml
oc get pod hello-ubi -w          # watch it go Pending -> ContainerCreating -> Running (Ctrl+C to stop)
oc get pod hello-ubi -o wide     # which node?
oc describe pod hello-ubi        # read the Events section at the bottom
oc get events --sort-by=.lastTimestamp
```

In the Events, find the lines for **Scheduled** (scheduler), **Pulling / Pulled** (kubelet + CRI-O), **Created / Started** (kubelet).

Questions:
1. Which node did the scheduler choose?
2. Which event shows the image was pulled?
3. Map each event to the 8-step flow in the README.

Now look at the object itself:

```bash
oc get pod hello-ubi -o yaml
```

Find `spec` and `status`. Which parts did you write? Which were added by the system?

```bash
oc explain pod.spec.containers.image
oc exec -it hello-ubi -- sh -c 'ps aux; hostname'
oc logs hello-ubi
```

## Part C: See the reconcile loop (10 min)

A bare Pod is not recreated if it dies. A Deployment (Day 04 preview) has a controller that keeps the desired number of replicas.

```bash
oc create deployment web --image=registry.access.redhat.com/ubi9/httpd-24 --replicas=2
oc get pods -l app=web
```

Open a second terminal and run `oc get pods -l app=web -w`. In the first terminal:

```bash
oc delete pod <one-of-the-web-pods>
```

Watch a new pod appear. Then:

```bash
oc scale deployment web --replicas=3
oc get pods -l app=web
oc get deployment web -o yaml | grep -A3 "replicas"
```

Questions:
1. Who created the replacement pod: you, the scheduler, or a controller?
2. In this experiment, what was the desired state? The actual state?
3. How does this relate to what the MQ Operator will do?

Peek at the API calls `oc` makes:

```bash
oc get pods -v=6 2>&1 | head -20
oc api-resources | head -20
```

## Part D: Control plane tour (cluster-admin only, 10 min)

```bash
oc get nodes
oc get nodes -o wide
oc describe node <a-control-plane-node> | head -60
oc get pods -n openshift-kube-apiserver
oc get pods -n openshift-etcd
oc get pods -n openshift-kube-scheduler
oc get pods -n openshift-kube-controller-manager
oc get clusteroperators
```

Questions:
1. How many control plane nodes and worker nodes are there? Why three control plane nodes?
2. Find the etcd pods. How many are there?
3. What do the roles shown in `oc get nodes` mean?

If you do not have admin rights, ask your trainer to project the output.

## Clean up

```bash
oc delete project day03-<yourname>
```

## Wrap-up

Draw, from memory, the control plane and node components and the 8 steps of "create a Pod". Compare with a teammate.
