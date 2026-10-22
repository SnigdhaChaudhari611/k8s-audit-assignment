- **How to open it:** Open `http://localhost/` on the machine running the kind cluster. The `/result` route currently returns HTTP 404, so it needs further investigation.

## Audit

| Concept | Status | Evidence | Why I used it in my app | Where to look |
|---|---|---|---|---|
| Deployment + ReplicaSet | ✅ | `./audit.sh`: 4 deployments detected. | Keeps application components managed and recoverable. | `k8s/voting-app.yaml` |
| Service | ✅ | `./audit.sh`: 4 services with endpoints. | Provides stable networking to pods. | `k8s/voting-app.yaml` |
| Namespace | ✅ | `./audit.sh`: `voting` namespace detected. | Separates app resources from system resources. | `k8s/voting-app.yaml` |
| Labels and selectors | ✅ | `./audit.sh`: 4 services have endpoints. | Connects services to the intended pods. | `k8s/voting-app.yaml` |
| Rolling update + rollback | ✅ | Changed the vote image tag to v2, waited for rollout, then ran `kubectl rollout undo` and waited again. v2 used the same image digest as v1. | Demonstrates Deployment revision and rollback mechanics. | `k8s/voting-app.yaml` |
| ConfigMap | ✅ | `./audit.sh`: 1 non-default ConfigMap. | Separates ordinary configuration from the container image. | `k8s/voting-app.yaml` |
| Secret | ✅ | `./audit.sh`: 1 Opaque Secret. Demo password only. | Supplies the database password separately from ordinary configuration. | `k8s/voting-app.yaml` |
| Requests and limits | ✅ | `./audit.sh`: 8 of 8 containers passed in the final 16/16 run. | Helps schedule workloads and constrain resource usage. | `k8s/voting-app.yaml`, `k8s/audit-extras.yaml` |
| Probes (liveness + readiness) | ✅ | `./audit.sh`: 8 of 8 containers passed in the final 16/16 run. | Checks container health and readiness for traffic. | `k8s/voting-app.yaml` |
| PVC | ✅ | `./audit.sh`: 1 bound PVC. | Persists PostgreSQL data. | `k8s/voting-app.yaml` |
| Ingress | ✅ | `./audit.sh`: 1 Ingress and 1 running controller; `curl -I http://localhost/` returned HTTP 200. `/result` returned HTTP 404. | Exposes the vote frontend through HTTP routing. | `k8s/voting-app.yaml`, `kind/README.md` |
| Multi-node kind cluster | ✅ | `kubectl get nodes` showed 3 nodes. | Demonstrates workload scheduling across multiple nodes. | `kind/kind-config.yaml` |
| HPA (stretch) | ✅ | `kubectl get hpa -n voting` showed CPU 1%/70% and 2 replicas. | Enables CPU-based scaling from 2 to 4 vote replicas. | `k8s/audit-extras.yaml` |
| RBAC + ServiceAccount (stretch) | ✅ | `./audit.sh`: 1 Role, 1 RoleBinding and 1 custom ServiceAccount. | Grants scoped pod-read permissions to the worker identity. | `k8s/voting-app.yaml`, `k8s/audit-extras.yaml` |
| CronJob (stretch) | ✅ | `kubectl get cronjob -n voting` showed a schedule of every 5 minutes. | Demonstrates a recurring scheduled task. | `k8s/audit-extras.yaml` |
| GitHub Actions deploying to kind (stretch) | ⚠️ | Workflow file exists and the audit script detects it; a successful GitHub Actions run has not yet been verified. | Automates cluster setup, deployment and audit checks. | `.github/workflows/kind-deploy.yml` |

## Score

- Must-have concepts with evidence: 12/12
- Stretch concepts detected by the local audit: 4/4
- Local audit result: 16/16

Note: The local audit detects the GitHub Actions workflow by its contents. This does not prove that the workflow succeeds on GitHub Actions.

## What was hard / what I would change

The main challenge was deploying the application components and dependencies together and validating their probes, storage, networking and scaling. I would improve the demo by investigating the `/result` route, using a genuinely changed image for the rollout test, and validating the workflow on GitHub Actions.
