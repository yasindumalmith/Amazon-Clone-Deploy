# **🚀 Amazon Clone – End-to-End DevSecOps CI/CD with GitOps on AWS EKS**

### **📌 Project Overview**

This project demonstrates a **real-world DevSecOps CI/CD pipeline** to build, scan, containerize, and deploy an Amazon Clone application on **AWS EKS** using **Jenkins, Helm, and Argo CD (GitOps)**.

The pipeline integrates **security, quality checks, container scanning, and automated Kubernetes deployments**, closely following **industry best practices**.

### **🧰 Technologies Used**

**🔹 CI / DevSecOps**

- Jenkins (Declarative Pipeline)
- SonarQube (Code Quality & Quality Gates)
- OWASP Dependency-Check
- Trivy (Filesystem & Image Scanning)
- Docker & Docker Hub
- GitHub (Source Control & GitOps)

**🔹 CD / GitOps**

- Helm (Kubernetes packaging & templating)
- Argo CD (GitOps-based continuous deployment)

**🔹 Cloud & Kubernetes**

- AWS EKS
- Kubernetes (Deployments, Services, Ingress)
- AWS ALB Ingress Controller
- AWS VPC (Public & Private Subnets, NAT Gateway)
- IAM Roles & Policies

\
\
\
\
**-----**

**🔄 CI Pipeline Workflow (Jenkins)**

The Jenkins pipeline automates the following steps:

**1️⃣ Workspace & Source Code**

- Cleans Jenkins workspace
- Clones application source code from GitHub

**2️⃣ Code Quality & Security**

- **SonarQube Analysis**
  - Code quality checks
  - Enforced quality gate
- **OWASP Dependency-Check**
  - Detects vulnerable dependencies
- **Trivy File System Scan**
  - Scans source code for vulnerabilities

**3️⃣ Build & Containerization**

- Installs Node.js dependencies
- Builds Docker image
- Tags image using Jenkins build number

**4️⃣ Container Security**

- **Trivy Image Scan**
  - Generates JSON & table reports
  - Scans for OS & application vulnerabilities

**5️⃣ Image Registry**

- Pushes Docker image to Docker Hub

**6️⃣ GitOps Deployment Trigger**

- Clones Kubernetes GitOps repository(Another Repository, Link mentioned in the below)
- Updates Helm values.yaml with new image tag
- Commits & pushes changes to GitHub\

**7️⃣ Notifications**
- Sends email with:
  - Build status
  - Reports (Trivy & OWASP)
  - Build URL
-----
**📦 GitOps-Based CD with Argo CD**

- Argo CD continuously watches the **Helm GitOps repository**
- Another repository is used for GitOps:- [**https://github.com/yasindumalmith/Amazon-K8s-Deploy.git**](https://github.com/yasindumalmith/Amazon-K8s-Deploy.git)

- When the image tag changes in values.yaml:
  - Argo CD automatically syncs
  - Deploys the updated application to EKS
- Ensures:
  - Declarative deployments
  - Versioned rollbacks
  - Audit-friendly workflow
-----

**AWS VPC**

- Create the VPC with 2 Availability Zone(us-east-1a, us-east-1b)
- Worker nodes are in the private subnets
- Use only one Nat Gateway(public subnet) for both private worker Node.
- Use ALB Manage the traffic.

-----

**🌐 Kubernetes Deployment Details**

- Application deployed using **Helm charts**
- Kubernetes resources:
  - Deployment
  - Service (ClusterIP)
  - Ingress (ALB)
- Ingress:
  - AWS Application Load Balancer
- Pods run in **private subnets** for security
-----

**🔐 Security Highlights**

- Static code analysis (SonarQube)
- Dependency vulnerability scanning (OWASP)
- Container image scanning (Trivy)
- Private Kubernetes nodes
- IAM-based access control

