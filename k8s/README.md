# Kubernetes Setup for `adaptive_tasks_k8s` (Minikube Local Images)

## Kubernetes Setup (Minikube - Local Development)

This project includes a full Kubernetes setup inside the `k8s/` folder for local development and deployment.

### Kubernetes Configuration Files

* `k8s/namespace.yaml`: Defines the isolated `task-manager` namespace for the application components.
* `k8s/configmap.yaml`: Stores non-confidential configuration data, such as environment variables for the frontend and backend.
* `k8s/secret.yaml`: Securely holds sensitive information, like database credentials and API keys.
* `k8s/postgres.yaml`: Configures the PostgreSQL database deployment and its associated persistent volume claim and service.
* `k8s/backend.yaml`: Contains the deployment and service configuration for the backend application.
* `k8s/frontend.yaml`: Contains the deployment and service configuration for the frontend application.
* `k8s/ingress.yaml`: Manages external access to the services in a cluster, routing HTTP traffic to the frontend and backend.
* `k8s/kustomization.yaml`: Manages the Kubernetes resources using Kustomize, allowing for easy application of all configurations together.
* `k8s/README.md`: Provides specific details and instructions related to the Kubernetes configurations and deployments.

### Running with Minikube

To set up and run the project locally using Minikube, follow these steps:

1. Start Minikube:
   ```bash
   minikube start
   ```

2. Enable the Ingress addon:
   ```bash
   minikube addons enable ingress
   ```

3. Point your shell to Minikube's Docker daemon to build images directly inside Minikube:
   ```bash
   eval "$(minikube docker-env)"
   ```

4. Build the backend Docker image:
   ```bash
   docker build -t taskmanager/backend:local ./backend
   ```

5. Build the frontend Docker image:
   ```bash
   docker build -t taskmanager/frontend:local ./frontend
   ```

6. Apply the Kubernetes configurations using Kustomize:
   ```bash
   kubectl apply -k k8s
   ```

7. Monitor the status of the pods (wait until they are in the `Running` state):
   ```bash
   kubectl get pods -n task-manager -w
   ```

---

This directory contains Kubernetes manifests for running `adaptive_tasks_k8s` on a local Minikube cluster.

## What is included

- `namespace.yaml`: Creates namespace `task-manager`
- `configmap.yaml`: Non-sensitive app settings
- `secret.yaml`: Sensitive app settings (replace defaults before use)
- `postgres.yaml`: PostgreSQL deployment, PVC, and service
- `backend.yaml`: Spring Boot backend deployment and service
- `frontend.yaml`: Next.js frontend deployment and service
- `ingress.yaml`: Host-based ingress (`taskmanager.local`)
- `kustomization.yaml`: Applies all manifests together

## 1) Start Minikube and enable ingress

```bash
minikube start
minikube addons enable ingress
```

## 2) Build Docker images inside Minikube Docker daemon

```bash
eval "$(minikube docker-env)"
docker build -t taskmanager/backend:local ./backend
docker build -t taskmanager/frontend:local ./frontend
```

## 3) Update secrets

Edit `k8s/secret.yaml` and set:

- `SPRING_DATASOURCE_PASSWORD`
- `JWT_SECRET` (base64-encoded 256-bit secret, e.g. `openssl rand -base64 32`)

## 4) Deploy everything

From the repository root (`adaptive_tasks_k8s`):

```bash
kubectl apply -k k8s
```

## 5) Wait for pods

```bash
kubectl get pods -n task-manager -w
```

## 6) Configure local hostname for ingress

Get Minikube IP:

```bash
minikube ip
```

Add an entry in `/etc/hosts` (replace `<MINIKUBE_IP>`):

```text
<MINIKUBE_IP> taskmanager.local
```

Then open:

- Frontend: `http://taskmanager.local`
- Backend API: `http://taskmanager.local/api`

## Useful troubleshooting

```bash
kubectl get all -n task-manager
kubectl describe pod -n task-manager <pod-name>
kubectl logs -n task-manager deploy/backend
kubectl logs -n task-manager deploy/frontend
kubectl logs -n task-manager deploy/postgres
```

## Notes

- Deployments use `imagePullPolicy: Never` so Kubernetes uses local Minikube images.
- PostgreSQL data is persisted with a PVC (`postgres-pvc`).
- If your backend does not expose `/actuator/health`, adjust probes in `backend.yaml`.
- Root project documentation is in `../README.md` and deployment details are in `../docs/DEPLOYMENT.md`.
