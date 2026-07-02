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
