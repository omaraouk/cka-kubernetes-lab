# Lab 001 — Kubernetes Deployment Lifecycle

## Objective

Practice managing Kubernetes Deployments using imperative `kubectl` commands, including:

- Creating Deployments from generated YAML
- Understanding ReplicaSet reconciliation
- Scaling workloads
- Performing rolling updates
- Rolling back Deployments
- Restarting Deployments

## Environment

| Component | Configuration |
|---|---|
| Kubernetes | Minikube |
| CLI | kubectl |
| Workload | nginx |
| Initial image | nginx:1.27 |
| Updated image | nginx:1.28 |
| Initial replicas | 3 |

## Exercise 1 — Create a Deployment

Generate a Deployment manifest using an imperative command:

```bash
kubectl create deployment nginx \
  --image=nginx:1.27 \
  --replicas=3 \
  --dry-run=client -o yaml > /tmp/lab01.yaml
```

Apply the manifest:

```bash
kubectl apply -f /tmp/lab01.yaml
```

Verify:

```bash
kubectl get deployment nginx
```

Result:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   3/3     3            3
```

**Key takeaway:** `--dry-run=client -o yaml` generates a Kubernetes manifest without creating the resource. The manifest can then be reviewed, modified, and applied.

## Exercise 2 — ReplicaSet Self-Healing

Delete one Pod managed by the Deployment:

```bash
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods -w
```

### Observations

- Kubernetes automatically created a replacement Pod.
- The replacement Pod had a different name.
- The ReplicaSet remained unchanged.

**Key takeaway:** Deployments manage ReplicaSets, while ReplicaSets maintain the desired number of Pods through reconciliation.

## Exercise 3 — Scaling Deployments

Scale the Deployment to five replicas:

```bash
kubectl scale deployment nginx --replicas=5
kubectl get deployment nginx
```

Result:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   5/5     5            5
```

Scale down to two replicas:

```bash
kubectl scale deployment nginx --replicas=2
kubectl get deployment nginx
```

Result:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   2/2     2            2
```

**Key takeaway:** Scaling changes the desired replica count of the existing ReplicaSet. It does not create a new ReplicaSet.

## Exercise 4 — Rolling Update

Update the container image:

```bash
kubectl set image deployment/nginx nginx=nginx:1.28
```

Monitor the rollout:

```bash
kubectl rollout status deployment nginx
```

Verify:

```bash
kubectl get deployment nginx -o wide
kubectl get rs
```

### Observations

| ReplicaSet | Desired | Ready |
|---|---:|---:|
| nginx-5fdc696c4c | 0 | 0 |
| nginx-7b95dc7d4d | 2 | 2 |

The Deployment successfully updated to `nginx:1.28`.

**Key takeaway:** Changing the Deployment
