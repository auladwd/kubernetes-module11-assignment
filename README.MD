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

**1. How does Kubernetes ensure high availability compared to traditional deployment?**

> **Answer:** In traditional deployments, if a server or application crashes, the website goes down completely until someone fixes it manually. Kubernetes prevents this by running multiple copies (replicas) of the app across different nodes. If a container or node fails, Kubernetes automatically detects it and creates a new one immediately using its self-healing feature, keeping the app online without any downtime for users.

**2. Explain the relationship between Pods, ReplicaSets, and Deployments.**

> **Answer:** These three components work in a hierarchy. A Pod is the smallest unit in Kubernetes that actually holds and runs our application container. A ReplicaSet sits behind the scenes to make sure the exact number of Pods we requested are always running. Finally, the Deployment is the top-level manager that controls the ReplicaSet. We use Deployments to easily update our app to a new version or roll it back if something goes wrong.

**3. Why are Services required in Kubernetes?**

> **Answer:** Pods in Kubernetes are temporary; they can die and get recreated at any time, and every time they do, their IP address changes. Because of this unstable networking, we cannot connect to Pods directly. A Service acts as a permanent gateway with a stable IP address and name for a group of Pods. It routes the incoming traffic and balances the load so that users or other parts of the app can always find and reach our application.

**4. Difference between ConfigMaps and Secrets with a practical example.**

> **Answer:** Both are used to store configuration settings outside of our application code, but they handle security differently. ConfigMaps store non-sensitive configuration settings in plain text, while Secrets are used to store private, sensitive information by encoding it in Base64 format so it is not visible to everyone.
> *Practical Example:* For a database connection, you would save the environment mode or database port (`APP_MODE=dev`, `DB_PORT=5432`) inside a ConfigMap, but the actual database password (`DB_PASSWORD=mysecurepass123`) would be safely stored inside a Secret.

## Observations

**Part 2 — Cluster Setup & Verification (Observation)**

The `kube-system` namespace contains the core control-plane components:
`coredns`, `etcd`, `kube-apiserver`, `kube-controller-manager`,
`kube-scheduler`, `kube-proxy`, and (in Minikube) `storage-provisioner`. All of
these are required for Kubernetes to manage itself — DNS resolution, cluster
state storage, scheduling, and node networking. Every pod showing `Running`
status with `READY` as `1/1` (or `2/2` where applicable) confirms the cluster's
control plane is healthy and fully operational.

---

**Part 5 — Scaling & Rolling Updates (Observation)**

When `kubectl set image` is run, Kubernetes creates a new ReplicaSet and
gradually replaces old Pods with new ones in a rolling fashion — it does not
terminate all old Pods at once, so the application stays available throughout
the update (zero downtime). `kubectl rollout status` shows the progress in real
time, indicating how many new Pods are up and how many old ones are being
terminated. Running `kubectl rollout undo` reverses this process: Kubernetes
scales the previous ReplicaSet back up and scales the current one down,
effectively restoring the deployment to its prior revision.
`kubectl rollout history` lists all revision numbers, making it possible to
track exactly which change was rolled back.

---

**Part 6 — Basic Troubleshooting (Observation)**

The problem was first identified by checking `kubectl get pods`, where the
STATUS column showed `ImagePullBackOff`/`ErrImagePull`. Running
`kubectl describe pod <pod-name>` confirmed the root cause in the Events section
— the container runtime failed to pull the specified image because the tag did
not exist in the registry. `kubectl logs <pod-name>` returned little or no
output, since the container never actually started; this itself is a useful
diagnostic signal, confirming the failure occurred at the image-pull stage
rather than inside a running container (which would show application logs or a
crash trace instead). The issue was resolved by running `kubectl set image`
again with a valid, existing image tag (`nginx:1.27`), and
`kubectl rollout status` was used to confirm all Pods returned to a healthy
`Running` state.

## Screenshots
