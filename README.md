The project will demonstrate the skills you’ve been working toward: **AWS + Terraform + Docker + Kubernetes/EKS + GitHub Actions + IAM/OIDC + networking + security + monitoring + rollback + cost optimization**.

## Project: Production-Grade E-Commerce / Order Management Platform

### 1. What you will build

A containerized backend application deployed on AWS:

```text
                         Internet
                            |
                            v
                    +----------------+
                    |  Route 53      |
                    |  DNS            |
                    +--------+-------+
                             |
                             v
                    +----------------+
                    | AWS ALB        |
                    | HTTPS           |
                    +--------+-------+
                             |
              +--------------+--------------+
              |                             |
              v                             v
       +-------------+               +-------------+
       | EKS         |               | EKS         |
       | AZ-1        |               | AZ-2        |
       | Private     |               | Private     |
       +------+------+               +------+------+
              |                             |
              +--------------+--------------+
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
         Backend API     Order Service   Notification
         Java/Spring     Java/Spring     Service
              |              |
              +------+-------+
                     |
          +----------+----------+
          |                     |
          v                     v
    +-----------+        +-------------+
    | Aurora /  |        | Redis       |
    | RDS       |        | ElastiCache |
    +-----------+        +-------------+
          
                     |
                     v
                 +-------+
                 | S3    |
                 | Logs/ |
                 | Files |
                 +-------+

 CI/CD
 =====

 Developer
    |
    v
 GitHub
    |
    v
 GitHub Actions
    |
    +--> Unit Tests
    |
    +--> SonarQube
    |
    +--> Docker Build
    |
    +--> Security Scan
    |
    +--> Push Image
    |       |
    |       v
    |      ECR
    |
    +--> Terraform
    |
    +--> Deploy to EKS
```

AWS currently supports EKS Auto Mode, which can manage significant portions of compute, networking, load balancing and storage infrastructure. For a learning project, however, I recommend first building the **traditional EKS architecture manually with Terraform**, because it exposes much more of the DevOps concepts you want to demonstrate. AWS's EKS guidance also emphasizes VPC design, security, reliability, scaling and cost optimization. ([AWS Documentation][1])

---

# 2. Technology stack

| Layer              | Technology                |
| ------------------ | ------------------------- |
| Application        | Java 17 + Spring Boot     |
| API                | REST                      |
| Container          | Docker                    |
| Container Registry | Amazon ECR                |
| Kubernetes         | Amazon EKS                |
| Infrastructure     | Terraform                 |
| Networking         | Amazon VPC                |
| Load Balancer      | AWS ALB                   |
| Database           | Amazon RDS PostgreSQL     |
| Cache              | ElastiCache Redis         |
| Storage            | Amazon S3                 |
| Secrets            | AWS Secrets Manager       |
| IAM                | IAM + EKS Pod Identity    |
| CI/CD              | GitHub Actions            |
| Monitoring         | CloudWatch                |
| Audit              | CloudTrail                |
| Security           | Security Groups, IAM, KMS |
| DNS                | Route 53                  |
| TLS                | ACM                       |
| Logs               | CloudWatch                |
| Code               | GitHub                    |
| IaC State          | S3 + locking mechanism    |

For new EKS Auto Mode designs, AWS recommends EKS Pod Identity for workload IAM, while traditional EKS deployments can also use IRSA. ([AWS Documentation][2])

---

# 3. Project requirements

The application can be simple.

For example:

### Order Management API

```text
POST   /api/orders
GET    /api/orders
GET    /api/orders/{id}
PUT    /api/orders/{id}
DELETE /api/orders/{id}

GET    /api/health
```

Example:

```json
POST /api/orders

{
  "customerId": "C1001",
  "productId": "P1001",
  "quantity": 2
}
```

Response:

```json
{
  "orderId": "ORD-10001",
  "status": "CREATED",
  "customerId": "C1001"
}
```

This is enough application functionality to demonstrate the complete DevOps lifecycle.

---

# 4. Repository structure

I recommend creating the repository like this:

