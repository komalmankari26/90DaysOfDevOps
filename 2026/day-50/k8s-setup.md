# Day 50 – Kubernetes Architecture and Cluster Setup

## Objective

Set up a local Kubernetes cluster using kind, understand Kubernetes architecture, and explore cluster components using `kubectl`.

## 1. Kubernetes History

Kubernetes is an open-source container orchestration platform originally developed at Google and later donated to the Cloud Native Computing Foundation (CNCF). It was inspired by Google's internal Borg system for managing large-scale workloads. Kubernetes automates container deployment, scaling, networking, and recovery across a cluster of machines. The name Kubernetes comes from Greek and means "helmsman" or "pilot."

## 2. Kubernetes Architecture

```text
                  KUBERNETES CLUSTER
                          |
             +------------+------------+
             |                         |
        CONTROL PLANE              WORKER NODE
             |                         |
        +----+------+             +----+------+
        |    |      |             |    |      |
       API  etcd  Scheduler     kubelet kube-proxy
      Server                       |
        |                    Container Runtime
   Controller Manager              |
             |                     Pod
             +-------------------------+
```

### Control Plane Components

- **API Server:** Entry point for Kubernetes API requests.
- **etcd:** Stores cluster state and configuration data.
- **Scheduler:** Selects a suitable node for newly created Pods.
- **Controller Manager:** Runs controllers that help maintain the desired state.

### Node Components

- **kubelet:** Ensures Pods assigned to the node are running as specified.
- **kube-proxy:** Implements Kubernetes Service networking rules when used by the cluster network implementation.
- **Container Runtime:** Runs containers, for example containerd.

### Request Flow: `kubectl apply -f pod.yaml`

1. kubectl sends the request to the API Server.
2. The API Server validates the request and persists the desired state in etcd.
3. The Scheduler selects a suitable node for the unscheduled Pod.
4. The kubelet on the selected node observes the Pod assignment.
5. The container runtime downloads images if necessary and starts containers.
6. Kubernetes components continue monitoring and reconciling the desired state.

### Failure Scenarios

**If the API Server goes down:** New API requests and cluster changes generally cannot be processed. Existing containers may continue running, but cluster management and recovery capabilities can be impaired.

**If a worker node goes down:** Workloads on that node may become unavailable. Controllers can create replacement Pods on healthy nodes when the workload is managed appropriately and resources are available.

## 3. Local Cluster Setup

- **Tool chosen:** kind (Kubernetes in Docker).
- **Reason:** It runs Kubernetes nodes as Docker containers and is suitable for local practice.
- **Cluster name:** `devops-cluster`
- **kubectl client version:** v1.36.1
- **Kustomize version:** v5.8.1
- **Current context:** `kind-devops-cluster`

### Commands Used

```powershell
kubectl version --client
kind version
kind create cluster --name devops-cluster
kubectl cluster-info
kubectl get nodes
```

## 4. Exploring the Cluster

```powershell
kubectl get nodes
kubectl describe node devops-cluster-control-plane
kubectl get namespaces
kubectl get pods -A
kubectl get pods -n kube-system
kubectl config current-context
kubectl config get-contexts
kubectl config view
```

### Important System Pods

| Component | Purpose |
|---|---|
| kube-apiserver | Processes Kubernetes API requests |
| etcd | Stores cluster state |
| kube-scheduler | Schedules Pods onto nodes |
| kube-controller-manager | Runs control loops |
| coredns | Provides DNS-based service discovery |
| kube-proxy | Manages Service networking rules |
| kindnet | Provides Pod networking in the kind cluster |
| local-path-provisioner | Provides local persistent storage provisioning |

The kubelet and container runtime are node-level components, not separate Pods in the `kube-system` namespace.

## 5. Cluster Lifecycle and Kubeconfig

### Useful Commands

```powershell
kind get clusters
kubectl config current-context
kubectl config get-contexts
kubectl get nodes
```

### Kubeconfig

Kubeconfig stores cluster connection details, contexts, and authentication information for kubectl. On Windows, the default location is:

`C:\Users\komsh\.kube\config`

### Cluster Recreation

```powershell
kind delete cluster --name devops-cluster
kind create cluster --name devops-cluster
kubectl get nodes
```

**Note:** Cluster deletion is destructive to workloads and data inside the local cluster. Perform this exercise only after capturing the required screenshots and confirming that no practice workloads need to be retained.

## 6. Screenshots

Add screenshots of your actual terminal output to document:

1. `kubectl get nodes` showing the node in `Ready` state.
2. `kubectl get pods -n kube-system` showing the system Pods running.

## 7. Outcome

Successfully configured kubectl to use the local kind cluster, inspected the Kubernetes system components, and practiced commands for cluster information and context management.<img width="1920" height="1080" alt="Screenshot (511)" src="https://github.com/user-attachments/assets/5de210c6-beb3-4678-bd25-e84ee0c523e9" />
