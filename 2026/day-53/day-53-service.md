# Day 53 – Kubernetes Services

## Overview

Today I learned how Kubernetes Services provide stable network access to Pods managed by Deployments.

Pods receive dynamic IP addresses, which can change when they restart or get replaced. Kubernetes Services solve this problem by providing a stable IP address, DNS name, and load balancing across matching Pods.

## Environment

- Kubernetes: v1.37.0
- Cluster: Kind (`day53-cluster`)
- Container runtime: containerd
- Application: NGINX 1.25
- Deployment: `web-app`
- Replicas: 3

## Task 1 – Deploy the Application

Created a Deployment using `app-deployment.yaml`.

Configuration:
- Deployment name: `web-app`
- Container image: `nginx:1.25`
- Replicas: 3
- Container port: 80
- Pod label: `app: web-app`

Commands:

```bash
kubectl apply -f app-deployment.yaml
kubectl get deployment web-app
kubectl get pods -o wide
```

**Result:** Successfully created three NGINX Pods and verified that all three became Ready.

## Task 2 – ClusterIP Service

Created `clusterip-service.yaml`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
```

Commands:

```bash
kubectl apply -f clusterip-service.yaml
kubectl get services
```

Observed ClusterIP:

`10.96.193.235`

**Key learning:**

- ClusterIP is the default Kubernetes Service type.
- It provides internal cluster communication.
- It creates a stable IP and DNS name.
- It distributes traffic across matching healthy Pods.

## Task 3 – Kubernetes DNS Discovery

Tested Service discovery from inside the Kubernetes cluster using a BusyBox test Pod.

Created `test-client.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-client
spec:
  containers:
    - name: test-client
      image: busybox:latest
      command: ["sh", "-c", "sleep 3600"]
  restartPolicy: Never
```

Commands:

```bash
kubectl apply -f test-client.yaml
kubectl exec -it test-client -- sh
```

Inside the Pod:

```sh
wget -qO- http://web-app-clusterip
wget -qO- http://web-app-clusterip.default.svc.cluster.local
nslookup web-app-clusterip
```

**Key learning:**

Kubernetes automatically creates DNS records for Services.

Full Service DNS format:

`<service-name>.<namespace>.svc.cluster.local`

Both short and fully qualified names can resolve to the same Service IP when used from the appropriate namespace.

The Service DNS name remains stable even when backend Pods change.

## Task 4 – NodePort Service

Created `nodeport-service.yaml`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Commands:

```bash
kubectl apply -f nodeport-service.yaml
kubectl get services
kubectl get nodes -o wide
```

Observed NodePort:

`30080`

**Key learning:**

- NodePort exposes an application on a port on each Kubernetes node.
- The default NodePort range is 30000–32767.
- External clients can access the Service through a reachable node IP and NodePort.
- Kind may require extra port mappings for direct host access.

### Connectivity verification

Used port forwarding to test the NGINX application from Windows:

```bash
kubectl port-forward service/web-app-nodeport 8080:80
```

Accessed:

`http://localhost:8080`

**Result:** Successfully verified NGINX access through port forwarding.

Note: Port forwarding verifies application access through the Service but does not prove direct NodePort access through the Kind node IP.

## Task 5 – LoadBalancer Service

Created `loadbalancer-service.yaml`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
```

Commands:

```bash
kubectl apply -f loadbalancer-service.yaml
kubectl get services
kubectl describe service web-app-loadbalancer
```

Observed configuration:

- Type: LoadBalancer
- ClusterIP: `10.96.197.71`
- Service port: `80`
- Automatically assigned NodePort: `32156`
- External IP: `<pending>`

**Why is External IP pending?**

The cluster runs locally using Kind. There is no cloud provider configured to provision an external load balancer.

This is expected in a standard local Kind environment.

In cloud environments such as AWS, Azure, or GCP, a LoadBalancer Service can provision an external load balancer through the cloud provider integration.

## Task 6 – Compare Kubernetes Service Types

| Feature | ClusterIP | NodePort | LoadBalancer |
|---|---|---|---|
| Internal access | Yes | Yes | Yes |
| External access | No | Through node IP and port | Through external load balancer |
| ClusterIP assigned | Yes | Yes | Yes |
| NodePort assigned | No | Yes | Yes |
| External IP | No | No | Cloud provider dependent |
| Common use | Internal services | Testing and development | Cloud application exposure |

### Service verification

Command:

```bash
kubectl get services -o wide
```

Observed results:

| Service | Type | ClusterIP | Port |
|---|---|---|---|
| web-app-clusterip | ClusterIP | 10.96.193.235 | 80 |
| web-app-nodeport | NodePort | 10.96.234.35 | 30080 |
| web-app-loadbalancer | LoadBalancer | 10.96.197.71 | 32156 |

The LoadBalancer Service also received a ClusterIP and NodePort automatically.

### EndpointSlice verification

Commands:

```bash
kubectl get endpointslices -l kubernetes.io/service-name=web-app-clusterip

kubectl get endpointslices -l kubernetes.io/service-name=web-app-nodeport

kubectl get endpointslices -l kubernetes.io/service-name=web-app-loadbalancer
```

EndpointSlices provide information about the Pod endpoints associated with a Service.

They help verify which backend Pods are available to receive traffic.

## Important Kubernetes Concepts Learned

1. Pods have dynamic IP addresses.
2. Deployments manage multiple application Pods.
3. Services provide stable network access to Pods.
4. Selectors connect Services to matching Pod labels.
5. ClusterIP supports internal communication.
6. NodePort exposes Services through node ports.
7. LoadBalancer supports external access through cloud load balancers.
8. Kubernetes DNS enables service discovery.
9. EndpointSlices show the backend endpoints of Services.
10. Port forwarding helps test applications running inside local clusters.

## Files Created

```text
day-53/
├── app-deployment.yaml
├── clusterip-service.yaml
├── nodeport-service.yaml
├── loadbalancer-service.yaml
├── test-client.yaml
└── day-53-services.md
```

## Screenshots to Attach

1. `kubectl get pods -o wide` – Three NGINX Pods running.
2. `kubectl get services -o wide` – ClusterIP, NodePort, and LoadBalancer Services.
3. NGINX welcome page at `http://localhost:8080` – Port-forwarding connectivity test.
4. `kubectl describe service web-app-loadbalancer` – LoadBalancer details.
5. EndpointSlice output – Backend Pod endpoints.

## Conclusion

Successfully deployed an NGINX application with three replicas and exposed it using ClusterIP, NodePort, and LoadBalancer Services.

Verified internal Service communication and tested application access using port forwarding.

This practical helped me understand Kubernetes networking, service discovery, stable endpoints, and how different Service types expose applications.

**Day 53 – Kubernetes Services completed.**

#90DaysOfDevOps #Kubernetes #DevOpsKaJosh #TrainWithShubham
