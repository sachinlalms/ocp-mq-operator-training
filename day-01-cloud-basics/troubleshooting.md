# Day 01: Common Issues and Misconceptions

Day 01 is conceptual, so most problems are misunderstandings or account setup issues.

## Misconceptions

| Misconception | Reality |
|---|---|
| "Cloud means someone else's computer, so security is their job" | Shared responsibility: you own identity, data, and configuration |
| "PaaS and Kubernetes are the same" | Kubernetes is a building block; OpenShift packages it as a platform |
| "More regions always means more resilience" | Cross-region adds latency and complexity; AZs are the first step |
| "Cloud is always cheaper" | Cost depends on usage; idle resources still cost money |
| "Containers are lightweight VMs" | They share the host kernel (see Day 02) |

## Practical issues

| Symptom | Likely cause | Fix |
|---|---|---|
| Free-tier signup rejected | Card verification or region restrictions | Try another provider, or use the instructor's shared lab |
| Unexpected bill | Resources left running | Set budget alerts, delete resources after labs |
| Cannot reach a cloud endpoint | Corporate proxy or firewall | Configure proxy variables or request access |
| `command not found` for git or curl | Tool not installed | macOS: `brew install git curl` |
| Console shows no resources | Wrong region selected | Check the region dropdown |

## Habit to build now

Whenever something is "missing" in a cloud console, check three things first: **region, account/project, and permissions**. This habit will save hours on OpenShift and MQ later.
