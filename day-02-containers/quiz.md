# Day 02 Quiz

Answers at the bottom.

**1.** What do containers share with the host?
- A) The kernel  B) A full guest OS  C) A hypervisor  D) Nothing

**2.** Which Linux feature limits the CPU and memory a container may use?
- A) Namespaces  B) cgroups  C) SELinux  D) iptables

**3.** Which Linux feature limits what a container can see (processes, network, mounts)?
- A) cgroups  B) Namespaces  C) Layers  D) Tags

**4.** In `icr.io/ibm-messaging/mq:9.4.0.0-r1`, what is `9.4.0.0-r1`?
- A) Registry  B) Namespace  C) Tag  D) Digest

**5.** You delete a container that wrote data to its own filesystem (no volume). What happens to the data?
- A) Saved in the image  B) Lost  C) Moved to the registry  D) Kept on the host

**6.** What does `-p 9443:9443` do?
- A) Sets a password  B) Publishes a container port on the host  C) Pulls an image  D) Pins a version

**7.** Which runtime runs containers on OpenShift nodes?
- A) Docker Desktop  B) CRI-O  C) VirtualBox  D) Vagrant

**8.** True or false: an image is a running container.

**9.** Why do MQ containers need a volume mounted at `/mnt/mqm`?

**10.** A Kubernetes pod shows `OOMKilled`. Which Day 02 concept explains it?

---

## Answers

1. A
2. B
3. B
4. C
5. B
6. B
7. B
8. False. An image is a read-only template; a container is a running instance.
9. So queue manager data (queues, logs, messages) survives container restarts and replacement.
10. cgroups: the container exceeded its memory limit.
