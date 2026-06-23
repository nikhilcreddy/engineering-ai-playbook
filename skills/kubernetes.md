# Skill: Kubernetes

> Running Spring Boot microservices reliably on Kubernetes with GitOps (ArgoCD).

## Best practices

- **Liveness ≠ readiness.** Readiness gates traffic (fails when dependencies are down); liveness restarts a wedged pod. Map to Actuator `/actuator/health/readiness` and `/liveness`.
- **Always set resource requests and limits.** Requests drive scheduling; limits cap blast radius.
- **Graceful shutdown.** Enable Spring `server.shutdown=graceful` and set `terminationGracePeriodSeconds` so in-flight requests drain.
- **Config via ConfigMaps/Secrets**, injected as env or mounted files — never baked into the image.
- **One process per container**, stateless where possible; state lives in PostgreSQL/queues.
- **PodDisruptionBudget + multiple replicas** to survive node drains and rollouts.
- **HorizontalPodAutoscaler** on meaningful signals (CPU and/or custom metrics).
- **GitOps with ArgoCD:** the cluster state is declared in git; changes flow through PRs, not `kubectl apply`.
- **Progressive delivery:** canary or blue-green with automated rollback on SLO breach.

## Common mistakes

- ❌ No resource requests/limits → noisy-neighbor and OOMKills.
- ❌ Same endpoint for liveness and readiness → restart storms when a dependency blips.
- ❌ No `preStop`/grace period → dropped requests on rollout.
- ❌ Secrets in images or plain ConfigMaps.
- ❌ Single replica for "stateless" services → downtime on every node event.
- ❌ `latest` image tags → non-reproducible, un-rollback-able deploys.
- ❌ Running as root / no securityContext.
- ❌ Manual `kubectl` changes that drift from git (defeats GitOps).

## Checklist

- [ ] Liveness and readiness probes distinct and correct.
- [ ] CPU/memory requests and limits set.
- [ ] `server.shutdown: graceful` + `terminationGracePeriodSeconds` + `preStop` configured.
- [ ] Secrets via Secret store; no secrets in image/ConfigMap.
- [ ] ≥ 2 replicas + PodDisruptionBudget.
- [ ] Immutable, version-pinned image tags (digest or semver).
- [ ] Non-root `securityContext`, read-only root FS where possible.
- [ ] Managed by ArgoCD; no manual drift.
- [ ] HPA configured for load.

## Example code

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders
spec:
  replicas: 3
  selector:
    matchLabels: { app: orders }
  template:
    metadata:
      labels: { app: orders }
    spec:
      terminationGracePeriodSeconds: 45
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
      containers:
        - name: orders
          image: registry.internal/orders:1.8.2     # pinned, immutable
          ports: [{ containerPort: 8080 }]
          resources:
            requests: { cpu: "250m", memory: "512Mi" }
            limits:   { cpu: "1",    memory: "768Mi" }
          envFrom:
            - secretRef: { name: orders-secrets }
            - configMapRef: { name: orders-config }
          readinessProbe:
            httpGet: { path: /actuator/health/readiness, port: 8080 }
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet: { path: /actuator/health/liveness, port: 8080 }
            initialDelaySeconds: 30
            periodSeconds: 10
          lifecycle:
            preStop:
              exec: { command: ["sh", "-c", "sleep 10"] }   # let LB deregister
```

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: orders-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels: { app: orders }
```

```yaml
# ArgoCD Application — declarative GitOps
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: orders
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://git.internal/platform/orders-deploy.git
    targetRevision: main
    path: k8s/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: orders
  syncPolicy:
    automated: { prune: true, selfHeal: true }   # auto-correct drift
```

## Recommended patterns

- **Kustomize overlays** per environment (base + dev/stage/prod).
- **Argo Rollouts** for canary/blue-green with metric-based analysis gates.
- **NetworkPolicies** for least-privilege pod-to-pod traffic.
- **External Secrets Operator** to sync from a vault into Kubernetes Secrets.
- Pairs with [skills/observability.md](observability.md) for probe/rollout health and [agents/sre-engineer.md](../agents/sre-engineer.md) for operability.
