# 🚀 End-to-End DevOps CI/CD Project (AWS, Jenkins, Kubernetes, Observability)

## 📌 Project Overview

This project demonstrates a **production-grade DevOps CI/CD pipeline** for a **Node.js application** deployed on **AWS Kubernetes (EKS)** with **full automation, security scanning, monitoring, and observability**.

---

## 🧱 Architecture

```
GitHub
   │
   ▼
Jenkins (EC2)
   │
   ├── Code Quality Scan (SonarCloud)
   ├── Docker Build
   ├── Push Image → AWS ECR
   └── Deploy → AWS EKS
                │
                ├── Kubernetes Deployment
                ├── Service (LoadBalancer)
                ├── HPA (Auto Scaling)
                └── ServiceMonitor
                        │
                        ▼
                Prometheus → Grafana
```

---

## 🛠 Tools & Technologies Used

| Category           | Tools                                   |
| ------------------ | --------------------------------------- |
| Cloud Provider     | AWS                                     |
| CI/CD              | Jenkins                                 |
| IaC                | Terraform (Infrastructure provisioning) |
| Containerization   | Docker                                  |
| Container Registry | AWS ECR                                 |
| Orchestration      | Kubernetes (EKS)                        |
| Monitoring         | Prometheus                              |
| Visualization      | Grafana                                 |
| Code Quality       | SonarCloud                              |
| Package Runtime    | Node.js                                 |
| Deployment Method  | GitOps-style via Jenkins                |

---

## 📂 Repository Structure

```
end-to-end-node-ci-cd/
├── app/
│   ├── app.js
│   ├── Dockerfile
│   ├── package-lock.json
│   ├── package.json
│   └── ...
├── infra/
│   ├── main.tf
│   ├── eks.tf
│   ├── outputs.tf
│   ├── provider.tf
│   └── variables.tf
├── K8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── hpa.yaml
│   └── service-monitor.yaml
├── Jenkinsfile
├── script.sh
└── README.md
```

---

## 🔄 CI/CD Pipeline Stages (Jenkins)

### 1️⃣ Checkout Code

* Pulls source code from GitHub

### 2️⃣ SonarCloud Code Quality Scan

* Runs static code analysis
* Enforces **quality gates** (pipeline fails if code quality is poor)

### 3️⃣ Docker Build

* Builds Docker image from `app/Dockerfile`
* Tags image using Jenkins `BUILD_NUMBER`

### 4️⃣ Push Image to AWS ECR

* Authenticates to ECR using AWS credentials
* Pushes versioned Docker image

### 5️⃣ Deploy to Kubernetes (EKS)

This stage performs **full Kubernetes automation**:

* Uses Jenkins-stored kubeconfig for cluster access
* Installs Prometheus + Grafana using Helm
* Updates Kubernetes deployment image dynamically
* Applies all Kubernetes manifests
* Verifies rollout success

---

## ⚙ Jenkinsfile – Deployment Logic Explained

### 🔹 Image Tag Replacement

```bash
sed -i "s|ECR_URI:latest|${REPOSITORY_URI}:${IMAGE_TAG}|g" K8s/deployment.yaml
```

✔ Dynamically injects the newly built image into Kubernetes manifests
✔ Ensures every deployment uses the correct image version

---

### 🔹 Kubernetes Apply

```bash
kubectl apply -f K8s/
```

✔ Creates or updates:

* Deployment
* Service
* Horizontal Pod Autoscaler
* ServiceMonitor

---

### 🔹 Rollout Verification

```bash
kubectl rollout status deployment/node-app
```

✔ Confirms zero-downtime rolling deployment
✔ Pipeline waits until new pods are healthy

---

## ☸ Kubernetes Manifests Explained

### 🔸 Deployment

* Runs Node.js app in pods
* Defines resource requests & limits
* Includes readiness & liveness probes

### 🔸 Service (LoadBalancer)

* Exposes application publicly via AWS ELB

Example:

```
a223e85c1d47649f18ac6b778744e41a.us-east-1.elb.amazonaws.com
```

### 🔸 Horizontal Pod Autoscaler (HPA)

* Automatically scales pods based on CPU usage

### 🔸 ServiceMonitor

* Tells Prometheus how to scrape application metrics

---

## 📊 Monitoring & Observability

### Prometheus

* Scrapes Kubernetes + application metrics
* Uses ServiceMonitor CRDs

### Grafana

* Visualizes metrics from Prometheus
* Includes Kubernetes dashboards out-of-the-box
* Custom dashboards can be created for app metrics

Access Grafana:

```bash
kubectl port-forward svc/prometheus-grafana -n monitoring 3001:80
```

Login:

* Username: `admin`
* Password: Retrieved via Kubernetes secret

---

## 🔐 Security & Best Practices