```text
aws-order-management/
│
├── application/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── terraform/
│   │
│   ├── environments/
│   │   ├── dev/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   ├── outputs.tf
│   │   │   └── terraform.tfvars
│   │   │
│   │   └── prod/
│   │       ├── main.tf
│   │       ├── variables.tf
│   │       ├── outputs.tf
│   │       └── terraform.tfvars
│   │
│   └── modules/
│       │
│       ├── vpc/
│       │   ├── main.tf
│       │   ├── variables.tf
│       │   └── outputs.tf
│       │
│       ├── eks/
│       │   ├── main.tf
│       │   ├── variables.tf
│       │   └── outputs.tf
│       │
│       ├── ecr/
│       ├── rds/
│       ├── redis/
│       ├── iam/
│       ├── s3/
│       └── monitoring/
│
├── kubernetes/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── serviceaccount.yaml
│   └── hpa.yaml
│
├── helm/
│   └── order-service/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│
├── .github/
│   └── workflows/
│       ├── ci.yaml
│       ├── terraform.yaml
│       └── deploy.yaml
│
├── scripts/
│   ├── build.sh
│   ├── deploy.sh
│   └── destroy.sh
│
├── docs/
│   ├── architecture.md
│   ├── deployment.md
│   ├── troubleshooting.md
│   └── disaster-recovery.md
│
└── README.md
```

This structure itself is something you can discuss during an interview.

---

# 5. Terraform architecture

Terraform will create:

```text
Terraform
   |
   +-- VPC
   |
   +-- Internet Gateway
   |
   +-- Public Subnets
   |
   +-- Private Subnets
   |
   +-- NAT Gateway
   |
   +-- Route Tables
   |
   +-- Security Groups
   |
   +-- EKS
   |
   +-- EKS Node Groups
   |
   +-- ECR
   |
   +-- RDS
   |
   +-- ElastiCache
   |
   +-- S3
   |
   +-- IAM
   |
   +-- KMS
   |
   +-- CloudWatch
```

AWS's EKS networking guidance explicitly supports the public/private subnet model, and EKS workloads are deployed into the VPC. ([AWS Documentation][3])

---

# 6. Terraform provider

Start with:

```hcl
terraform {
  required_version = ">= 1.13.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = "aws-order-management"
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
```

AWS's current EKS Terraform documentation uses modern Terraform versions, and AWS's own examples explicitly warn that EKS infrastructure such as clusters, compute and load balancers incurs charges, so cleanup should be part of the hands-on workflow. ([AWS Documentation][4])

---

# 7. VPC Terraform

Create:

```text
VPC
10.0.0.0/16

        VPC
         |
   +-----+-----+
   |           |
Public        Private
Subnet        Subnet
   |             |
 ALB            EKS
   |             |
 NAT -----------+
```

Example:

```hcl
module "vpc" {
  source = "../../modules/vpc"

  name = "order-management"

  cidr = "10.0.0.0/16"

  availability_zones = [
    "ap-south-1a",
    "ap-south-1b"
  ]

  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnets = [
    "10.0.11.0/24",
    "10.0.12.0/24"
  ]

  environment = var.environment
}
```

---

# 8. EKS

Terraform will create:

```text
EKS Cluster
    |
    +--- Node Group AZ-1
    |
    +--- Node Group AZ-2
```

Example:

```hcl
module "eks" {
  source = "../../modules/eks"

  cluster_name    = "order-management-${var.environment}"
  cluster_version = "1.33"

  vpc_id = module.vpc.vpc_id

  private_subnet_ids = module.vpc.private_subnet_ids

  node_instance_types = [
    "t3.medium"
  ]

  desired_capacity = 2
  min_capacity     = 1
  max_capacity     = 4
}
```

**Important:** before applying, use a currently supported EKS Kubernetes version in your chosen AWS Region rather than hard-coding an old version. AWS's current documentation shows EKS Auto Mode requires Kubernetes 1.29+, while supported versions continue to change. ([AWS Documentation][5])

---

# 9. ECR

Terraform:

```hcl
resource "aws_ecr_repository" "order_service" {
  name                 = "order-service"
  image_tag_mutability = "IMMUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }

  encryption_configuration {
    encryption_type = "AES256"
  }
}
```

Your CI pipeline will build:

```text
Java Application
       |
       v
Docker Image
       |
       v
ECR
       |
       v
EKS
```

---

# 10. RDS

For the learning version:

```text
EKS
 |
 | private network
 v
RDS PostgreSQL
```

Terraform:

```hcl
resource "aws_db_instance" "postgres" {
  identifier = "order-management-db"

  engine         = "postgres"
  engine_version = "17"

  instance_class = "db.t3.micro"

  allocated_storage = 20
  storage_type      = "gp3"

  db_name  = "orders"
  username = var.db_username
  password = var.db_password

  publicly_accessible = false

  skip_final_snapshot = true
}
```

