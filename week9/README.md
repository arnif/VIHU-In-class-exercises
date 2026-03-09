# Week 9: Orchestrating Applications with Kubernetes

## Objective

Learn how to orchestrate containerized applications using **Kubernetes** with a local cluster powered by **Kind** (Kubernetes in Docker). By the end, you'll understand Kubernetes core concepts — Pods, Deployments, Services, scaling, and self-healing — and be able to deploy, expose, and manage the Docker container you built in Week 5.

---

## Why Application Orchestration?

In Week 5 you containerized an application with Docker. But a single container running on one machine isn't enough for real-world systems. You need:

- **Deployment** — Getting your container running in any environment reliably
- **Scaling** — Running multiple copies to handle more traffic
- **Coordination** — Managing how containers communicate with each other
- **Self-healing** — Automatically restarting containers that crash

This is what **orchestration** solves. And Kubernetes is the industry standard for it.

---

## What is Kubernetes?

Kubernetes (often abbreviated **K8s**) is an open-source platform for automating deployment, scaling, and management of containerized applications.

| Feature | What It Does |
|---|---|
| Service discovery and load balancing | Routes traffic to healthy containers automatically |
| Automated rollouts and rollbacks | Deploy new versions gradually, roll back if something breaks |
| Self-healing | Restarts failed containers, replaces unresponsive ones |
| Scaling | Scale up or down with a single command |
| Secret and configuration management | Manage passwords and config separately from code |
| Storage orchestration | Attach storage systems automatically |

### What Kubernetes is NOT

- Not a traditional Platform as a Service (PaaS)
- Does not build your application or deploy source code
- Does not limit what languages or frameworks you can use
- Provides building blocks — you choose how to assemble them

### Kubernetes Architecture (Simplified)

```
┌─────────────────────────────────────────────────┐
│                   Cluster                        │
│                                                  │
│  ┌──────────────────┐   ┌──────────────────┐    │
│  │  Control Plane    │   │  Worker Node(s)   │    │
│  │                  │   │                  │    │
│  │  - API Server    │   │  - Pods          │    │
│  │  - Scheduler     │──▶│  - Containers    │    │
│  │  - Controller    │   │  - Kubelet       │    │
│  │  - etcd          │   │                  │    │
│  └──────────────────┘   └──────────────────┘    │
│                                                  │
└─────────────────────────────────────────────────┘
```

- **Cluster** — A set of machines (nodes) that run containerized applications
- **Control Plane** — Makes decisions about the cluster (scheduling, detecting failures)
- **Worker Node** — Runs your application containers inside Pods
- **Pod** — The smallest deployable unit — one or more containers running together

---

## Why Kind?

**Kind** (Kubernetes in Docker) runs a full Kubernetes cluster using Docker containers as "nodes". It's ideal for learning because:

| | Kind | Minikube | Cloud (EKS/GKE/AKS) |
|---|---|---|---|
| Setup | `brew install kind` | `brew install minikube` | Cloud account + CLI setup |
| Runs in | Docker containers | VM or Docker | Cloud VMs |
| Multi-node | Yes (via config) | Limited | Yes |
| Speed | Fast (seconds) | Slower (VM boot) | Depends on cloud |
| CI-friendly | Yes (used by K8s project itself) | Possible | Complex |
| Resource usage | Lightweight | Heavier (VM) | N/A (cloud) |

Kind is what the Kubernetes project itself uses for testing. If it's good enough for them, it's good enough for us.

---

## Prerequisites

