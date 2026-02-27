# Blogging App — Enterprise DevSecOps Pipeline 🔐

> A production-grade DevSecOps CI/CD pipeline for a Java Spring Boot blogging application — built with Jenkins, secured with Trivy and SonarQube, artifact-managed via Nexus, and deployed on AWS EKS provisioned entirely through Terraform.

---

## 🏗️ Architecture Overview

```
Developer Push
      │
      ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Jenkins Pipeline                          │
│                                                                  │
│  Git Checkout → Compile → Test → Trivy FS Scan                  │
│       │                                │                         │
│       ▼                                ▼                         │
│  SonarQube Analysis              Security Report (fs.html)       │
│       │                                                          │
│       ▼                                                          │
│    Build (mvn package) → Publish to Nexus                        │
│       │                                                          │
│       ▼                                                          │
│  Docker Build → Trivy Image Scan → Docker Push (DockerHub)       │
│       │                    │                                     │
│       ▼                    ▼                                     │
│  k8s Deploy           Security Report (image.html)               │
│  (AWS EKS)                                                       │
│       │                                                          │
│       ▼                                                          │
│  Verify Deployment (kubectl get pods/svc)                        │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
  AWS EKS Cluster (Provisioned via Terraform)
  Namespace: webapps
```

---

## ✨ Key Features

- **10-Stage Jenkins Pipeline** — Full lifecycle from code checkout to live deployment verification
- **Dual Security Scanning** — Trivy scans both the filesystem AND the Docker image, generating HTML security reports at each stage
- **SonarQube Code Quality Gate** — Static analysis enforced before build, blocking low-quality code from reaching production
- **Nexus Artifact Repository** — Maven artifacts published and versioned centrally via Nexus for traceability
- **RBAC-Secured Kubernetes** — Jenkins communicates with EKS using scoped kubeconfig credentials, not broad cluster access
- **Terraform-Provisioned Infrastructure** — Entire EKS cluster built as code — repeatable, version-controlled, and team-shareable
- **Deployment Verification** — Pipeline automatically verifies pods and services are running after every deploy

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Application** | Java, Spring Boot, Maven |
| **CI/CD** | Jenkins (Declarative Pipeline) |
| **Code Quality** | SonarQube |
| **Security Scanning** | Trivy (filesystem + image) |
| **Artifact Management** | Nexus Repository |
| **Containerization** | Docker |
| **Container Registry** | DockerHub |
| **Orchestration** | Kubernetes (AWS EKS) |
| **Infrastructure as Code** | Terraform |
| **Namespace** | webapps |

---

## 🔄 Pipeline Deep Dive — 10 Stages

### Stage 1 — Git Checkout
Pulls the latest code from the `main` branch using secured Git credentials stored in Jenkins.

### Stage 2 — Compile
Compiles the Java source code using Maven and JDK 17, catching any compilation errors early.

### Stage 3 — Test
Runs the full Maven test suite. Pipeline halts if any tests fail — no broken code moves forward.

### Stage 4 — Trivy Filesystem Scan ⭐
Scans the entire repository for vulnerabilities in dependencies and config files **before** a Docker image is even built. Outputs a detailed `fs.html` security report.

### Stage 5 — SonarQube Analysis
Runs static code analysis against the compiled Java binaries. Checks for bugs, code smells, security hotspots, and maintainability issues.

### Stage 6 — Build
Packages the application into a deployable JAR using `mvn package`.

### Stage 7 — Publish Artifacts to Nexus
Deploys the built artifact to Nexus Repository using global Maven settings for traceability and version management across environments.

### Stage 8 — Docker Build & Tag
Builds the Docker image and tags it as `bijit5/bloggingapp:latest` using secured DockerHub credentials.

### Stage 9 — Trivy Image Scan ⭐
Scans the final Docker image for OS-level and application-level CVEs **before** it gets pushed. Outputs `image.html` report. This is the second security gate.

### Stage 10 — Docker Push
Pushes the security-verified image to DockerHub only after passing both Trivy scans.

### Stage 11 — Kubernetes Deploy
Applies `deployment-service.yml` to the `webapps` namespace on the AWS EKS cluster using RBAC-scoped kubeconfig credentials.

### Stage 12 — Verify Deployment
Runs `kubectl get pods` and `kubectl get svc` to confirm pods are running and services are exposed — automated health check after every deploy.

---

## 🔐 Security Architecture

This pipeline implements **defense in depth** with two independent security scanning layers:

```
Source Code          →    Trivy FS Scan      →  Catches vulnerable dependencies
                                                  & misconfigurations early

Docker Image         →    Trivy Image Scan   →  Catches OS-level CVEs
                                                  before image is pushed

Java Code            →    SonarQube          →  Catches bugs, security hotspots,
                                                  code smells before build
```

Both Trivy scans generate HTML reports stored as pipeline artifacts for audit and compliance purposes.

---

## ☁️ Infrastructure — Terraform on AWS EKS

The Kubernetes cluster is not manually set up — it is fully provisioned via Terraform:

```
Terraform files for EKS/
├── main.tf          # EKS cluster definition
├── variables.tf     # Configurable parameters
└── outputs.tf       # Cluster endpoint and kubeconfig outputs
```

This means the entire infrastructure can be spun up, torn down, or replicated in any AWS region with a single `terraform apply`.

---

## 📁 Project Structure

```
Blogging-App-Devops/
├── Terraform files for EKS/     # IaC for AWS EKS cluster
├── src/                         # Java Spring Boot source code
├── Dockerfile                   # Container build definition
├── deployment-service.yml       # Kubernetes Deployment + Service manifest
├── jenkinfile                   # 10-stage declarative Jenkins pipeline
├── pom.xml                      # Maven project configuration
└── mvnw                         # Maven wrapper
```

---

## 📈 Impact

| Metric | Result |
|---|---|
| Deployment process automated | **90%** |
| Security scanning stages | **2 independent layers** (FS + Image) |
| Infrastructure provisioning | Fully automated via **Terraform** |
| Code quality | Enforced via **SonarQube** on every push |
| Artifact management | Centralized via **Nexus** |
| Cluster access | **RBAC-scoped** — least privilege principle |

---

## 🚀 How to Run This Pipeline

**Prerequisites:**
- Jenkins with plugins: Maven, Docker, SonarQube, Kubernetes, Nexus, Trivy
- AWS account with EKS access
- DockerHub account
- SonarQube server running
- Nexus Repository running

**Steps:**
1. Provision EKS cluster: `cd "Terraform files for EKS" && terraform init && terraform apply`
2. Configure Jenkins credentials: `git-login`, `docker-cred`, `k8-cred`
3. Configure SonarQube server as `sonar-server` in Jenkins
4. Configure Nexus in Maven global settings as `maven-settings`
5. Create a Jenkins pipeline job pointing to this repo
6. Push to `main` — the pipeline runs automatically

---

## 👤 Author

**Bijit Kalita** — DevOps Engineer
- 📧 bijit987kalita@gmail.com
- 💼 [linkedin.com/in/bijit-kalita](https://linkedin.com/in/bijit-kalita/)
- 🐙 [github.com/bijit5](https://github.com/bijit5)