✔ Secrets stored in Jenkins Credentials
✔ No hardcoded AWS keys
✔ Immutable Docker images
✔ Automated quality gates
✔ Versioned deployments
✔ Infrastructure separated from application code


## 🧪 How to Verify Deployment

1. Check application URL (LoadBalancer):

```bash
kubectl get svc node-app
```

2. Verify pods:

```bash
kubectl get pods
```

3. Check Prometheus targets
4. View Grafana dashboards

---

Key lessons:

* CI/CD before Kubernetes
* Image immutability
* Observability is not optional
* Automation over manual work
* Pipelines enforce standards

---

## 🏁 Note

This project is intentionally designed to mirror **enterprise DevOps workflows**.

You can extend it with:

* Snyk container scanning
* Canary deployments
* ArgoCD GitOps
* Blue/Green deployments

---

> ⚠️ Important clarification:
>
> * The **Dockerfile already exists** in the repository
> * The **script.sh already exists** and is used to bootstrap the Jenkins server
> * **Docker build & push**, **SonarQube scanning**, and **Prometheus/Grafana installation** are executed **inside the Jenkins pipeline**, not manually

This guide below is written so that **anyone can follow it step-by-step and successfully deploy the application**.

---

## 1. App Creation

The application is a simple **Node.js HTTP service** exposing:

* `/` – application endpoint
* `/health` – Kubernetes readiness & liveness probes
* `/metrics` – Prometheus metrics endpoint


No manual `npm install` is required on Jenkins — dependencies are installed during Docker build.

---

## 2. Dockerization of the App

The application is containerized using an **existing Dockerfile** located in the `app/` directory.

Key points:

* Uses Node 18 runtime
* Installs dependencies inside the image
* Exposes port `3000`

Docker image creation and tagging are handled **entirely by Jenkins**.

---

## 3. Infrastructure Provisioning with Terraform

Terraform provisions:

* VPC and networking
* EKS cluster (`devops-eks`)
* Node groups
* IAM roles and policies
* ECR repository

### Apply infrastructure

```bash
cd infra
terraform init
terraform apply
```

### Configure kubectl access

```bash
aws eks update-kubeconfig --region us-east-1 --name devops-eks
```

This kubeconfig is later uploaded to Jenkins as a **file credential**.

---

## 4. Jenkins Server Setup and Installation

Jenkins runs on an EC2 instance.

An **existing `script.sh`** is used to bootstrap the server.

### On the Jenkins EC2 instance

```bash
touch script.sh
chmod +x script.sh
nano script.sh
```
copy the script.sh in the project folder into the script.sh in the jenkins server

Reboot the server after running the script.

---

## 5. GitHub Integration

* Repository hosted on GitHub
* Jenkins job configured as **Pipeline from SCM**
* Webhook triggers builds on push

---

## 6. Jenkins Pipeline

The Jenkins pipeline performs:

1. Checkout code
2. SonarCloud SAST scan
3. Docker build
4. Push image to Amazon ECR
5. Deploy to Kubernetes
6. Install monitoring stack

### Jenkins Tools Configuration

* **JDK**: jdk17
* **NodeJS**: node18

### Required Jenkins Plugins

* NodeJS
* Docker Commons
* Docker Pipeline
* GitHub Integration
* SonarQube Scanner
* Kubernetes CLI
* AWS Credentials

---

## 7. SAST with SonarQube (SonarCloud)

Static code analysis is performed **inside the pipeline** using SonarCloud.

The scan:

* Analyzes all source files (`sonar.sources=.`)
* Enforces Quality Gates
* Fails the pipeline if gate is not met

Sonar scanning is executed via a Dockerized scanner image.

---

## 8. Docker Build and Push to ECR

Docker build and push are fully automated in Jenkins:

* Image tag = Jenkins build number
* Image pushed to Amazon ECR

Example image:

```
717279705656.dkr.ecr.us-east-1.amazonaws.com/devops-lab:7
```

No manual Docker commands are required.

---

## 9. Kubernetes Deployment & Monitoring

### Kubernetes Deployment

Jenkins:

* Updates image tag in `deployment.yaml`
* Applies manifests from `K8s/`
* Waits for rollout completion

Resources deployed:

* Deployment
* Service (LoadBalancer)
* HPA
* ServiceMonitor

### Monitoring with Prometheus & Grafana

Installed automatically via Helm inside the pipeline:

```bash
helm upgrade --install prometheus \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

Grafana dashboards visualize:

* Pod CPU & memory
* Application metrics
* Kubernetes health

---

## Cleanup of Resources

After testing, clean up to avoid costs:

```bash
helm uninstall prometheus -n monitoring
kubectl delete -f K8s/
cd infra
terraform destroy
```

---

## What This Project Demonstrates

* Real-world CI/CD pipeline design
* Secure image delivery to EKS
* Infrastructure as Code
* Kubernetes production deployment
* Observability and monitoring
* DevOps best practices end-to-end

---

