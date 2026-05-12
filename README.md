# 🚗 AutoElite — EKS Cluster (Terraform)

Terraform infrastructure for the **car-project-microservices** EKS cluster on AWS `ap-south-1`.

---

## 📁 Project Structure

```
.
├── main.tf                      # Root module — wires VPC, SG, IAM, EKS
├── provider.tf                  # AWS provider (ap-south-1)
├── variables.tf                 # Input variable declarations
├── terraform.tfvars             # Cluster config (name, region, AMI, K8s version)
├── application-deployment.yaml  # K8s manifests — car-backend & car-frontend
├── dev.sh                       # Helper: fetch EKS addon versions
├── dev.json                     # Addon version reference output
└── modules/
    ├── vpc/                     # VPC, subnets, IGW, route tables
    ├── security-group/          # Worker node security group
    ├── iam/                     # Master & worker IAM roles + autoscaler policy
    └── eks/                     # EKS cluster, launch template, node group, addons
```

---

## ⚙️ Configuration (terraform.tfvars)

| Variable          | Value                      |
|-------------------|----------------------------|
| `cluster_name`    | `autoelite-cluster`        |
| `region`          | `ap-south-1`               |
| `instance_size`   | `t3.medium`                |
| `instance_count`  | `2`                        |
| `cluster_version` | `1.32`                     |

---

## 🚀 Deploy Order

```bash
# 1. Initialise & apply Terraform
terraform init
terraform plan
terraform apply

# 2. Connect kubectl to the new cluster
aws eks update-kubeconfig --name autoelite-cluster --region ap-south-1

# 3. Create ECR repositories
aws ecr create-repository --repository-name car-backend  --region ap-south-1
aws ecr create-repository --repository-name car-frontend --region ap-south-1

# 4. Apply K8s manifests (replace <AWS_ACCOUNT_ID> first)
kubectl apply -f application-deployment.yaml

# 5. Get LoadBalancer endpoints
kubectl get svc -n autoelite
```

---

## 🔄 GitOps CD Flow

```
Jenkins CI  →  build image  →  push to ECR
Jenkins CD  →  update k8s/backend-deployment.yaml in car-project-microservices repo
ArgoCD      →  detects git change  →  syncs to autoelite-cluster automatically
```

---

## 🔁 Rollback

```bash
kubectl rollout undo deployment/car-backend  -n autoelite
kubectl rollout undo deployment/car-frontend -n autoelite

# Rollback to a specific revision
kubectl rollout undo deployment/car-backend --to-revision=2 -n autoelite

# Check rollout history
kubectl rollout history deployment/car-backend -n autoelite
```

---

## 🏷️ AWS Resource Naming

| Resource                   | Name                                    |
|----------------------------|-----------------------------------------|
| EKS Cluster                | `autoelite-cluster`                     |
| VPC                        | `autoelite-cluster-vpc`                 |
| Internet Gateway           | `autoelite-cluster-igw`                 |
| Security Group             | `autoelite-cluster-EKS-security-group`  |
| IAM Master Role            | `autoelite-EKS-Master`                  |
| IAM Worker Role            | `autoelite-eks-worker`                  |
| IAM Autoscaler Policy      | `autoelite-eks-autoscaler-policy`       |
| Worker Instance Profile    | `autoelite-EKS-worker-nodes-profile`    |
| Launch Template            | `autoelite-worker-node-launch-template` |
| Node Group                 | `autoelite-Worker-Node-Group`           |
| K8s Namespace              | `autoelite`                             |
| ECR Repo (backend)         | `car-backend`                           |
| ECR Repo (frontend)        | `car-frontend`                          |

---

## 🔗 Related Repos

- **App + K8s manifests**: [car-project-microservices](https://github.com/puttamoses/car-project-microservices)
- **This repo**: [AWS-EKS-CREATE-AND-UPDATE-BY-TERRAFORM](https://github.com/puttamoses/AWS-EKS-CREATE-AND-UPDATE-BY-TERRAFORM)
