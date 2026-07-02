# Module 11 — Kubernetes Application Deployment & Management

## Overview
This submission demonstrates deployment and management of a containerized Nginx application on a local Kubernetes cluster (Minikube), covering cluster verification, multi-resource deployment, configuration management, scaling, rolling updates, troubleshooting, and namespace isolation.

## Environment
- **Cluster:** Minikube (Docker driver)
- **Container Image:** `nginx:1.25` → updated to `nginx:1.27`
- **Tooling:** `kubectl`, `minikube`

## How to Reproduce

```bash
# 1. Start the cluster
minikube start --driver=docker

# 2. Verify cluster and system pods
kubectl get nodes
kubectl get pods -n kube-system

# 3. Deploy application resources
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# 4. Verify
kubectl get pods,deployment,svc

# 5. Access in browser
minikube service nginx-service --url

# 6. Scale and update
kubectl scale deployment nginx-deployment --replicas=4
kubectl set image deployment/nginx-deployment nginx=nginx:1.27
kubectl rollout status deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment

# 7. Namespace isolation
kubectl create namespace dev-env
kubectl apply -f deployment.yaml -n dev-env
kubectl apply -f service.yaml -n dev-env
```

## Part 1: Conceptual Understanding
*(Answered independently, without AI assistance, per assignment instructions — see attached written answers.)*

## Part 2: Cluster Setup & Verification — Observation
The `kube-system` namespace contains the core control-plane components: `coredns`, `etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `kube-proxy`, and (in Minikube) `storage-provisioner`. These are required for Kubernetes to manage itself — DNS resolution, cluster state storage, scheduling, and node networking. All pods showing `Running` status with `READY` at full capacity (e.g. `1/1`) confirm the cluster's control plane is healthy and operational.

## Part 3 & 4: Deployment, Service, ConfigMap, Secret
- Deployment runs 2 replicas of `nginx:1.25`, labeled `app: nginx-demo`.
- Service (`NodePort`, port `30080`) selects pods via the same label for traffic routing.
- ConfigMap injects `APP_MODE=dev`; Secret injects dummy `USERNAME`/`PASSWORD` — both mounted as environment variables via `envFrom`.
- Verified inside the container using `kubectl exec -it <pod> -- env`.

## Part 5: Scaling & Rolling Updates — Observation
Running `kubectl set image` triggers Kubernetes to create a new ReplicaSet and gradually replace old Pods with new ones in a rolling fashion, avoiding downtime. `kubectl rollout status` tracks this progress in real time. `kubectl rollout undo` reverses the process by scaling the previous ReplicaSet back up and the current one down, restoring the prior revision. `kubectl rollout history` lists all revisions for traceability.

## Part 6: Basic Troubleshooting — Observation
An intentional bad image tag caused pods to enter `ImagePullBackOff`/`ErrImagePull`, visible via `kubectl get pods`. `kubectl describe pod` confirmed the exact cause in the Events section (failed image pull). `kubectl logs` returned no output since the container never started, confirming the failure occurred at the image-pull stage rather than inside a running container. The issue was resolved by re-applying a valid image tag (`nginx:1.27`) and confirming recovery with `kubectl rollout status`.

## Part 7: Namespace Isolation
A new namespace (`dev-env`) was created and the same application deployed into it. Resources in `dev-env` are only visible with `kubectl get pods -n dev-env`, while `default` namespace resources remain separately accessible via `-n default` — confirming namespace-level isolation.

## Screenshots
