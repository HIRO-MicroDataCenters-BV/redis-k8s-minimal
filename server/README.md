# redis-k8s-minimal

Simple, production-ready Redis Helm chart — in-memory only, no Bitnami, no Redis Stack modules.  
Designed to be small, predictable, and easy to run on Minikube or cloud clusters.

## Features
- Pure Redis (official image) — no additional modules
- Uses an existing Kubernetes Secret for the Redis password
- Optional nodeSelector / tolerations / affinity
- In-memory only (no PVCs) but configurable to enable persistence later
- Reasonable CPU/memory requests & limits for small clusters
- Minimal values.yaml and clear defaults to get you up and running quickly

## Quickstart

1. Add the chart (local helm chart)
```bash
# from repo root
helm install my-redis ./chart/redis -n marketplace --create-namespace -f ./examples/values-minikube.yaml
