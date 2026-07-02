
---

### Part 1: Conceptual Understanding

**1. How does Kubernetes ensure high availability compared to traditional deployment?**

> **Answer:** In traditional deployments, if a server or application crashes, the website goes down completely until someone fixes it manually. Kubernetes prevents this by running multiple copies (replicas) of the app across different nodes. If a container or node fails, Kubernetes automatically detects it and creates a new one immediately using its self-healing feature, keeping the app online without any downtime for users.

**2. Explain the relationship between Pods, ReplicaSets, and Deployments.**

> **Answer:** These three components work in a hierarchy. A Pod is the smallest unit in Kubernetes that actually holds and runs our application container. A ReplicaSet sits behind the scenes to make sure the exact number of Pods we requested are always running. Finally, the Deployment is the top-level manager that controls the ReplicaSet. We use Deployments to easily update our app to a new version or roll it back if something goes wrong.

**3. Why are Services required in Kubernetes?**

> **Answer:** Pods in Kubernetes are temporary; they can die and get recreated at any time, and every time they do, their IP address changes. Because of this unstable networking, we cannot connect to Pods directly. A Service acts as a permanent gateway with a stable IP address and name for a group of Pods. It routes the incoming traffic and balances the load so that users or other parts of the app can always find and reach our application.

**4. Difference between ConfigMaps and Secrets with a practical example.**

> **Answer:** Both are used to store configuration settings outside of our application code, but they handle security differently. ConfigMaps store non-sensitive configuration settings in plain text, while Secrets are used to store private, sensitive information by encoding it in Base64 format so it is not visible to everyone.
> *Practical Example:* For a database connection, you would save the environment mode or database port (`APP_MODE=dev`, `DB_PORT=5432`) inside a ConfigMap, but the actual database password (`DB_PASSWORD=mysecurepass123`) would be safely stored inside a Secret.