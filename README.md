# Testkube Sample Application

A sample 3-tier application to run tests on.
The application is composed of a React frontend, NodeJs backend and a PostgreSQL database.

It can be used to showcase Testkube tests workflows.

![Testkube Sample Application](./docs/images/app.png)

## Running the application

You can either use docker-compose.yaml:

```
docker-compose up
```

Or run it locally:

```
npm run start
```

When running locally you should start your own PostgreSQL database.
There are many ways to do this, one suggestion is to use Docker:

```
docker run --name testkube-sample-db \
  -p 15432:5432 \
  -e POSTGRES_DB=api-db \
  -e POSTGRES_USER=api-user \
  -e POSTGRES_PASSWORD=api-password \
  --rm -d postgres
```

## Running tests

Unit tests can be executed without a running appliction:

```
npm run test
```

For E2E test, you should first run the application as described above:

```
npm run test:e2e
```

## Running on local Kubernetes

The repository includes a Helm chart and a PowerShell script for deploying the
complete application to a local `kind` cluster:

```text
Docker images -> kind -> Kubernetes
                         |-> frontend
                         |-> backend
                         `-> PostgreSQL + persistent volume
```

### Prerequisites

Install and start Docker Desktop. Docker must be running because `kind` runs
Kubernetes nodes as Docker containers. Install the following command-line tools:

- Docker Desktop
- `kubectl`
- `kind`
- Helm 3

On Windows, the tools can be installed with WinGet:

```powershell
winget install --id Kubernetes.kubectl --exact
winget install --id Kubernetes.kind --exact
winget install --id Helm.Helm --exact
```

Open a new PowerShell terminal after installation and verify the tools:

```powershell
docker version
kubectl version --client
kind version
helm version
```

`docker version` must show both Client and Server information. If Docker
reports that the engine is unavailable, open Docker Desktop and wait until it
is running.

### Deploy the application

From the repository root, allow the deployment script for the current PowerShell
session and run it:

```powershell
cd C:\Users\<your-user>\testkube-samples
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\scripts\deploy-kind.ps1
```

The script builds the backend and frontend images, creates or reuses a kind
cluster named `testkube-samples`, loads the images into the cluster, installs
the `helm/app` chart, and waits for the application Deployments to become
available.

You can customize the cluster, Helm release, or image tag:

```powershell
.\scripts\deploy-kind.ps1 `
  -ClusterName testkube-samples `
  -ReleaseName sample-app `
  -ImageTag dev
```

### Verify the deployment

Use these commands from another PowerShell terminal while the deployment script
is running or after it completes:

```powershell
kubectl get nodes
kubectl get pods
kubectl get services
kubectl get pvc
```

The Pods should become `Running`, and the services should include:

```text
sample-app-frontend
sample-app-backend
sample-app-postgres
```

If a Pod is not starting, inspect it with:

```powershell
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Access the application

The Helm services use `ClusterIP`, so they are available inside Kubernetes but
not directly from the host. Keep each port-forward command in its own terminal.

Frontend:

```powershell
kubectl port-forward service/sample-app-frontend 4173:4173
```

Open <http://localhost:4173> in a browser.

Backend:

```powershell
kubectl port-forward service/sample-app-backend 8080:8080
```

In another terminal, test the API and database connection:

```powershell
Invoke-WebRequest http://localhost:8080/hello
Invoke-WebRequest http://localhost:8080/hello-pg
```

The `/hello-pg` response should contain:

```text
hello world from postgres
```

### Inspect or remove the deployment

Inspect the Helm release:

```powershell
helm list
helm status sample-app
helm history sample-app
```

Remove the application but keep the cluster:

```powershell
helm uninstall sample-app
```

Remove the local Kubernetes cluster completely:

```powershell
kind delete cluster --name testkube-samples
```

If Docker Desktop reports that virtualization is not available, hardware
virtualization must be enabled in BIOS/UEFI or by the IT administrator before
Docker and kind can run.