For a production implementation, we'd change this to a stronger configuration with Multi-AZ, backups, encryption and appropriate parameter/security settings.

---

# 11. Redis

Use Redis for:

```text
API
 |
 +--> PostgreSQL
 |
 +--> Redis
       |
       +-- frequently accessed data
       +-- session/cache
       +-- performance optimization
```

This gives you a strong interview discussion around:

* caching
* TTL
* cache invalidation
* database load reduction
* scalability

---

# 12. Kubernetes deployment

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 2

  strategy:
    type: RollingUpdate

  selector:
    matchLabels:
      app: order-service

  template:
    metadata:
      labels:
        app: order-service

    spec:
      containers:
        - name: order-service

          image: ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/order-service:latest

          ports:
            - containerPort: 8080

          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"

            limits:
              cpu: "500m"
              memory: "1Gi"

          readinessProbe:
            httpGet:
              path: /api/health
              port: 8080

          livenessProbe:
            httpGet:
              path: /api/health
              port: 8080
```

---

# 13. Horizontal Pod Autoscaler

This is important for your DevOps project.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: order-service

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service

  minReplicas: 2
  maxReplicas: 6

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

So:

```text
Normal traffic
     |
     v
2 Pods

Traffic increases
     |
     v
3 Pods
     |
     v
4 Pods
     |
     v
6 Pods
```

---

# 14. CI/CD pipeline

This is where the project becomes a **real DevOps project**.

```text
Developer
    |
    v
Git Push
    |
    v
GitHub
    |
    v
GitHub Actions
    |
    +---- Unit Tests
    |
    +---- Code Quality
    |
    +---- Security Scan
    |
    +---- Docker Build
    |
    +---- Docker Scan
    |
    +---- Push to ECR
    |
    +---- Deploy EKS
    |
    +---- Smoke Test
    |
    +---- Notify
```

GitHub Actions should authenticate to AWS through **OIDC**, rather than storing long-lived AWS access keys in GitHub.

---

# 15. GitHub Actions

Example:

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ap-south-1

      - name: Login to ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build Docker Image
        run: |
          docker build \
            -t $ECR_REGISTRY/order-service:$GITHUB_SHA \
            ./application

      - name: Push Docker Image
        run: |
          docker push \
            $ECR_REGISTRY/order-service:$GITHUB_SHA

      - name: Deploy
        run: |
          aws eks update-kubeconfig \
            --region ap-south-1 \
            --name order-management-dev

          kubectl set image deployment/order-service \
            order-service=$ECR_REGISTRY/order-service:$GITHUB_SHA

          kubectl rollout status deployment/order-service
```

---

# 16. Terraform remote state

Don't keep the real production Terraform state only on your laptop.

Use:

```text
Terraform
    |
    v
S3
    |
    +-- terraform.tfstate
```

and state locking appropriate to your current Terraform/AWS setup.

Example:

```hcl
terraform {
  backend "s3" {
    bucket = "my-company-terraform-state"
    key    = "order-management/dev/terraform.tfstate"
    region = "ap-south-1"
  }
}
```

---

# 17. Secrets

Do **not** do this:

```yaml
password: MyPassword123
```

Instead:

```text
AWS Secrets Manager
        |
        v
     EKS Pod
        |
        v
 Spring Boot
        |
        v
      RDS
```

Store:

```text
DB_HOST
DB_NAME
DB_USERNAME
DB_PASSWORD
```

in Secrets Manager.

Then grant the workload only the permissions it needs.

---

# 18. Security architecture

Your project should demonstrate:

```text
Internet
   |
   v
ALB
   |
   v
EKS
   |
   +------> RDS
   |
   +------> Redis
   |
   +------> S3
```

Security controls:

### Network

* Public subnets only for internet-facing components
* EKS workloads in private subnets
* RDS private
* Redis private
* Security groups
* Network ACLs where appropriate

### IAM

Use least privilege.

For example:

```text
GitHub Actions
      |
      v
IAM Role
      |
      +--> ECR
      +--> EKS deployment

Application Pod
      |
      v
IAM Role
      |
      +--> S3
      +--> Secrets Manager
```

AWS's EKS security guidance recommends least-privilege workload access and monitoring cluster activity through CloudTrail/CloudWatch. ([AWS Documentation][6])

---

# 19. Monitoring

Implement:

