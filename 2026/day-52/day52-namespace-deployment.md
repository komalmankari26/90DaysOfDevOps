
@'
# Day 52 – Kubernetes Namespaces and Deployments

## Objective
Learn Kubernetes namespaces, Deployments, self-healing, scaling, rolling updates, and rollbacks.

## 1. Explore Built-in Namespaces

Command:
```bash
kubectl get namespaces
kubectl get pods -n kube-system
```

Kubernetes includes built-in namespaces such as:
- `default`: Default location for resources when no namespace is specified.
- `kube-system`: Contains Kubernetes system components.
- `kube-public`: Intended for publicly readable cluster information.
- `kube-node-lease`: Stores node lease objects used for node heartbeat tracking.

## 2. Create Custom Namespaces

Commands:
```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace production
kubectl get namespaces
```

Namespaces organize resources within the same cluster. They help separate environments such as development, staging, and production.

## 3. Run Pods in Different Namespaces

Commands:
```bash
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
kubectl get pods
kubectl get pods -A
```

`kubectl get pods` displays Pods in the current namespace, normally `default`. The `-A` option lists Pods across all namespaces.

## 4. Deployment Manifest

File: `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.24
          ports:
            - containerPort: 80
```

### Explanation

- `apiVersion: apps/v1`: API version for Deployments.
- `kind: Deployment`: Defines a Deployment resource.
- `metadata.name`: Name of the Deployment.
- `metadata.namespace`: Namespace in which it runs.
- `labels`: Identifies the resource.
- `replicas: 3`: Requests three Pod replicas.
- `selector.matchLabels`: Selects the Pods managed by the Deployment.
- `template`: Blueprint used to create Pods.
- `containers`: Defines the container and its image.
- `containerPort: 80`: Documents the port exposed by the container.

Apply and verify:
```bash
kubectl apply -f nginx-deployment.yaml
kubectl get deployments -n dev
kubectl get pods -n dev
```

### Deployment status columns

- `READY`: Number of ready replicas compared with the desired number.
- `UP-TO-DATE`: Replicas updated to match the latest Deployment specification.
- `AVAILABLE`: Replicas available to serve requests.

## 5. Self-Healing

A standalone Pod does not have a Deployment controller to recreate it when deleted.

A Deployment maintains the requested number of replicas. If a managed Pod is deleted, Kubernetes creates a replacement Pod. The replacement usually has a different name.

Example:
```bash
kubectl get pods -n dev
kubectl delete pod <pod-name> -n dev
kubectl get pods -n dev
```

## 6. Scaling

### Imperative scaling

Scale up to five replicas:
```bash
kubectl scale deployment nginx-deployment --replicas=5 -n dev
kubectl get pods -n dev
```

Scale down to two replicas:
```bash
kubectl scale deployment nginx-deployment --replicas=2 -n dev
kubectl get deployments -n dev
kubectl get pods -n dev
```

When scaling down, Kubernetes terminates excess Deployment-managed Pods until the desired replica count is reached.

### Declarative scaling

Update `spec.replicas` in `nginx-deployment.yaml`, then apply it:
```bash
kubectl apply -f nginx-deployment.yaml
```

Declarative scaling stores the desired configuration in a manifest. Note that applying the original manifest with `replicas: 3` resets the desired count to three.

## 7. Rolling Update

Update the Nginx image:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
kubectl rollout status deployment/nginx-deployment -n dev
kubectl rollout history deployment/nginx-deployment -n dev
kubectl describe deployment nginx-deployment -n dev
```

A rolling update gradually replaces old Pods with new ones. Readiness checks and rollout settings help Kubernetes maintain availability during the update.

## 8. Rollback

Restore the previous Deployment revision:
```bash
kubectl rollout undo deployment/nginx-deployment -n dev
kubectl rollout status deployment/nginx-deployment -n dev
kubectl describe deployment nginx-deployment -n dev
```

The rollback restores the previous Deployment specification, including its earlier container image when that was the previous revision.

## 9. Important Learnings

- Namespaces organize resources inside a cluster.
- Deployments maintain the desired number of Pod replicas.
- Deployments provide self-healing for managed Pods.
- Scaling changes the desired replica count.
- Rolling updates gradually replace Pods with a new version.
- Rollbacks restore a previous Deployment revision.
- `kubectl get pods -A` lists Pods across namespaces.

## 10. Screenshots

Attach screenshots showing:
1. `kubectl get namespaces`
2. `kubectl get deployments -n dev`
3. `kubectl get pods -A`
4. Five replicas during scale-up and two replicas after scale-down.
5. Successful rolling update and rollback.

## 11. Cleanup

The following commands remove the Day 52 practice resources:
```bash
kubectl delete deployment nginx-deployment -n dev
kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging
kubectl delete namespace dev staging production
```

Verify cleanup:
```bash
kubectl get namespaces
kubectl get pods -A
```

**Note:** Cleanup deletes the practice namespaces and their resources. Capture the required screenshots before running cleanup.
'@ | Set-Content -Encoding utf8 day-52-namespaces-deployments.md
<img width="1920" height="1080" alt="Screenshot (513)" src="https://github.com/user-attachments/assets/08635772-61a5-4a3b-ae39-89df4186d69d" />
