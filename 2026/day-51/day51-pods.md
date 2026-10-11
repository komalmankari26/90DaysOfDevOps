# Day 51 – Kubernetes Manifests and Your First Pods

## Objective
Learned to create Kubernetes Pod manifests from scratch, deploy Pods, inspect logs, validate YAML, and work with labels.

## 1. Anatomy of a Kubernetes Manifest

A Kubernetes manifest uses four required top-level fields:

- **apiVersion:** Specifies the API version. For Pods, use `v1`.
- **kind:** Defines the resource type, such as `Pod`.
- **metadata:** Contains the resource name and labels.
- **spec:** Describes the desired state, including containers, images, commands, and ports.

## 2. Nginx Pod

**File:** `nginx-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Apply and verify:

```powershell
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

Test the Nginx welcome page inside the container:

```powershell
kubectl exec -it nginx-pod -- /bin/bash
```

Inside the container:

```bash
curl localhost:80
exit
```

## 3. BusyBox Pod

**File:** `busybox-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
    - name: busybox
      image: busybox:latest
      command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

Apply and verify:

```powershell
kubectl apply -f busybox-pod.yaml
kubectl get pods
kubectl logs busybox-pod
```

Expected log output:

```text
Hello from BusyBox
```

The command prints a message and keeps the container running for one hour.

## 4. Third Pod — Apache HTTP Server

**File:** `httpd-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd
    environment: dev
    team: platform
spec:
  containers:
    - name: httpd
      image: httpd:latest
      ports:
        - containerPort: 80
```

Apply and verify:

```powershell
kubectl apply -f httpd-pod.yaml
kubectl get pods --show-labels
```

This manifest demonstrates how multiple labels can organize Kubernetes resources.

## 5. Imperative vs. Declarative

### Imperative approach

A command directly instructs Kubernetes to create a resource.

```powershell
kubectl run redis-pod --image=redis:latest
```

Inspect the generated Pod configuration:

```powershell
kubectl get pod redis-pod -o yaml
```

### Declarative approach

A YAML file describes the desired state, which Kubernetes applies.

```powershell
kubectl apply -f nginx-pod.yaml
```

Declarative configuration is useful for version control, repeatability, and collaboration.

### Generate YAML without creating a Pod

```powershell
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```

Save the generated YAML:

```powershell
kubectl run test-pod --image=nginx --dry-run=client -o yaml | Set-Content test-pod-generated.yaml
```

Both hand-written and generated manifests describe a resource using fields such as `apiVersion`, `kind`, `metadata`, and `spec`. Generated templates may include different defaults.

## 6. Validate Manifests

Client-side dry run:

```powershell
kubectl apply -f nginx-pod.yaml --dry-run=client
```

Server-side dry run:

```powershell
kubectl apply -f nginx-pod.yaml --dry-run=server
```

Client-side validation checks the manifest locally. Server-side dry run asks the Kubernetes API server to validate the request without persisting the resource.

A missing required container `image` field causes Kubernetes to reject the Pod manifest. Exact error wording can vary.

## 7. Labels and Filtering

Display Pods with labels:

```powershell
kubectl get pods --show-labels
```

Filter by application:

```powershell
kubectl get pods -l app=nginx
```

Filter by environment:

```powershell
kubectl get pods -l environment=dev
```

Filter by team:

```powershell
kubectl get pods -l team=platform
```

Add a label to the Nginx Pod:

```powershell
kubectl label pod nginx-pod environment=production
```

Remove the label:

```powershell
kubectl label pod nginx-pod environment-
```

Labels are key-value pairs used to organize resources and select matching Pods.

## 8. Useful Kubernetes Commands

```powershell
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl exec -it nginx-pod -- /bin/bash
kubectl get pod redis-pod -o yaml
kubectl get pods --show-labels
kubectl apply -f nginx-pod.yaml
kubectl delete -f nginx-pod.yaml
```

## 9. What Happens When a Standalone Pod Is Deleted?

A standalone Pod is not automatically recreated after deletion because no controller manages it.

In production, Deployments are commonly used to maintain the desired number of Pod replicas and replace failed or deleted Pods.

## 10. Screenshot Evidence

Capture and include these actual screenshots:

- [ ] `kubectl get pods -o wide` showing `nginx-pod`, `busybox-pod`, and `httpd-pod` in the Running state.
- [ ] `kubectl get pods --show-labels` showing the labels.
- [ ] Nginx welcome page returned by `curl localhost:80`.
- [ ] BusyBox logs displaying `Hello from BusyBox`.
- [ ] Optional: Server-side validation error for a manifest with a missing image.

## 11. Key Learnings

- Created three Pod manifests from scratch.
- Deployed Nginx, BusyBox, and Apache Pods.
- Used imperative and declarative resource creation.
- Generated YAML using `--dry-run=client`.
- Validated manifests with client-side and server-side dry runs.
- Inspected Pod details and logs.
- Applied and removed labels and filtered Pods using selectors.
- Learned why standalone Pods are not automatically recreated after deletion.

## 12. Completion Checklist

- [ ] `nginx-pod.yaml`
- [ ] `busybox-pod.yaml`
- [ ] `httpd-pod.yaml`
- [ ] `test-pod-generated.yaml`
- [ ] `day-51-pods.md`
- [ ] Screenshot evidence captured
- [ ] Changes committed and pushed to GitHub
<img width="1920" height="1080" alt="Screenshot (512)" src="https://github.com/user-attachments/assets/37097890-a6e0-45b0-92b1-bf17118b19cd" />