```text
EKS
 |
 +-- CloudWatch
 |
 +-- Application Logs
 |
 +-- Container Logs
 |
 +-- Metrics
 |
 +-- Alerts
```

Create alarms for:

```text
CPU > 80%

Memory > 80%

Pod restart count

HTTP 5xx

RDS CPU

RDS storage

Database connections

ALB 5xx

ALB latency
```

---

# 20. Disaster recovery

Define:

```text
RTO = 1 hour
RPO = 15 minutes
```

Then design around them.

For example:

```text
                    Production
                        |
             +----------+----------+
             |                     |
             v                     v
           AZ-1                  AZ-2
             |                     |
           EKS                   EKS
             |                     |
             +----------+----------+
                        |
                       RDS
```

Backups:

```text
RDS automated backup
       |
       v
Point-in-time recovery
```

S3:

```text
Versioning
Lifecycle
Encryption
```

---

# 21. Cost optimization

For your hands-on project, **don't leave everything running continuously**.

AWS explicitly notes that EKS clusters, compute and load balancers can incur charges. ([AWS Documentation][4])

For learning:

```text
Development

EKS:
1–2 small nodes

RDS:
small instance

Redis:
small configuration

NAT:
minimize unnecessary usage

Logs:
short retention
```

And destroy the environment after practice:

```bash
terraform destroy
```

You can then explain:

> "I designed the development environment with cost optimization in mind by using smaller compute resources, controlling log retention, applying resource limits and destroying non-production infrastructure when it was not required."

---

# 22. Terraform commands

Your complete workflow becomes:

### Step 1 — Configure AWS

```bash
aws configure
```

### Step 2 — Clone repository

```bash
git clone <your-github-repository>
cd aws-order-management
```

### Step 3 — Initialize Terraform

```bash
cd terraform/environments/dev

terraform init
```

### Step 4 — Validate

```bash
terraform validate
```

### Step 5 — Format

```bash
terraform fmt -recursive
```

### Step 6 — Plan

```bash
terraform plan
```

### Step 7 — Deploy

```bash
terraform apply
```

### Step 8 — Connect to EKS

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name order-management-dev
```

### Step 9 — Verify

```bash
kubectl get nodes
kubectl get pods
kubectl get svc
kubectl get ingress
```

### Step 10 — Deploy application

```bash
kubectl apply -f kubernetes/
```

### Step 11 — Check rollout

```bash
kubectl rollout status deployment/order-service
```

### Step 12 — Test

```bash
curl https://your-domain.com/api/health
```

---

# 23. Failure testing

This is an important part of making the project **hands-on rather than just infrastructure creation**.

Test:

### Pod failure

```bash
kubectl delete pod <pod-name>
```

Expected:

```text
Pod deleted
     |
     v
Deployment detects failure
     |
     v
New pod created
```

### Deployment rollback

```bash
kubectl rollout history deployment/order-service
```

Then:

```bash
kubectl rollout undo deployment/order-service
```

### Scaling

Generate traffic:

```text
Traffic
   |
   v
CPU increases
   |
   v
HPA
   |
   v
More pods
```

### Database failure

Test application behavior when RDS becomes unavailable.

### Bad deployment

Deploy an intentionally broken image:

```bash
kubectl set image deployment/order-service \
order-service=bad-image
```

Observe:

```text
ImagePullBackOff
```

Then rollback.

This gives you excellent interview material.

---

# 24. Terraform modules

The final architecture should be modular:

```text
environment
     |
     +--- VPC module
     |
     +--- EKS module
     |
     +--- ECR module
     |
     +--- RDS module
     |
     +--- Redis module
     |
     +--- IAM module
     |
     +--- S3 module
     |
     +--- Monitoring module
```

That lets you explain:

> "I avoided putting the entire infrastructure in one Terraform file. I created reusable modules and environment-specific configurations for development and production."

---

# 25. Dev → QA → Production

Eventually:

```text
                GitHub
                   |
                   v
              Pull Request
                   |
                   v
              CI Pipeline
                   |
                   v
                  Dev
                   |
                Testing
                   |
                   v
                  QA
                   |
             Approval Gate
                   |
                   v
                Production
```

Terraform:

```text
terraform/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
└── modules/
    ├── vpc/
    ├── eks/
    ├── rds/
    ├── ecr/
    ├── redis/
    └── iam/