- **Docker Desktop** installed and running (https://www.docker.com/products/docker-desktop)
- **Week 5 project** completed (your Dockerized Bun app)

---

## Project Structure (What You'll Build)

```
week-9-k8s/
├── kind-config.yaml       ← Kind cluster configuration
├── deployment.yaml        ← Kubernetes Deployment manifest
├── service.yaml           ← Kubernetes Service manifest
└── (your Week 5 app)      ← The Dockerfile and app code from Week 5
```

You won't build a new application — you'll deploy your existing Week 5 Docker image into a Kubernetes cluster.

---

## Part 1: Set Up the Tools

### Exercise 1: Install Kind and kubectl

1. Install Kind:

   **macOS (Homebrew):**
   ```bash
   brew install kind
   ```

   **Windows (Chocolatey):**
   ```bash
   choco install kind
   ```

   **Linux:**
   ```bash
   # For AMD64
   [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-amd64
   # For ARM64
   [ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-arm64
   chmod +x ./kind
   sudo mv ./kind /usr/local/bin/kind
   ```

2. Install kubectl (the Kubernetes command-line tool):

   **macOS (Homebrew):**
   ```bash
   brew install kubectl
   ```

   **Windows (Chocolatey):**
   ```bash
   choco install kubernetes-cli
   ```

   **Linux:** Follow the [official kubectl install guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).

3. Verify both tools are installed:
   ```bash
   kind version
   kubectl version --client
   ```

   You should see version numbers for both tools.

---

## Part 2: Create a Kubernetes Cluster

### Exercise 2: Create a Kind Cluster with Port Mapping

Since Kind runs Kubernetes inside Docker containers, we need to configure **port mapping** so we can access our application from the host machine (your browser).

1. Create a project directory:
   ```bash
   mkdir week-9-k8s && cd week-9-k8s
   ```

2. Create a Kind configuration file `kind-config.yaml`:

   ```yaml
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
   - role: control-plane
     extraPortMappings:
     - containerPort: 30080
       hostPort: 30080
   ```

   **What this does:**
   - Creates a single-node cluster (one control-plane node that also runs workloads)
   - Maps port `30080` from the Kind container to your host machine — this is how you'll access your app in the browser later

3. Create the cluster:
   ```bash
   kind create cluster --config kind-config.yaml --name week9
   ```

   This takes 30-60 seconds. Kind downloads a Kubernetes node image and bootstraps a full cluster inside a Docker container.

4. Verify the cluster is running:
   ```bash
   kubectl cluster-info --context kind-week9
   ```

   You should see the Kubernetes control plane address.

5. Check the nodes:
   ```bash
   kubectl get nodes
   ```

   You should see one node with status `Ready`.

---

## Part 3: Build and Load Your Docker Image

### Exercise 3: Build Your Image and Load It into Kind

Unlike a cloud Kubernetes cluster that pulls images from a registry (like Docker Hub), Kind runs inside Docker containers and doesn't share your local Docker images automatically. You need to explicitly **load** your image into the cluster.

1. Copy your Week 5 application files (including the `Dockerfile`) into the current directory, or navigate to your Week 5 project folder.

   If you don't have your Week 5 project handy, create a minimal app:

   Create `index.ts`:
   ```typescript
   const server = Bun.serve({
     port: 3000,
     fetch(req) {
       return new Response("Hello from Kubernetes!");
     },
   });

   console.log(`Server running at http://localhost:${server.port}`);
   ```

   Create `Dockerfile`:
   ```dockerfile
   FROM oven/bun:latest
   WORKDIR /app
   COPY . .
   EXPOSE 3000
   CMD ["bun", "run", "index.ts"]
   ```

2. Build the Docker image:
   ```bash
   docker build -t my-app:1.0 .
   ```

   **Important:** We use the tag `1.0` instead of `latest`. Kubernetes treats `latest` specially — it defaults to `imagePullPolicy: Always`, which tries to pull from a registry and fails since our image is local.

3. Load the image into the Kind cluster:
   ```bash
   kind load docker-image my-app:1.0 --name week9
   ```

   This copies the image from your local Docker daemon into the Kind node's container runtime.

4. Verify the image is available inside the cluster:
   ```bash
   docker exec -it week9-control-plane crictl images | grep my-app
   ```

   You should see `my-app` with tag `1.0` listed.

---

## Part 4: Deploy Your Application

### Exercise 4: Create a Deployment

A **Deployment** tells Kubernetes what container to run, how many copies (replicas), and how to update them.

1. Create `deployment.yaml`:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-app
           image: my-app:1.0
           imagePullPolicy: Never
           ports:
           - containerPort: 3000
   ```

   **Key fields explained:**

   | Field | What It Does |
   |---|---|
   | `replicas: 1` | Run one copy of the container |
   | `selector.matchLabels` | How the Deployment finds its Pods |
   | `template.metadata.labels` | Labels applied to each Pod |
   | `image: my-app:1.0` | The Docker image to run |
   | `imagePullPolicy: Never` | Don't try to pull from a registry — use the locally loaded image |
   | `containerPort: 3000` | The port your app listens on inside the container |

2. Apply the Deployment:
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. Check the Deployment status:
   ```bash
   kubectl get deployments
   ```

   You should see `my-app` with `1/1` ready.

4. Check the Pod:
   ```bash
   kubectl get pods
   ```

   You should see a Pod with a name like `my-app-xxxxx-xxxxx` with status `Running`.

   **What just happened?** You told Kubernetes: "I want one copy of `my-app:1.0` running at all times." Kubernetes created a Pod, pulled the image from the node's local store, and started the container. If the container crashes, Kubernetes will restart it automatically.

### Exercise 5: Explore Your Pod

1. View the logs from your Pod:
   ```bash
   kubectl logs -l app=my-app
   ```

   You should see your app's startup message (e.g., "Server running at http://localhost:3000").

2. Get detailed information about the Pod:
   ```bash
   kubectl describe pod -l app=my-app
   ```

   This shows events, status, IP address, and more. Useful for debugging.

3. Execute a command inside the Pod (like `docker exec`):
   ```bash
   kubectl exec -it $(kubectl get pod -l app=my-app -o jsonpath='{.items[0].metadata.name}') -- /bin/sh
   ```

   You're now inside the running container. Try:
   ```bash
   curl http://localhost:3000
   ```

   Type `exit` to leave.

---

## Part 5: Expose Your Application

### Exercise 6: Create a Service

Your app is running inside the cluster, but it's not accessible from your browser yet. A **Service** exposes Pods to network traffic.

1. Create `service.yaml`:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-app-service
   spec:
     type: NodePort
     selector:
       app: my-app
     ports:
     - port: 3000
       targetPort: 3000
       nodePort: 30080
   ```

   **How traffic flows:**

   ```
   Browser (localhost:30080)
       │
       ▼
   Kind container (port 30080 mapped from kind-config.yaml)
       │
       ▼
   NodePort Service (port 30080 → targetPort 3000)
       │
       ▼
   Pod (container listening on port 3000)
   ```

   | Field | What It Does |
   |---|---|
   | `type: NodePort` | Exposes the service on a static port on each node |
   | `selector: app: my-app` | Routes traffic to Pods with label `app: my-app` |
   | `port: 3000` | The port the Service listens on inside the cluster |
   | `targetPort: 3000` | The port on the Pod to forward traffic to |
   | `nodePort: 30080` | The external port on the node (must be 30000-32767) |

2. Apply the Service:
   ```bash
   kubectl apply -f service.yaml
   ```

3. Verify the Service:
   ```bash
   kubectl get services
   ```

   You should see `my-app-service` with type `NodePort` and port `3000:30080/TCP`.

4. Open your browser and go to:
   ```
   http://localhost:30080
   ```

   You should see your app's response. Your application is now running inside Kubernetes and accessible from your browser.

---

## Part 6: Scaling and Self-Healing

This is where Kubernetes really shines. You can scale your application and Kubernetes will automatically handle failures.

### Exercise 7: Scale Your Deployment

1. Scale to 3 replicas:
   ```bash
   kubectl scale deployment my-app --replicas=3
   ```

2. Watch the Pods come up:
   ```bash
   kubectl get pods -w
   ```

   Press `Ctrl+C` to stop watching once all 3 Pods show `Running`.

3. The Service automatically load-balances across all 3 Pods. Each request to `http://localhost:30080` can hit a different Pod.

4. Check the Deployment:
   ```bash
   kubectl get deployment my-app
   ```

   You should see `3/3` ready.

### Exercise 8: Test Self-Healing

Kubernetes automatically restarts failed containers and replaces deleted Pods.

1. List current Pods and pick one to delete:
   ```bash
   kubectl get pods
   ```

2. Delete one of the Pods (replace the name with an actual Pod name from the output above):
   ```bash
   kubectl delete pod <pod-name>
   ```

3. Immediately list Pods again:
   ```bash
   kubectl get pods
   ```

   You'll see Kubernetes has already started a **new Pod** to replace the deleted one. The Deployment controller ensures there are always 3 replicas running — that's the **desired state** model.

4. Scale back to 1 replica:
   ```bash
   kubectl scale deployment my-app --replicas=1
   ```

   Kubernetes terminates the extra Pods gracefully.

---

## Part 7: Declarative Updates

### Exercise 9: Update the Deployment Declaratively

So far you've used `kubectl scale` (imperative command). In production, you'd update the YAML file and re-apply it — this is the **declarative** approach, and it's what you should version control.

1. Edit `deployment.yaml` and change `replicas: 1` to `replicas: 2`:

   ```yaml
   spec:
     replicas: 2
   ```

2. Apply the updated file:
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. Verify:
   ```bash
   kubectl get pods
   ```

   You should see 2 Pods. The key insight: **the YAML file is the source of truth**. Anyone on your team can see exactly what's deployed by reading the YAML files in Git — the same principle as infrastructure as code.

---

## Part 8: Use kubectl Port-Forward (Alternative Access Method)

### Exercise 10: Forward a Pod's Port Directly

Sometimes you want to access a specific Pod without setting up a Service (useful for debugging).

1. Forward port 8080 on your machine to port 3000 on the Pod:
   ```bash
   kubectl port-forward deployment/my-app 8080:3000
   ```

2. In another terminal (or browser), access:
   ```
   http://localhost:8080
   ```

3. Press `Ctrl+C` to stop the port-forward.

   **When to use each approach:**

   | Method | Use Case |
   |---|---|
   | Service (NodePort) | Permanent access, load balancing across Pods |
   | `kubectl port-forward` | Quick debugging, accessing a specific Pod |

---

## Part 9: Clean Up

### Exercise 11: Delete Everything

1. Delete the Service and Deployment:
   ```bash
   kubectl delete -f service.yaml
   kubectl delete -f deployment.yaml
   ```

2. Delete the Kind cluster:
   ```bash
   kind delete cluster --name week9
   ```

3. Verify no clusters remain:
   ```bash
   kind get clusters
   ```

   The output should be empty (or not list `week9`).

---

## Challenge Exercises

### Challenge 1: Multi-Node Cluster

1. Create a new `kind-multi.yaml` configuration with 1 control-plane and 2 worker nodes:
   ```yaml
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
   - role: control-plane
   - role: worker
   - role: worker
   ```
2. Create the cluster: `kind create cluster --config kind-multi.yaml --name multi`
3. Deploy your app with 3 replicas
4. Run `kubectl get pods -o wide` to see which Pods land on which nodes
5. Delete a worker node's Docker container (`docker stop multi-worker`) and observe what Kubernetes does
6. Clean up: `kind delete cluster --name multi`

### Challenge 2: Rolling Updates

1. Modify your app to return a different response (e.g., "Hello from v2!")
2. Build a new image: `docker build -t my-app:2.0 .`
3. Load it into the cluster: `kind load docker-image my-app:2.0 --name week9`
4. Update `deployment.yaml` to use `image: my-app:2.0`
5. Apply the change and watch the rolling update:
   ```bash
   kubectl apply -f deployment.yaml
   kubectl rollout status deployment/my-app
   ```
6. Check the rollout history: `kubectl rollout history deployment/my-app`
7. Roll back to v1: `kubectl rollout undo deployment/my-app`

### Challenge 3: Add Environment Variables

1. Update your `deployment.yaml` to pass environment variables to the container:
   ```yaml
   containers:
   - name: my-app
     image: my-app:1.0
     imagePullPolicy: Never
     ports:
     - containerPort: 3000
     env:
     - name: APP_ENV
       value: "production"
     - name: LOG_LEVEL
       value: "info"
   ```
2. Update your app to read and display these environment variables
3. Apply the change and verify the variables are available inside the Pod

---

## Quick Reference

### Kind Commands

| Command | Description |
|---|---|
| `kind create cluster --name <name>` | Create a new cluster |
| `kind create cluster --config <file>` | Create a cluster with configuration |
| `kind get clusters` | List all Kind clusters |
| `kind load docker-image <image> --name <cluster>` | Load a Docker image into a cluster |
| `kind delete cluster --name <name>` | Delete a cluster |

### kubectl — Core Commands

| Command | Description |
|---|---|
| `kubectl get nodes` | List cluster nodes |
| `kubectl get pods` | List Pods |
| `kubectl get pods -w` | Watch Pods in real-time |
| `kubectl get pods -o wide` | List Pods with extra details (node, IP) |
| `kubectl get deployments` | List Deployments |
| `kubectl get services` | List Services |
| `kubectl get all` | List all resources |

### kubectl — Working with Resources

| Command | Description |
|---|---|
| `kubectl apply -f <file>` | Create or update resources from a YAML file |
| `kubectl delete -f <file>` | Delete resources defined in a YAML file |
| `kubectl describe <resource> <name>` | Show detailed info about a resource |
| `kubectl logs -l app=<label>` | View logs from Pods matching a label |
| `kubectl exec -it <pod> -- /bin/sh` | Open a shell inside a Pod |
| `kubectl port-forward <resource> <local>:<remote>` | Forward a port to your machine |

### kubectl — Scaling and Updates

| Command | Description |
|---|---|
| `kubectl scale deployment <name> --replicas=<n>` | Scale a Deployment |
| `kubectl rollout status deployment/<name>` | Watch a rolling update |
| `kubectl rollout history deployment/<name>` | View rollout history |
| `kubectl rollout undo deployment/<name>` | Roll back to previous version |

### Kubernetes Resource Types

| Resource | What It Is |
|---|---|
| **Pod** | Smallest unit — one or more containers running together |
| **Deployment** | Manages Pods — handles replicas, updates, rollbacks |
| **Service** | Exposes Pods to network traffic (internal or external) |
| **Node** | A machine (real or virtual) in the cluster |
| **Cluster** | The entire set of nodes managed by Kubernetes |

### Service Types

| Type | Access From | Use Case |
|---|---|---|
| `ClusterIP` | Inside the cluster only | Internal communication between services |
| `NodePort` | Outside the cluster (via node IP + port) | Development, simple external access |
| `LoadBalancer` | Outside the cluster (via cloud load balancer) | Production (cloud environments) |

---

## Next Steps

After completing these exercises, explore:
- **ConfigMaps and Secrets** — Manage configuration and sensitive data separately from your container images
- **Ingress** — Route external HTTP traffic to services based on URL paths or hostnames
- **Persistent Volumes** — Attach storage that survives Pod restarts
- **Helm** — Package manager for Kubernetes that simplifies deploying complex applications
- **Namespaces** — Isolate resources within a single cluster for different teams or environments
