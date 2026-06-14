# SRE Homelab — Learning by Breaking Things

Hands-on SRE training lab: I deliberately break a local Kubernetes cluster, then investigate each failure down to the Linux/network mechanism behind it.

**Method**: for each mission I only get a scenario and success criteria — no commands, no tutorial. I struggle first (30-60 min minimum), check the solution only after, then answer interview-style questions out loud and write everything down here. Mistakes and dead ends are documented on purpose: the reasoning matters more than the result.

**Stack**: k3d (3-node cluster) · kube-prometheus-stack · Grafana · a microservices demo app as the victim.

Related project: [lgtm-helm-chart](https://github.com/Rysekk/lgtm-helm-chart) — my LGTM observability stack ported to Helm (Mission 9).

## Missions

| # | Mission | Topic | Status |
|---|---------|-------|--------|
| 1 | Le pod qui meurt en silence | Linux — OOM, cgroups | ✅ |
| 2 | Rapide mais lent | Linux — CPU throttling, CFS | 🔧 |
| 3 | Le serveur mystère | Linux — USE method, I/O | ⬜ |
| 4 | Tout le monde timeout | Network — DNS, NetworkPolicy | ⬜ |
| 5 | L'autopsie des connexions | Network — TCP states | ⬜ |
| 6 | Le chemin invisible | Network — Services, kube-proxy | ⬜ |
| 7 | L'app saine qu'on assassine | K8s — probes, CrashLoopBackOff | ⬜ |
| 8 | Le node qui craque | K8s — eviction, QoS | ⬜ |
| 9 | Reconstruire la stack | K8s — Helm, StatefulSets | ⬜ |
| 10 | La panne composée | All — blind debugging | ⬜ |
| 11 | Entretien blanc | All — verbalization | ⬜ |

Legend: ✅ done · 🔧 in progress · ⬜ not started

## Repo layout

- `missions/` — one folder per mission: investigation notes, interview answers, key command outputs
- `post-mortems/` — incident write-ups in professional format
