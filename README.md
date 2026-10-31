# Example Voting App — DevOps Capstone

A containerized distributed voting application deployed to Amazon EKS using Terraform, GitHub Actions, and Helm.

## Project Overview

The Example Voting App is a distributed application consisting of five services:

- **Vote** — Python web application where users cast votes.
- **Redis** — Stores incoming votes temporarily.
- **Worker** — .NET service that consumes votes from Redis and stores them in PostgreSQL.
- **PostgreSQL** — Persistent data store for the voting results.
- **Result** — Node.js web application that displays the results.

### Application Flow

```text
User
  |
  v
Vote App
  |
  v
Redis
  |
  v
Worker
  |
  v
PostgreSQL
  |
  v
Result App
```

## DevOps Implementation

This capstone demonstrates:

- Docker containerization and hardening
- Non-root containers
- Pinned Docker base images
- Docker `.dockerignore` files
- Trivy vulnerability scanning
- GitHub Actions CI/CD
- GitHub Container Registry (GHCR)
- Terraform infrastructure provisioning
- Amazon EKS
- Kubernetes
- Helm deployments
- Helm rollback

## 1. Run Locally with Docker Compose

Start the application:

```bash
docker compose up
```

The Vote application is available at:

```text
http://localhost:8080
```

The Result application is available at:

```text
http://localhost:8081
```

## 2. Container Security

The application images were hardened by:

- Pinning Docker base images to immutable SHA256 digests.
- Running application containers as non-root users where applicable.
- Adding `.dockerignore` files.
- Removing unnecessary packages and dependencies.
- Scanning built images with Trivy.

The GitHub Actions pipeline fails when HIGH or CRITICAL vulnerabilities are detected.

## 3. Infrastructure with Terraform

Terraform was used to provision the Amazon EKS cluster.

The Terraform configuration creates the Kubernetes cluster infrastructure in AWS and allows the environment to be rebuilt from code.

Terraform directory:

```text
terraform/
```

Validate the configuration:

```bash
terraform validate
```

Apply the infrastructure:

```bash
terraform apply
```

Configure kubectl:

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name voting-app-cluster
```

Verify the cluster:

```bash
kubectl get nodes
```

## 4. GitHub Actions CI/CD

The GitHub Actions workflow is located at:

```text
.github/workflows/ci-cd.yaml
```

Every push to `main` runs the following pipeline:

```text
GitHub Push
    |
    v
Integration Tests
    |
    v
Build Docker Images
    |
    v
Trivy Security Scan
    |
    v
Push Images to GHCR
```

Images are tagged using the Git commit SHA.

Example:

```text
ghcr.io/chidinma-git/example-voting-app-vote:<commit-sha>
```

The pipeline builds and scans:

- Vote
- Result
- Worker

## 5. Deploy with Helm

The Kubernetes deployment is packaged as a Helm chart:

```text
helm/
├── Chart.yaml
├── values.yaml
├── .helmignore
└── templates/
    ├── db-deployment.yaml
    ├── db-service.yaml
    ├── redis-deployment.yaml
    ├── redis-service.yaml
    ├── result-deployment.yaml
    ├── result-service.yaml
    ├── vote-deployment.yaml
    ├── vote-service.yaml
    └── worker-deployment.yaml
```

Validate the chart:

```bash
helm lint helm
```

Render the Kubernetes manifests:

```bash
helm template voting-app helm
```

Install the application:

```bash
helm install voting-app ./helm
```

Check the deployment:

```bash
kubectl get pods
kubectl get services
```

## 6. Application Services

The deployed services use the following Kubernetes configuration:

| Service | Type | Port |
|------------|------------|------:|
| Vote       | NodePort   | 31000 |
| Result     | NodePort   | 31001 |
| Redis      | ClusterIP  | 6379  |
| PostgreSQL | ClusterIP  | 5432  |

The Vote and Result applications are externally accessible through the EKS worker nodes.

## 7. Helm Rollback Demonstration

A bad release was intentionally deployed using an invalid Vote image tag:

```bash
helm upgrade voting-app ./helm \
  --set images.vote.tag=bad-release-demo
```

The release history was then inspected:

```bash
helm history voting-app
```

The application was restored to the known-good revision:

```bash
helm rollback voting-app 1
```

The rollback created a new deployed revision.

Final verification:

```bash
kubectl get pods
```

All five application components returned to:

```text
1/1 Running
```

This demonstrates that the application can be rolled back after a bad deployment.

## 8. Verification

The deployed application was tested end-to-end.

The Vote service returned:

```text
HTTP/1.1 200 OK
```

The Result service returned:

```text
HTTP/1.1 200 OK
```

A real vote was submitted through the deployed application and successfully appeared in the Result application.

This verified the complete application flow:

```text
Vote
  ↓
Redis
  ↓
Worker
  ↓
PostgreSQL
  ↓
Result
```

## 9. Repository Structure

```text
example-voting-app/
├── .github/
│   └── workflows/
│       └── ci-cd.yaml
├── helm/
├── k8s-specifications/
├── result/
├── vote/
├── worker/
├── terraform/
├── docker-compose.yml
└── README.md
```

## 10. Capstone 1 Deliverables

The project includes:

- [x] Forked Voting App repository
- [x] Docker containerization
- [x] Hardened Docker images
- [x] Pinned base images
- [x] Non-root containers
- [x] Docker `.dockerignore`
- [x] Terraform EKS infrastructure
- [x] GitHub Actions CI/CD pipeline
- [x] Trivy vulnerability scanning
- [x] SHA-tagged container images
- [x] GHCR image publishing
- [x] Helm chart
- [x] Working EKS deployment
- [x] Successful end-to-end voting test
- [x] Helm rollback demonstration
- [ ] Architecture diagram

## Architecture

The application architecture diagram should show:

```text
                    AWS / Amazon EKS
                         |
              +----------+----------+
              |                     |
        Vote NodePort          Result NodePort
          :31000                  :31001
              |                     |
              v                     v
          Vote Pod              Result Pod
              |                     |
              v                     |
          Redis Pod                 |
              |                     |
              v                     |
          Worker Pod                |
              |                     |
              v                     |
        PostgreSQL Pod <------------+
```

## Technologies

- Docker
- Docker Compose
- Kubernetes
- Amazon EKS
- Terraform
- Helm
- GitHub Actions
- GitHub Container Registry
- Trivy
- Python
- Node.js
- .NET
- Redis
- PostgreSQL