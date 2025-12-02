# Redis Kubernetes Minimal

[![Version](https://img.shields.io/badge/version-0.1.0-blue.svg)](https://github.com/HIRO-MicroDataCenters-BV/redis-k8s-minimal)
[![Kubernetes](https://img.shields.io/badge/kubernetes-%3E%3D1.19-blue)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/helm-v3-blue)](https://helm.sh/)
[![Redis](https://img.shields.io/badge/redis-7--alpine-red)](https://hub.docker.com/_/redis)

A minimal, production-ready Redis deployment for Kubernetes using Helm. This chart provides a simple, lightweight Redis instance with authentication, resource management, and configurable deployment options.

## Features

- **Pure Redis**: Uses official Redis 7 Alpine image without additional modules
- **Security**: Built-in authentication using Kubernetes secrets
- **Lightweight**: Minimal footprint designed for small to medium clusters
- **Configurable**: Support for node selectors, tolerations, and affinity rules
- **Production Ready**: Includes proper health checks, resource limits, and monitoring
- **In-Memory**: Optimized for in-memory operations (persistence can be enabled)
- **Minikube Friendly**: Works perfectly on local development environments

## Prerequisites

- Kubernetes cluster (>= 1.19)
- Helm 3.x
- kubectl configured to connect to your cluster

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/HIRO-MicroDataCenters-BV/redis-k8s-minimal.git
cd redis-k8s-minimal
```

### 2. Create Redis Credentials Secret

Before deploying, create a Kubernetes secret with Redis credentials:

```bash
kubectl create namespace marketplace
kubectl create secret generic redis-credentials \
  --from-literal=username=default \
  --from-literal=password=your-secure-password \
  -n marketplace
```

### 3. Deploy Redis

```bash
helm install my-redis ./server/charts/redis-repository \
  -n marketplace \
  --create-namespace
```

### 4. Verify Deployment

```bash
kubectl get pods -n marketplace
kubectl logs -f deployment/redis-repository -n marketplace
```

## ⚙️ Configuration

### Default Values

The chart comes with sensible defaults configured in `server/charts/redis-repository/values.yaml`:

| Parameter | Default Value | Description |
|-----------|---------------|-------------|
| `replicaCount` | `1` | Number of Redis replicas |
| `image.repository` | `redis` | Redis Docker image repository |
| `image.tag` | `7-alpine` | Redis image tag |
| `service.type` | `ClusterIP` | Kubernetes service type |
| `service.port` | `6379` | Redis service port |
| `resources.requests.memory` | `512Mi` | Memory request |
| `resources.requests.cpu` | `250m` | CPU request |
| `resources.limits.memory` | `1Gi` | Memory limit |
| `resources.limits.cpu` | `500m` | CPU limit |
| `nodeSelector.role` | `marketplace` | Node selector for pod placement |

### Custom Configuration

Create a custom `values.yaml` file:

```yaml
# custom-values.yaml
replicaCount: 1

image:
  repository: redis
  tag: "7-alpine"
  pullPolicy: IfNotPresent

resources:
  requests:
    memory: 256Mi
    cpu: 100m
  limits:
    memory: 512Mi
    cpu: 200m

env:
  - name: REDIS_MAXMEMORY
    value: "256mb"
  - name: REDIS_MAXMEMORY_POLICY
    value: "allkeys-lru"

nodeSelector: {}  # Remove node selector constraint
```

Deploy with custom configuration:

```bash
helm install my-redis ./server/charts/redis-repository \
  -n marketplace \
  -f custom-values.yaml
```

## Architecture

### Components

1. **Deployment**: Manages Redis pod lifecycle
2. **Service**: Exposes Redis within the cluster
3. **ServiceAccount**: Provides pod identity and permissions
4. **ConfigMap**: Stores Redis configuration (optional)
5. **Secret**: Stores Redis authentication credentials

### Resource Management

- **Memory**: Configured with `REDIS_MAXMEMORY` and `REDIS_MAXMEMORY_POLICY`
- **CPU**: Reasonable requests and limits for small to medium workloads
- **Storage**: In-memory by default (no persistent volumes)

### Health Checks

- **Liveness Probe**: TCP socket check on port 6379
- **Readiness Probe**: Redis PING command with authentication
- **Startup Probe**: Redis PING command for initial startup verification

## Advanced Usage

### Enable Persistence

To enable Redis persistence, modify `values.yaml`:

```yaml
persistence:
  enabled: true
  size: 1Gi
  storageClass: standard
  accessMode: ReadWriteOnce
```

### Custom Node Placement

Configure node selectors, tolerations, and affinity:

```yaml
nodeSelector:
  disktype: ssd

tolerations:
  - key: "redis-dedicated"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app.kubernetes.io/name
            operator: In
            values:
            - redis-repository
        topologyKey: kubernetes.io/hostname
```

### Enable Ingress

To expose Redis externally (not recommended for production):

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
    - host: redis.example.com
      paths:
        - path: /
          pathType: Prefix
```

## Security Considerations

1. **Authentication**: Always use strong passwords in the `redis-credentials` secret
2. **Network Policies**: Implement Kubernetes network policies to restrict access
3. **TLS**: Consider enabling TLS for production deployments
4. **RBAC**: Review and customize ServiceAccount permissions
5. **Pod Security**: Configure appropriate security contexts

### Example Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-network-policy
  namespace: marketplace
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: redis-repository
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: application-namespace
    ports:
    - protocol: TCP
      port: 6379
```

## Monitoring and Troubleshooting

### Check Pod Status

```bash
kubectl get pods -n marketplace -l app.kubernetes.io/name=redis-repository
```

### View Logs

```bash
kubectl logs -f deployment/redis-repository -n marketplace
```

### Connect to Redis

```bash
# Get the Redis password
REDIS_PASSWORD=$(kubectl get secret redis-credentials -n marketplace -o jsonpath="{.data.password}" | base64 --decode)

# Port forward to Redis
kubectl port-forward svc/redis-repository 6379:6379 -n marketplace

# Connect using redis-cli
redis-cli -h localhost -p 6379 -a $REDIS_PASSWORD
```

### Performance Monitoring

Monitor Redis performance using:

```bash
# Redis INFO command
redis-cli -h localhost -p 6379 -a $REDIS_PASSWORD info

# Check memory usage
redis-cli -h localhost -p 6379 -a $REDIS_PASSWORD info memory

# Monitor real-time commands
redis-cli -h localhost -p 6379 -a $REDIS_PASSWORD monitor
```
## 📁 Repository Structure

```
├── LICENSE                          # Project license
├── README.md                        # This documentation
├── VERSION                          # Current version
├── server/
│   ├── README.md                   # Server-specific documentation
│   └── charts/
│       └── redis-repository/       # Helm chart directory
│           ├── Chart.yaml          # Chart metadata
│           ├── values.yaml         # Default configuration values
│           └── templates/          # Kubernetes manifests
│               ├── deployment.yaml # Redis deployment
│               ├── service.yaml    # Service definition
│               ├── serviceaccount.yaml # Service account
│               ├── ingress.yaml    # Ingress configuration
│               └── _helpers.tpl    # Template helpers
└── tools/
    ├── deploy_k8s.sh              # Deployment automation script
    ├── extract_openapi.py         # OpenAPI extraction utility
    ├── version.sh                 # Version management script
    └── client_generator/          # Client generation tools
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines

- Test changes on a local Minikube cluster
- Update documentation for configuration changes
- Follow Helm chart best practices
- Ensure backward compatibility when possible
