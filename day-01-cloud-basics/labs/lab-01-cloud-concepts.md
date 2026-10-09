# Lab 01: Cloud Concepts in Practice

**Time:** 30 to 45 minutes | **Cost:** Free

## Goal

Turn the theory into muscle memory: classify services, read a region map, and (optionally) explore a real cloud console.

---

## Exercise 1: Classify the service (10 min)

For each item, write IaaS, PaaS, or SaaS.

| Item | Your answer |
|---|---|
| A virtual machine you SSH into |  |
| Microsoft 365 |  |
| A managed Kubernetes or OpenShift service |  |
| Object storage bucket |  |
| A hosted MQ queue manager you only connect to |  |
| A Linux server in your own data center |  | 

Discuss: which would you pick for MQ, and what would you give up?

<details>
<summary>Answers</summary>

IaaS, SaaS, PaaS, IaaS (storage is infrastructure), SaaS, and the last one is not cloud at all (on-premises).

</details>

## Exercise 2: Shared responsibility (10 min)

You run MQ on OpenShift in a public cloud. Mark each task as **Provider (P)** or **You (Y)**:

| Task | P / Y |
|---|---|
| Replace a failed physical disk |  |
| Restrict who can create queues |  |
| Enable TLS on a channel |  |
| Patch the hypervisor |  |
| Upgrade the MQ version |  |
| Open a firewall rule for port 1414 |  |

<details>
<summary>Answers</summary>

P, Y, Y, P, Y, Y.

</details>

## Exercise 3: Explore your local environment (5 min)

Check the tools you will need in later days.

```bash
git --version
curl --version
# Optional now, required from Day 02:
podman --version || docker --version
```

Record what is missing in a notes file. You will install it on Day 02.

## Exercise 4: Region and latency (10 min)

Measure round-trip time from your location to different regions. Replace the URLs with endpoints from the provider you choose (many providers publish region ping endpoints).

```bash
curl -o /dev/null -s -w "Total: %{time_total}s\n" https://example.com
```

Questions:

1. Which region is closest to you?
2. Would you place both MQ replicas of a synchronous HA pair in different regions? Why or why not?

## Exercise 5 (optional): Explore a real console

Create a free account on any provider (IBM Cloud Lite, AWS Free Tier, Azure Free, or Google Cloud Free Tier).

1. Find the **region** selector.
2. Find the list of **availability zones** for your region.
3. Find the **IAM** section and list who has admin rights.
4. Open the **pricing calculator** and estimate the monthly cost of 3 small VMs.

> **Safety:** set a budget alert immediately and delete anything you create. Never commit credentials, access keys, or account IDs to this repo.

## Wrap-up

Write three sentences in your own words:

1. What is the difference between IaaS and PaaS?
2. Why would a bank choose hybrid cloud?
3. Why do we spread MQ replicas across availability zones?

Share your answers with the group.
