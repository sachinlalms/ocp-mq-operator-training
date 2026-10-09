# Day 01: Cloud Basics

**Phase 1: Cloud & Container Foundations** | Duration: ~90 minutes | Level: Beginner

## Objectives

By the end of this day you will be able to:

- Explain what "cloud" means and why teams adopt it
- Distinguish IaaS, PaaS, and SaaS and give an example of each
- Compare public, private, and hybrid cloud
- Describe regions, availability zones, and why they matter for MQ
- Explain the shared responsibility model
- Explain why containers and Kubernetes are the next step (bridge to Day 02)

## Prerequisites

- A laptop with a terminal and a browser
- A free cloud account is optional (needed only for the optional lab step)

---

## 1. What is cloud computing?

Cloud computing is on-demand access to computing resources (servers, storage, networking, databases, platforms) over a network, usually paid for by usage, and provisioned in minutes through a console, CLI, or API.

| Traditional data center | Cloud |
|---|---|
| Buy hardware, wait weeks | Provision in minutes |
| Capacity planned for peak load | Scale up and down on demand |
| Large upfront cost (CapEx) | Pay as you go (OpEx) |
| You manage everything | Provider manages part of the stack |

**Five essential characteristics** (NIST definition): on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service.

## 2. Service models: IaaS, PaaS, SaaS

Think of it as how much of the stack **you** manage.

| Layer | On-premises | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Application | You | You | You | Provider |
| Data | You | You | You | Provider |
| Runtime / middleware | You | You | Provider | Provider |
| Operating system | You | You | Provider | Provider |
| Virtualization / servers | You | Provider | Provider | Provider |
| Storage / networking | You | Provider | Provider | Provider |

| Model | Examples | MQ relevance |
|---|---|---|
| **IaaS** | AWS EC2, Azure VMs, IBM Cloud VPC | Install MQ yourself on a VM |
| **PaaS** | OpenShift, Cloud Foundry, App Engine | Run MQ as a container via the MQ Operator |
| **SaaS** | Gmail, Salesforce, IBM MQ as a Service | Consume MQ with no infrastructure work |

> OpenShift sits between IaaS and PaaS. It is a Kubernetes **platform** you can run on any IaaS or on-premises. We start on it in Phase 2.

## 3. Deployment models

| Model | Description | Typical use |
|---|---|---|
| **Public cloud** | Shared infrastructure owned by a provider (AWS, Azure, GCP, IBM Cloud) | Elastic workloads, fast start |
| **Private cloud** | Cloud-style infrastructure dedicated to one organization | Regulated data, strict control |
| **Hybrid cloud** | Private and public connected and managed together | Most enterprises, including most MQ estates |
| **Multicloud** | More than one public provider | Avoid lock-in, best-of-breed |

## 4. Regions and availability zones

- **Region:** a geographic area containing multiple data centers (e.g. `ap-south-1` Mumbai).
- **Availability Zone (AZ):** an isolated data center (or group) within a region, with independent power and networking.

Why this matters for MQ and OpenShift:

- OpenShift worker nodes are spread across AZs for resilience.
- MQ Native HA runs three replicas of a queue manager, ideally one per AZ, so a zone outage does not stop messaging.
- Network latency between regions is too high for synchronous messaging replication.
- Data residency laws may restrict which region you may use.

## 5. Shared responsibility model

The provider secures **the cloud**. You secure **what you put in the cloud**.

| Provider is responsible for | You are responsible for |
|---|---|
| Physical data centers | Identity and access management |
| Hardware and hypervisor | Data classification and encryption choices |
| Core network | Application and middleware configuration |
| Managed-service internals | Patching what you manage (OS, MQ version) |
| | Network rules (security groups, firewalls) |

Example: in OpenShift on a public cloud, the provider runs the hardware, Red Hat/you run the cluster, and **you** configure MQ TLS, users, and queue permissions.

## 6. Key cloud-native concepts (preview)

| Concept | One-line meaning | Covered in |
|---|---|---|
| Containers | Package an app with its dependencies | Day 02 |
| Orchestration | Automatically run and heal containers at scale | Day 03 |
| Declarative config | Describe the desired state in YAML, the system makes it so | Day 04 onward |
| Immutable infrastructure | Replace, don't patch in place | Day 06 |
| Operators | Encode operational knowledge in software | Day 11 |

## 7. Why this matters for MQ

Traditional MQ ran on dedicated servers managed by hand. Cloud-native MQ is:

- Described as a YAML `QueueManager` resource
- Created, healed, and upgraded by the **MQ Operator**
- Stored on persistent volumes
- Exposed through Routes and Services

You cannot troubleshoot that stack without knowing the layers beneath it, which is exactly the path this series follows.

## Key Takeaways

- Cloud = on-demand resources, pay per use, API-driven
- IaaS gives you VMs, PaaS gives you a platform, SaaS gives you an app
- Hybrid cloud is the enterprise reality
- Spread workloads across availability zones for resilience
- You always own your data, identity, and configuration

## Hands-on

Go to [labs/lab-01-cloud-concepts.md](labs/lab-01-cloud-concepts.md).

## Review

- [Quiz](quiz.md)
- [Common issues](troubleshooting.md)

## Further reading

- NIST SP 800-145, *The NIST Definition of Cloud Computing*
- Your cloud provider's shared responsibility documentation

---

**Next:** [Day 02: Containers](../day-02-containers/)
