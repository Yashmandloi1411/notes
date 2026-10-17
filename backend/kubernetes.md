# <span style="color:#326CE5;">Kubernetes — Local Setup & Command Reference</span>

![Kubernetes](https://img.shields.io/badge/Kubernetes-Local%20Setup-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker Desktop](https://img.shields.io/badge/Docker-Desktop-1D63ED?style=for-the-badge&logo=docker&logoColor=white)
![MERN Stack](https://img.shields.io/badge/Examples-MERN%20Stack-1A6B4A?style=for-the-badge)
![Commands](https://img.shields.io/badge/Reference-kubectl-C84B1F?style=for-the-badge)

> <span style="color:#C84B1F; font-weight:700;">Key idea:</span> A GitHub-ready collection of Kubernetes learning notes and command cheat sheets. This guide is organized into a local setup section followed by a practical kubectl reference.

## <span style="color:#1A6B4A;">Original color palette</span>

The first guide uses a warm paper background, dark ink, and accent colors. GitHub controls the page theme and sanitizes many custom CSS styles, so the palette is represented here with badges and labels rather than unsupported page-wide CSS.

- Accent orange: ![orange](https://img.shields.io/badge/Accent-C84B1F-C84B1F?style=flat-square)
- Accent green: ![green](https://img.shields.io/badge/Accent-1A6B4A-1A6B4A?style=flat-square)
- Accent blue: ![blue](https://img.shields.io/badge/Accent-1B4587-1B4587?style=flat-square)
- Accent gold: ![gold](https://img.shields.io/badge/Accent-B8860B-B8860B?style=flat-square)

## <span style="color:#1B4587;">Contents</span>

1. [Kubernetes Local Setup — Complete Notes](#part-1-kubernetes-local-setup--complete-notes)
2. [Kubernetes Commands Cheat Sheet](#part-2-kubernetes-commands-cheat-sheet)

---

# <span style="color:#C84B1F;">Part 1: Kubernetes Local Setup — Complete Notes</span>

## <span style="color:#1B4587;">Overview</span>

- [What is Kubernetes?](#what-is-kubernetes)
- [Architecture](#architecture)
- [Pod](#pod)
- [Deployment](#deployment)
- [ReplicaSet](#replicaset)
- [Service](#service)
- [Ingress](#ingress)
- [Ingress Controller](#ingress-controller)
- [Enable Kubernetes](#enable-kubernetes-on-docker-desktop)
- [Writing YAML Files](#writing-yaml-files)
- [Labels & Selectors](#labels--selectors)
- [Deploy Your App](#deploy-your-app)
- [Debug Commands](#debug-commands)
- [Autoscaling (HPA)](#autoscaling-hpa)
- [Managing Secrets Locally](#managing-secrets-locally)

## <span style="color:#1A6B4A;">Kubernetes on Your Local Machine</span>

A complete guide to understanding and running Kubernetes locally using Docker Desktop. It covers the core components with examples from a real MERN stack project.

### <span style="color:#C84B1F;">What is Kubernetes?</span>

Kubernetes (K8s) is a container orchestration system. It manages how your Docker containers run, making sure the right number are running, restarting them when they crash, routing traffic between them, and scaling them up when load increases.

You describe your desired state in YAML files, and Kubernetes makes it happen.

#### Without vs With Kubernetes

| Situation          | Without Kubernetes                   | With Kubernetes                                   |
| ------------------ | ------------------------------------ | ------------------------------------------------- |
| Container crashes  | App goes down, manual restart needed | Kubernetes restarts it automatically              |
| Traffic spike      | App slows down or crashes            | HPA adds more containers automatically            |
| Multiple services  | Manual ports, complex networking     | Internal DNS; services talk by name               |
| Deploy new version | Downtime during update               | Rolling update with zero downtime                 |
| Multiple replicas  | Manage each container manually       | Set `replicas: 3` and Kubernetes handles the rest |

#### Key terms at a glance

| Term               | Simple meaning                                                                   |
| ------------------ | -------------------------------------------------------------------------------- |
| Pod                | Smallest unit. One or more containers running together.                          |
| Deployment         | Manages pods, keeps them running, handles updates and rollbacks.                 |
| Service            | Stable network endpoint that finds pods using labels and routes traffic to them. |
| Ingress            | HTTP routing rules: which domain/path goes to which service.                     |
| Ingress Controller | The actual pod (nginx) that reads Ingress rules and routes real traffic.         |
| Namespace          | Virtual cluster inside a cluster used to separate environments.                  |
| ConfigMap          | Stores non-secret config like environment variables separately from the image.   |
| Secret             | Stores sensitive config like passwords and tokens encoded in the cluster.        |

### <span style="color:#1B4587;">Architecture</span>

A Kubernetes cluster has two parts:

- The Control Plane: the brain that decides what runs where.
- Worker Nodes: the muscle where your pods actually run.

On Docker Desktop, both live on your local machine.

#### Full traffic flow

Browser / Client -> Ingress Controller (nginx pod) -> Ingress Resource -> Service -> Pod (Express container)

#### Cluster architecture diagram

```text
Kubernetes Cluster (Docker Desktop)
┌──────────────────────────────────────────────┐
│                  Master Node                 │
│  API Server                                  │
│  Scheduler                                   │
│  Controller Manager                          │
└──────────────────────────────────────────────┘
                     │
                     ▼
                Worker Node / Node
                ┌──────────────────────┐
                │  Pod (Express app)   │
                │  Container           │
                └──────────────────────┘
```

#### Project file structure

```text
your-project/
├── Backend/
│   ├── server.js
│   ├── package.json
│   └── Dockerfile
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── secret.yaml
└── .gitignore
```

### <span style="color:#1A6B4A;">Pod</span>

A Pod is a wrapper around one or more containers. All containers in a pod share the same network and storage. In practice, most pods contain a single container, which is your app.

You rarely create pods directly; a Deployment creates and manages them for you.

#### What a pod contains

| What         | Description                                                                                                                                         |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Container(s) | One or more Docker containers. They share the same localhost inside the pod.                                                                        |
| IP address   | Each pod gets its own IP inside the cluster. This changes every time a pod restarts, which is why you use a Service instead of the pod IP directly. |
| Labels       | Key-value tags like `app: express`. Services and Deployments use these to find the pod.                                                             |
| Resources    | CPU and memory limits and requests defined in the pod spec.                                                                                         |

#### Pod lifecycle

```text
Pending -> pod is scheduled, image is being pulled
    ↓
Running -> container started successfully
    ↓
Ready -> readiness probe passed, pod receives traffic ✅
    ↓
CrashLoopBackOff -> container keeps crashing, Kubernetes retries
    ↓
ImagePullBackOff -> cannot pull the Docker image
```

> You never create pods directly in production. Always use a Deployment. If a pod is created directly and crashes, nothing restarts it. A Deployment automatically replaces it.

#### Check pod status

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- sh
```

### <span style="color:#C84B1F;">Deployment</span>

A Deployment is a manager for your pods. You tell it which image to run, how many replicas you want, and what resources each pod needs. It ensures the desired state is always maintained. If a pod crashes, the Deployment creates a new one. If you update the image, it performs a rolling update with zero downtime.

#### Responsibilities of a Deployment

| Responsibility     | What it means                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------- |
| Replica management | Always keeps the exact number of pods running. If one crashes, a new one is created immediately.                 |
| Rolling update     | When you push a new image, Kubernetes replaces pods one by one. Old pods serve traffic until new ones are ready. |
| Rollback           | If a new version has a bug, you can revert to the previous working version.                                      |
| Scaling            | Change the replica count manually or let HPA do it automatically based on CPU.                                   |

#### Full `deployment.yaml` with explanation

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: express-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: express
  template:
    metadata:
      labels:
        app: express
        owner: ankur
    spec:
      containers:
        - name: express
          image: express-k8s:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
          resources:
            limits:
              memory: "128Mi"
              cpu: "500m"
            requests:
              memory: "64Mi"
              cpu: "250m"
```

#### Key fields explained

| Field                      | Value          | What it does                                                                 |
| -------------------------- | -------------- | ---------------------------------------------------------------------------- |
| `apiVersion`               | `apps/v1`      | Deployment belongs to the apps API group.                                    |
| `replicas`                 | `2`            | Always maintain 2 running pods.                                              |
| `selector.matchLabels`     | `app: express` | Deployment manages pods that have matching labels.                           |
| `template.metadata.labels` | `app: express` | Labels stamped on every pod created by this Deployment.                      |
| `imagePullPolicy`          | `IfNotPresent` | Use local image if available.                                                |
| `resources.requests`       | `cpu: 250m`    | CPU and memory reserved for the pod. HPA uses this to calculate utilization. |
| `resources.limits`         | `cpu: 500m`    | Maximum CPU/memory the pod can use.                                          |

#### CPU units explained

| Value   | Meaning               |
| ------- | --------------------- |
| `1000m` | 1 full CPU core       |
| `500m`  | Half a CPU core       |
| `250m`  | Quarter of a CPU core |

#### Useful Deployment commands

```bash
kubectl get deployments
kubectl rollout status deployment/express-deployment
kubectl rollout undo deployment/express-deployment
kubectl scale deployment express-deployment --replicas=5
kubectl rollout restart deployment/express-deployment
```

### <span style="color:#1B4587;">ReplicaSet</span>

A ReplicaSet is the component that guarantees N copies of your pod are always running. If a pod crashes, the ReplicaSet creates a replacement. If there are too many pods, it deletes extras.

You almost never create a ReplicaSet directly. A Deployment creates and manages a ReplicaSet for you, and also handles rolling updates and rollbacks.

#### Deployment → ReplicaSet → Pods relationship

```text
Deployment -> owns and manages a ReplicaSet
    ↓
ReplicaSet -> ensures replicas: 2 pods are always alive
    ↓
Pod 1 (running ✅) Pod 2 (running ✅)
    ↓ Pod 1 crashes
ReplicaSet detects: 1 pod running, desired is 2
    ↓
ReplicaSet creates Pod 3 automatically ✅
```

#### Why you don't write ReplicaSet YAML directly

| If you use          | What you get                                                                           |
| ------------------- | -------------------------------------------------------------------------------------- |
| Deployment          | ReplicaSet management, rolling updates, rollback history. Always use this in practice. |
| ReplicaSet directly | Pod count maintenance only. No rolling updates, no rollback. Not recommended.          |

> When you run `kubectl get replicasets`, you will see an auto-generated ReplicaSet like `express-deployment-7d9f8b6c4` — the suffix is a fingerprint of the pod template. When you update the image, a new ReplicaSet is created and the old one is scaled down to zero but kept for rollback.

#### Viewing ReplicaSets

```bash
kubectl get replicasets
kubectl describe replicaset <rs-name>

# After a rollout you will see TWO replicasets:
# express-deployment-7d9f8b6c4   2   2   2   (new — active)
# express-deployment-5c8a3d1b2   0   0   0   (old — kept for rollback)
```

#### How ReplicaSet selects pods

ReplicaSet uses label selectors — it looks for pods with matching labels and counts them. If you manually create a pod with the same labels, the ReplicaSet will count it and may delete one of its own pods to maintain the desired count. This is why labels must be unique per Deployment.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: express-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: express
  template:
    metadata:
      labels:
        app: express
    spec:
      containers:
        - name: express
          image: express-k8s:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
```

### <span style="color:#1A6B4A;">Service</span>

Pods are temporary — they crash and restart with new IP addresses. A Service gives you a stable endpoint that never changes. It finds pods using label selectors and load balances traffic across all matching pods automatically. When a pod is added or removed, the Service updates its list instantly.

#### How Service finds pods (Labels & Selectors)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: express-service
spec:
  selector:
    app: express
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

#### Three Service types

| Type           | Accessible from          | Use case                                                   |
| -------------- | ------------------------ | ---------------------------------------------------------- |
| `ClusterIP`    | Inside cluster only      | Pod-to-pod communication. Default type. Used with Ingress. |
| `NodePort`     | External via port number | Quick local testing. Access via `localhost:30001`.         |
| `LoadBalancer` | External via cloud LB    | Production on AWS/GCP. Creates a real load balancer.       |

#### `port` vs `targetPort`

| Field        | Value  | Meaning                                                 |
| ------------ | ------ | ------------------------------------------------------- |
| `port`       | `80`   | The port the Service exposes inside the cluster.        |
| `targetPort` | `3000` | The port your app is listening on inside the container. |

#### NodePort for local testing

```yaml
type: NodePort
ports:
  - port: 80
    targetPort: 3000
    nodePort: 30001
```

#### How Service load balances

```text
Service receives request (selector: app=express)
    ↓
Looks up Endpoints list (live pod IPs)
    ↓
Request 1 -> Pod 1 (192.168.1.10)
Request 2 -> Pod 2 (192.168.1.11)
Request 3 -> Pod 3 (192.168.1.12)
Request 4 -> Pod 1 (round robin) ✅
```

```bash
kubectl get services
kubectl get endpoints express-service
```

### <span style="color:#C84B1F;">Ingress</span>

An Ingress resource is a set of HTTP routing rules written in YAML. It says things like: `traffic for express.local/ should go to express-service`, or `traffic for express.local/auth should go to auth-service`.

The Ingress itself does nothing; it only works when an Ingress Controller is installed to read and act on these rules.

#### Full `ingress.yaml` with explanation

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: express-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: express.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: express-service
                port:
                  number: 80
```

#### Path-based routing — multiple services on one domain

```yaml
rules:
  - host: express.local
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: main-service
              port:
                number: 80
        - path: /auth
          pathType: Prefix
          backend:
            service:
              name: auth-service
              port:
                number: 80
```

#### Key fields explained

| Field              | Value           | What it does                                                       |
| ------------------ | --------------- | ------------------------------------------------------------------ |
| `ingressClassName` | `nginx`         | Tells Kubernetes which Ingress Controller should handle this rule. |
| `host`             | `express.local` | Only traffic with this Host header matches the rule.               |
| `pathType: Prefix` | `Prefix`        | Matches the path and everything after it.                          |
| `pathType: Exact`  | `Exact`         | Only matches that exact path.                                      |

#### For local development — skip the host

```yaml
rules:
  - http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: express-service
              port:
                number: 80
```

Access via `http://localhost` directly, without editing `/etc/hosts`.

### <span style="color:#1B4587;">Ingress Controller</span>

The Ingress Controller is a pod running nginx inside your cluster. It watches all Ingress resources and configures itself to route traffic according to those rules. Without the controller, your `ingress.yaml` is just a config file sitting there doing nothing.

#### Ingress vs Ingress Controller

| What                   | Ingress Resource                         | Ingress Controller                                 |
| ---------------------- | ---------------------------------------- | -------------------------------------------------- |
| What it is             | A YAML file with routing rules           | An actual nginx pod running in your cluster        |
| Created by             | You with `kubectl apply -f ingress.yaml` | You install once with `kubectl apply -f nginx-url` |
| Does it route traffic? | No, it only defines rules                | Yes, it reads rules and routes real HTTP traffic   |
| Namespace              | default                                  | `ingress-nginx`                                    |

#### Install nginx Ingress Controller

```bash
kubectl apply -f \
  https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/cloud/deploy.yaml
```

#### Verify controller is running

```bash
kubectl get pods -n ingress-nginx
```

Expected output:

```text
NAME                                        READY   STATUS
ingress-nginx-controller-xxxxxxxxx-xxxxx    1/1     Running ✅
```

#### Add `express.local` to `/etc/hosts` (Mac)

```bash
sudo nano /etc/hosts

# Add this line at the bottom:
127.0.0.1  express.local
```

Now `http://express.local` in your browser will route to your nginx Ingress Controller on localhost.

> `ingressClassName: nginx` in your `ingress.yaml` tells the controller which rules are meant for it. If you had Traefik installed, it would only pick up rules with `ingressClassName: traefik`.

### <span style="color:#1A6B4A;">Enable Kubernetes on Docker Desktop</span>

#### Step 1 — Enable in Docker Desktop settings

Open Docker Desktop → click the gear icon (Settings) → click Kubernetes in the left sidebar → check Enable Kubernetes → click Apply & Restart.

Wait 2–3 minutes. When the bottom status bar shows a green Kubernetes icon, it is ready.

#### Step 2 — Verify `kubectl` is working

```bash
kubectl version
kubectl get nodes
```

Expected output:

```text
NAME             STATUS   ROLES
docker-desktop   Ready    control-plane ✅
```

#### Step 3 — Check current context

```bash
kubectl config current-context
# Should return: docker-desktop

kubectl config get-contexts
# Lists all clusters kubectl knows about
```

> The `~/.kube/config` file stores all cluster connection info. When you later create an EKS cluster with `eksctl`, it adds a new entry here and switches the current context to EKS automatically. Switch back with `kubectl config use-context docker-desktop`.

### <span style="color:#C84B1F;">Writing YAML Files</span>

Every Kubernetes resource is defined in a YAML file. Each file must have `apiVersion`, `kind`, `metadata`, and `spec`. The `apiVersion` tells Kubernetes which API group the resource belongs to.

#### `apiVersion` — which API group

| Resource                           | `apiVersion`           | Why                                              |
| ---------------------------------- | ---------------------- | ------------------------------------------------ |
| Pod, Service, ConfigMap, Secret    | `v1`                   | Core group — fundamental Kubernetes resources.   |
| Deployment, StatefulSet, DaemonSet | `apps/v1`              | Apps group — higher-level workload resources.    |
| Ingress, NetworkPolicy             | `networking.k8s.io/v1` | Networking group — cluster networking resources. |
| HorizontalPodAutoscaler            | `autoscaling/v2`       | Autoscaling group — controls scaling behavior.   |

#### Putting multiple resources in one file

```yaml
apiVersion: apps/v1
kind: Deployment
# ... deployment config
---
apiVersion: v1
kind: Service
# ... service config
```

> Keep files separate as your project grows. One file per resource makes it easier to find and edit things. Use `kubectl apply -f k8s/` to apply the entire folder at once.

### <span style="color:#1B4587;">Labels & Selectors</span>

Labels are key-value tags on pods. Selectors are filters used by Deployments and Services to find pods with specific labels. This is how all three resources connect: the Deployment owns pods via selector, and the Service routes to pods via selector.

#### The three-way connection

```yaml
# deployment.yaml
selector:
  matchLabels:
    app: express

template:
  metadata:
    labels:
      app: express
```

```yaml
# service.yaml
selector:
  app: express
```

#### Selector matching rules

| Pod labels                   | Selector: `app=express`                       | Result      |
| ---------------------------- | --------------------------------------------- | ----------- |
| `app: express`               | Matches                                       | Selected ✅ |
| `app: express, env: prod`    | Matches (extra label ignored)                 | Selected ✅ |
| `app: backend`               | Does not match                                | Ignored ❌  |
| `app: express, env: staging` | Partially matches selector `app=express` only | Selected ✅ |

#### Multiple deployments using labels

```yaml
# staging-deployment.yaml
selector:
  matchLabels:
    app: express
    env: staging
```

```yaml
# production-deployment.yaml
selector:
  matchLabels:
    app: express
    env: production
```

Both deployments exist in the same cluster but never interfere with each other because their selectors are different.

### <span style="color:#1A6B4A;">Deploy Your App</span>

#### Step 1 — Build Docker image

```bash
cd Backend/
docker build -t express-k8s:latest .
```

#### Step 2 — Apply all YAML files

```bash
kubectl apply -f k8s/

# Or apply individually:
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
```

#### Step 3 — Verify everything is running

```bash
kubectl get pods
kubectl get services
kubectl get ingress
```

#### Step 4 — Access in browser

| Setup                                   | URL                      |
| --------------------------------------- | ------------------------ |
| No host in ingress                      | `http://localhost`       |
| `host: express.local` with `/etc/hosts` | `http://express.local`   |
| NodePort service                        | `http://localhost:30001` |

#### Update your app

```bash
# 1. Rebuild image after code changes
docker build -t express-k8s:latest .

# 2. Restart deployment to use new image
kubectl rollout restart deployment/express-deployment
```

### <span style="color:#C84B1F;">Debug Commands</span>

#### Common errors and what to do

| Error                     | Cause                                                                   | Debug command                                                |
| ------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------ |
| `ImagePullBackOff`        | Cannot pull Docker image. Usually local image not built or wrong name.  | `kubectl describe pod <name>`                                |
| `CrashLoopBackOff`        | Container keeps crashing. App has a startup error.                      | `kubectl logs <pod-name>`                                    |
| `503 Service Unavailable` | Ingress controller running but no pods found. Label mismatch.           | `kubectl get endpoints <service>`                            |
| `404 Not Found`           | nginx running but no matching ingress rule for that host/path.          | `kubectl describe ingress <name>`                            |
| `Pending (pod)`           | No node has enough CPU/memory to schedule the pod.                      | `kubectl describe pod <name>`                                |
| `ECONNREFUSED`            | Pod is calling another service using localhost instead of service name. | Use `http://service-name` instead of `http://localhost:PORT` |

#### Essential debug commands

```bash
# Check what is running
kubectl get pods
kubectl get services
kubectl get ingress
kubectl get all

# Deep inspect
kubectl describe pod <pod-name>
kubectl describe ingress <ingress-name>
kubectl describe service <service-name>

# Logs
kubectl logs <pod-name>
kubectl logs <pod-name> -f
kubectl logs <pod-name> --previous

# Network debug
kubectl get endpoints <service-name>
kubectl exec -it <pod-name> -- sh

# Inside pod: test another service
wget -qO- http://some-service/api/data
```

### <span style="color:#1B4587;">Autoscaling (HPA)</span>

The HorizontalPodAutoscaler (HPA) watches CPU usage across your pods and automatically scales the replica count up or down. When CPU exceeds the threshold, it adds pods. When traffic drops, it removes them.

It requires `metrics-server` to read CPU data. On Docker Desktop it is not pre-installed and must be added manually.

#### Step 0 — Install Metrics Server (required for HPA)

> HPA will not work without `metrics-server`. Running `kubectl top pods` will return `Metrics API not available`, and HPA will show `<unknown>/50%` until metrics-server is installed and running.

```bash
# Option A: official manifest with kubelet insecure TLS patch
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Docker Desktop uses a self-signed cert — patch metrics-server to skip TLS verify
kubectl patch deployment metrics-server -n kube-system \
  --type=json \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'

# Wait for it to be ready (~30 seconds)
kubectl rollout status deployment/metrics-server -n kube-system

# Verify it works
kubectl top nodes
kubectl top pods
```

Option B with Helm:

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm upgrade --install metrics-server metrics-server/metrics-server \
  --namespace kube-system \
  --set args={--kubelet-insecure-tls}
```

> `metrics-server` runs in the `kube-system` namespace on the master/control-plane node. Verify with `kubectl get pods -n kube-system | grep metrics`.

#### Create HPA via command

```bash
kubectl autoscale deployment express-deployment \
  --min=1 \
  --max=5 \
  --cpu-percent=50
```

#### Or as a YAML file (recommended)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: express-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: express-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

#### HPA scaling flow

```text
Traffic increases → CPU per pod goes above 50%
    ↓
HPA adds new pod (up to maxReplicas: 5)
    ↓
Requests spread across more pods
    ↓
CPU per pod drops — each pod handles less ✅
    ↓
Traffic drops → CPU goes low
    ↓
HPA removes extra pods (down to minReplicas: 1) ✅
```

> `resources.requests` must be set in your deployment for HPA to work. HPA calculates utilization as `current CPU / requested CPU × 100`.

```bash
kubectl get hpa
kubectl top pods
```

### <span style="color:#1A6B4A;">Managing Secrets Locally</span>

A Secret stores sensitive data like database passwords, API keys, and JWT secrets inside the Kubernetes cluster — encoded in base64. Pods then access these values as environment variables or file mounts.

> `base64` is not encryption. It is only encoding. Anyone with `kubectl get secret` access can decode it. Never commit secret YAML files to git.

#### Step 1 — Create base64 values

```bash
# Encode any string to base64
echo -n 'mypassword' | base64
# -> bXlwYXNzd29yZA==

echo -n 'mongodb://localhost:27017/mydb' | base64
# -> bW9uZ29kYjovL2xvY2FsaG9zdDoyNzAxNy9teWRi

# Decode to verify
echo 'bXlwYXNzd29yZA==' | base64 --decode
```

#### Step 2 — Write `secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: express-secret
type: Opaque
data:
  MONGO_URI: bW9uZ29kYjovL2xvY2FsaG9zdDoyNzAxNy9teWRi
  JWT_SECRET: bXlzdXBlcnNlY3JldGtleQ==
  DB_PASSWORD: bXlwYXNzd29yZA==
```

Alternative with `stringData` for local development:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: express-secret
type: Opaque
stringData:
  MONGO_URI: "mongodb://localhost:27017/mydb"
  JWT_SECRET: "mysupersecretkey"
  DB_PASSWORD: "mypassword"
```

#### Step 3 — Apply the Secret

```bash
kubectl apply -f k8s/secret.yaml

# Verify it was created
kubectl get secrets
kubectl describe secret express-secret

# Decode a specific value to verify
kubectl get secret express-secret -o jsonpath='{.data.MONGO_URI}' | base64 --decode
```

#### Step 4 — Inject Secret into your Deployment as env vars

```yaml
spec:
  containers:
    - name: express
      image: express-k8s:latest
      env:
        - name: MONGO_URI
          valueFrom:
            secretKeyRef:
              name: express-secret
              key: MONGO_URI
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: express-secret
              key: JWT_SECRET
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: express-secret
              key: DB_PASSWORD
```

#### Access in your Node.js/Express code

```js
const mongoUri = process.env.MONGO_URI;
const jwtSecret = process.env.JWT_SECRET;
const dbPassword = process.env.DB_PASSWORD;
```

#### Secret vs ConfigMap — when to use which

| Use case                  | ConfigMap             | Secret     |
| ------------------------- | --------------------- | ---------- |
| Database password         | ❌ Never              | ✅ Yes     |
| API base URL              | ✅ Yes                | Not needed |
| JWT secret key            | ❌ Never              | ✅ Yes     |
| `NODE_ENV=production`     | ✅ Yes                | Not needed |
| MongoDB connection string | ❌ If it has password | ✅ Yes     |
| Port number               | ✅ Yes                | Not needed |

#### Add `secret.yaml` to your project structure

```text
your-project/
├── Backend/
│   ├── server.js
│   ├── package.json
│   └── Dockerfile
└── k8s/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    └── secret.yaml   # add to .gitignore
```

> Always add `secret.yaml` to `.gitignore`. Committing secrets to git is a serious security risk.

#### Apply order matters

```bash
# 1. Secret must exist before pods that reference it start up
kubectl apply -f k8s/secret.yaml

# 2. Then apply deployment
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml

# Verify env vars were injected into the pod
kubectl exec -it <pod-name> -- sh
# Inside pod:
echo $MONGO_URI
echo $JWT_SECRET
```

---

# <span style="color:#C84B1F;">Part 2: Kubernetes Commands Cheat Sheet</span>

Each flag is listed on its own line so commands and options are easy to scan.

## <span style="color:#1B4587;">Basic Cluster & Context</span>

| Command                             | What it does                    | Important Flags | Flag Description |
| ----------------------------------- | ------------------------------- | --------------- | ---------------- |
| `kubectl version`                   | Shows client and server version | `--short`       | Short output     |
| `kubectl cluster-info`              | Displays cluster info           | —               | —                |
| `kubectl config get-contexts`       | Lists all contexts              | —               | —                |
| `kubectl config use-context <name>` | Switch context                  | —               | —                |
| `kubectl config current-context`    | Shows current context           | —               | —                |

## <span style="color:#1A6B4A;">Pods</span>

| Command                          | What it does           | Important Flags            | Flag Description  |
| -------------------------------- | ---------------------- | -------------------------- | ----------------- |
| `kubectl get pods`               | List pods              | `-o wide`                  | More details      |
| `kubectl get pods`               | List pods              | `-A`                       | All namespaces    |
| `kubectl describe pod <pod>`     | Detailed pod info      | —                          | —                 |
| `kubectl logs <pod>`             | Show logs              | `-f`                       | Follow logs       |
| `kubectl logs <pod>`             | Show logs              | `--tail=50`                | Last 50 lines     |
| `kubectl exec -it <pod> -- bash` | Access container shell | `-c <container>`           | Specify container |
| `kubectl delete pod <pod>`       | Delete pod             | `--grace-period=0 --force` | Force delete      |

## <span style="color:#C84B1F;">Deployments</span>

| Command                                          | What it does        | Important Flags | Flag Description |
| ------------------------------------------------ | ------------------- | --------------- | ---------------- |
| `kubectl get deployments`                        | List deployments    | `-A`            | All namespaces   |
| `kubectl create deployment <name> --image=<img>` | Create deployment   | `--replicas=3`  | Set replicas     |
| `kubectl scale deployment <name>`                | Scale deployment    | `--replicas=5`  | Number of pods   |
| `kubectl rollout status deployment <name>`       | Check rollout       | —               | —                |
| `kubectl rollout undo deployment <name>`         | Rollback deployment | —               | —                |

## <span style="color:#1B4587;">Services</span>

| Command                            | What it does    | Important Flags      | Flag Description |
| ---------------------------------- | --------------- | -------------------- | ---------------- |
| `kubectl get svc`                  | List services   | —                    | —                |
| `kubectl expose deployment <name>` | Create service  | `--type=NodePort`    | Service type     |
| `kubectl expose deployment <name>` | Create service  | `--port=80`          | Service port     |
| `kubectl expose deployment <name>` | Create service  | `--target-port=8080` | Container port   |
| `kubectl describe svc <name>`      | Service details | —                    | —                |

## <span style="color:#1A6B4A;">Namespaces</span>

| Command                    | What it does      | Important Flags | Flag Description |
| -------------------------- | ----------------- | --------------- | ---------------- |
| `kubectl get ns`           | List namespaces   | —               | —                |
| `kubectl create ns <name>` | Create namespace  | —               | —                |
| `kubectl delete ns <name>` | Delete namespace  | —               | —                |
| `kubectl get pods -n <ns>` | Pods in namespace | —               | —                |

## <span style="color:#C84B1F;">Nodes</span>

| Command                        | What it does            | Important Flags       | Flag Description |
| ------------------------------ | ----------------------- | --------------------- | ---------------- |
| `kubectl get nodes`            | List nodes              | `-o wide`             | Extra details    |
| `kubectl describe node <node>` | Node info               | —                     | —                |
| `kubectl cordon <node>`        | Mark node unschedulable | —                     | —                |
| `kubectl uncordon <node>`      | Enable scheduling       | —                     | —                |
| `kubectl drain <node>`         | Evacuate pods           | `--ignore-daemonsets` | Skip daemonsets  |

## <span style="color:#1B4587;">Config & Secrets</span>

| Command                                | What it does     | Important Flags            | Flag Description |
| -------------------------------------- | ---------------- | -------------------------- | ---------------- |
| `kubectl create configmap <name>`      | Create configmap | `--from-literal=key=value` | Inline data      |
| `kubectl get configmap`                | List configmaps  | —                          | —                |
| `kubectl create secret generic <name>` | Create secret    | `--from-literal=key=value` | Inline secret    |
| `kubectl get secrets`                  | List secrets     | —                          | —                |

## <span style="color:#1A6B4A;">Apply & Files</span>

| Command                        | What it does     | Important Flags | Flag Description |
| ------------------------------ | ---------------- | --------------- | ---------------- |
| `kubectl apply -f file.yaml`   | Apply config     | `-R`            | Recursive        |
| `kubectl delete -f file.yaml`  | Delete resources | —               | —                |
| `kubectl diff -f file.yaml`    | Show changes     | —               | —                |
| `kubectl replace -f file.yaml` | Replace resource | —               | —                |

## <span style="color:#C84B1F;">Debugging & Monitoring</span>

| Command              | What it does          | Important Flags                         | Flag Description        |
| -------------------- | --------------------- | --------------------------------------- | ----------------------- |
| `kubectl top pods`   | Resource usage (pods) | —                                       | Metrics server required |
| `kubectl top nodes`  | Node usage            | —                                       | —                       |
| `kubectl get events` | Cluster events        | `--sort-by=.metadata.creationTimestamp` | Sort events             |
| `kubectl describe`   | Debug any resource    | —                                       | Works for all           |

## Labels & Selectors

| Command                              | What it does    | Important Flags | Flag Description |
| ------------------------------------ | --------------- | --------------- | ---------------- |
| `kubectl label pods <pod> key=value` | Add label       | `--overwrite`   | Replace existing |
| `kubectl get pods -l key=value`      | Filter by label | —               | —                |

## <span style="color:#1A6B4A;">Port Forwarding</span>

| Command                                  | What it does    | Important Flags | Flag Description |
| ---------------------------------------- | --------------- | --------------- | ---------------- |
| `kubectl port-forward pod/<pod> 8080:80` | Forward port    | —               | Local:Remote     |
| `kubectl port-forward svc/<svc> 8080:80` | Forward service | —               | Local:Remote     |

## <span style="color:#C84B1F;">Quick Power Moves ⚡</span>

| Command                      | What it does            |
| ---------------------------- | ----------------------- |
| `kubectl get all`            | Everything in namespace |
| `kubectl delete all --all`   | Delete everything       |
| `kubectl api-resources`      | List resource types     |
| `kubectl explain <resource>` | Docs inside terminal    |

## <span style="color:#1B4587;">Pro Tips (Real Dev Stuff)</span>

- Use aliases:

```bash
alias k=kubectl
```

- Auto-completion:

```bash
source <(kubectl completion bash)
```

- Dry run before applying:

```bash
kubectl apply -f file.yaml --dry-run=client
```

---