```

---

# 26. What you will learn

After completing this project, you can demonstrate:

### AWS

* VPC
* Subnets
* Route tables
* NAT Gateway
* Internet Gateway
* EKS
* EC2
* ECR
* ALB
* RDS
* ElastiCache
* S3
* IAM
* KMS
* Secrets Manager
* CloudWatch
* CloudTrail
* Route 53
* ACM

### Terraform

* Providers
* Variables
* Outputs
* Modules
* Data sources
* Resources
* State
* Remote backend
* Workspaces/environments
* `plan`
* `apply`
* `destroy`
* dependency management

### Kubernetes

* Namespace
* Deployment
* Service
* Ingress
* ConfigMap
* Secrets
* ServiceAccount
* HPA
* Readiness probes
* Liveness probes
* Rolling deployments
* Rollbacks

### DevOps

* Git
* GitHub
* GitHub Actions
* CI/CD
* Docker
* ECR
* OIDC
* Infrastructure as Code
* DevSecOps
* Monitoring
* Logging
* Troubleshooting
* Disaster recovery
* Cost optimization

---

# 27. How to explain this project in an interview

You can say:

> **"I designed and implemented a production-style containerized application on AWS using Terraform as Infrastructure as Code. The application was containerized using Docker and deployed on Amazon EKS across multiple availability zones. I created the VPC, public and private subnets, routing, security groups, EKS cluster, ECR repositories, RDS database, Redis cache, IAM roles, Secrets Manager configuration and monitoring infrastructure using reusable Terraform modules."**

Then:

> **"For CI/CD, I implemented GitHub Actions with OIDC-based AWS authentication. The pipeline performs application testing, Docker image creation, security validation, pushes the image to ECR and deploys the application to EKS. Kubernetes rolling deployments, readiness/liveness probes and HPA were used to provide resilient application deployment and scaling."**

Then:

> **"I also implemented monitoring through CloudWatch, centralized application logging, IAM least privilege, private database connectivity, secrets management and failure testing. I validated pod recovery, horizontal scaling and deployment rollback. For cost optimization, I used appropriately sized development resources, controlled log retention and destroyed non-production infrastructure when it was not required."**

That is a **much stronger DevOps project story** than simply saying "I created an EC2 instance using Terraform."

---

## Recommended build sequence

Don't try to create everything at once. Build it in these **12 phases**:

```text
PHASE 1   GitHub repository
    ↓
PHASE 2   Spring Boot application
    ↓
PHASE 3   Docker
    ↓
PHASE 4   ECR
    ↓
PHASE 5   VPC + networking
    ↓
PHASE 6   EKS
    ↓
PHASE 7   Kubernetes deployment
    ↓
PHASE 8   RDS + Redis
    ↓
PHASE 9   Secrets + IAM
    ↓
PHASE 10  GitHub Actions CI/CD
    ↓
PHASE 11  Monitoring + security
    ↓
PHASE 12  Failure testing + DR + cost optimization
```

AWS itself provides Terraform-based EKS examples and an EKS learning path, so this approach also maps well to current AWS practices. ([AWS Documentation][4])

**I recommend we build this incrementally rather than dumping 50 Terraform files at once.** The next step should be **Phase 1 + Phase 2: create the complete GitHub repository structure and a working Java 17/Spring Boot Order Management API**, followed by the Dockerfile; then we can build the Terraform modules one by one and actually deploy them to AWS.

[1]: https://docs.aws.amazon.com/eks/latest/best-practices/introduction.html?utm_source=chatgpt.com "Amazon EKS Best Practices Guide - Amazon EKS"
[2]: https://docs.aws.amazon.com/eks/latest/best-practices/autosecure.html?utm_source=chatgpt.com "EKS Auto Mode - Security - Amazon EKS"
[3]: https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html?utm_source=chatgpt.com "Create an Amazon VPC for your Amazon EKS cluster - Amazon EKS"
[4]: https://docs.aws.amazon.com/eks/latest/userguide/ml-cluster-setup-tf.html?utm_source=chatgpt.com "Set up Amazon EKS cluster for AI/ML workloads using Terraform - Amazon EKS"
[5]: https://docs.aws.amazon.com/eks/latest/userguide/create-auto.html?utm_source=chatgpt.com "Create a cluster with Amazon EKS Auto Mode - Amazon EKS"
[6]: https://docs.aws.amazon.com/eks/latest/userguide/auto-security.html?utm_source=chatgpt.com "Security considerations for Amazon EKS Auto Mode - Amazon EKS"
