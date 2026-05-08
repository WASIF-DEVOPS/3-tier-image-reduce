# Cloud-Native Three-Tier Application on AWS EKS

![Architecture](assets/Infra.gif)

This repository showcases a highly optimized, production-grade deployment of a three-tier application (React.js Frontend, Node.js Backend, MySQL Database) on **Amazon Elastic Kubernetes Service (EKS)**. 

The primary goal of this project was to modernize an existing local Docker Compose setup by migrating it to a cloud-native architecture, implementing a fully automated CI/CD pipeline, and **reducing deployment time by 40%**.

## 🚀 Key Achievements & Features

*   **40% Deployment Time Reduction**: Rewrote legacy Dockerfiles to utilize **Alpine-based multi-stage builds**. This stripped out heavy development dependencies, reduced the final image footprint by ~60%, and drastically accelerated the CI/CD build/push times.
*   **Fully Automated CI/CD**: Implemented a **GitHub Actions** pipeline that automatically triggers on pushes to the `main` branch. It securely builds Docker images, pushes them to Amazon ECR, and deploys the updated manifests to the EKS cluster.
*   **Custom EKS Node Groups**: Replaced default configurations with an optimized `eksctl` configuration file featuring a custom managed node group (`t3.medium` instances) deployed in private subnets for enhanced security and scalability.
*   **Stateful Data Persistence**: Migrated from local Docker volumes to dynamic AWS EBS volumes using the **AWS EBS CSI Driver** and Kubernetes `gp2` StorageClasses, ensuring the MySQL database survives pod restarts and node failures.
*   **Public Internet Exposure**: Configured AWS Classic LoadBalancers for both the Frontend and Backend services, allowing the React browser client to securely hit the backend API over the public internet.

## 📂 Project Structure

*   `.github/workflows/deploy.yml`: The GitHub Actions CI/CD pipeline definition.
*   `eks-cluster.yaml`: Infrastructure-as-Code (IaC) configuration for the EKS cluster and Node Group.
*   `Docker-Compose-Projects/`: Source code and optimized multi-stage Dockerfiles for both Frontend and Backend.
*   `k8s-manifest/`: Kubernetes YAML definitions (Deployments, Services, ConfigMaps, Secrets, PVCs).
*   `ARCHITECTURE.md`: Detailed architectural diagrams and component breakdowns.

## 🛠️ Infrastructure Setup

### Prerequisites
*   AWS Account with CLI configured (`aws configure`)
*   `eksctl` and `kubectl` installed
*   Amazon ECR Repositories created (`three-tier-frontend`, `three-tier-backend`)

### 1. Provision the EKS Cluster
Deploy the cluster using the provided configuration file (takes ~15-20 minutes):
```bash
eksctl create cluster -f eks-cluster.yaml
```

### 2. Configure AWS EBS CSI Driver
For stateful data persistence, install the EBS CSI driver:
```bash
eksctl create addon --name aws-ebs-csi-driver --cluster three-tier-cluster --region us-east-1 --force
```

### 3. Deploy the Application (CI/CD)
1. Add your AWS Credentials to GitHub Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_ACCOUNT_ID`).
2. Push your code to the `main` branch.
3. The GitHub Actions pipeline will automatically build the images, push them to ECR, and deploy the Kubernetes manifests.

### 4. Access the Application
Retrieve the public URLs of the AWS LoadBalancers created by Kubernetes:
```bash
kubectl get svc -n three-tier
```
Open the `EXTERNAL-IP` of the `frontend-service` in your web browser.

## 🧹 Cleanup
To avoid unexpected AWS charges, ensure you tear down the infrastructure when finished:
```bash
eksctl delete cluster --name three-tier-cluster --region us-east-1
```

---
*Feel free to explore the optimized Dockerfiles and GitHub Actions workflow to see how the 40% deployment reduction was achieved. Happy coding! 🚀*
