# Project Architecture

This document outlines the technical architecture of the Three-Tier Application deployed on AWS EKS using a fully automated GitHub Actions CI/CD pipeline.

## System Architecture Diagram

```mermaid
graph TD
    %% Define styles
    classDef aws fill:#FF9900,stroke:#232F3E,stroke-width:2px,color:black;
    classDef k8s fill:#326CE5,stroke:#fff,stroke-width:2px,color:white;
    classDef pipeline fill:#2088FF,stroke:#fff,stroke-width:2px,color:white;
    
    User([End User]) -->|HTTP Access| FrontendLB

    subgraph AWS Environment
        FrontendLB[AWS Classic LoadBalancer<br/>Frontend Service]:::aws
        BackendLB[AWS Classic LoadBalancer<br/>Backend Service]:::aws
        
        subgraph EKS Cluster [Amazon EKS Cluster - 1.32]
            subgraph Custom Node Group [t3.medium Instances]
                
                %% Frontend
                subgraph Frontend Tier
                    F_Pod1[Frontend Pod 1]:::k8s
                    F_Pod2[Frontend Pod 2]:::k8s
                    F_Pod3[Frontend Pod 3]:::k8s
                end
                
                %% Backend
                subgraph Backend Tier
                    B_Pod1[Backend Pod 1]:::k8s
                    B_Pod2[Backend Pod 2]:::k8s
                    B_Pod3[Backend Pod 3]:::k8s
                end
                
                %% Database
                subgraph Database Tier
                    DB_Pod[MySQL Pod]:::k8s
                end
            end
        end
        
        EBS[(AWS EBS Volume<br/>gp2 via CSI Driver)]:::aws
        ECR[Amazon ECR<br/>Container Registry]:::aws
    end

    subgraph CI/CD Pipeline
        Github[GitHub Repository]:::pipeline
        GHA[GitHub Actions]:::pipeline
    end

    %% Network Flow
    FrontendLB --> F_Pod1 & F_Pod2 & F_Pod3
    F_Pod1 & F_Pod2 & F_Pod3 -->|API Requests via Public Internet| BackendLB
    BackendLB --> B_Pod1 & B_Pod2 & B_Pod3
    B_Pod1 & B_Pod2 & B_Pod3 -->|Internal ClusterIP| DB_Pod
    DB_Pod -->|Persistent Storage| EBS

    %% CI/CD Flow
    Github -->|Push to main| GHA
    GHA -->|Build & Push Multi-Stage Images| ECR
    GHA -->|Deploy Manifests| EKS Cluster
    ECR -.->|Pull Image| EKS Cluster
```

## Architecture Components

### 1. The CI/CD Pipeline (GitHub Actions)
- **Trigger**: Automatically triggered on pushes to the `main` branch.
- **Build Phase**: Uses highly optimized **multi-stage Dockerfiles** based on `alpine` to reduce container footprint by ~40%, speeding up the build process and reducing network overhead.
- **Registry Phase**: Images are securely tagged with the Git SHA and pushed to **Amazon ECR**.
- **Deploy Phase**: Updates the Kubernetes deployment manifests dynamically with the latest image tags via `sed` and applies them using `kubectl` connected to the EKS cluster.

### 2. AWS EKS Cluster & Node Group
- **Control Plane**: Fully managed by AWS (EKS version 1.32).
- **Custom Node Group**: Rather than using default node groups, the cluster is configured with a managed node group consisting of `t3.medium` instances. The nodes are deployed into private subnets for enhanced security.
- **Auto-Scaling**: Configured with a `minSize` of 1 and `maxSize` of 4, adapting dynamically to resource demands.

### 3. Frontend Tier (React)
- **Container**: Slimmed-down Nginx image serving static React build files.
- **Service**: Exposed via an AWS Classic LoadBalancer, allowing internet traffic to reach the frontend pods.
- **Configuration**: Uses an injected environment variable (`REACT_APP_API_BASE_URL`) to route API calls to the backend's LoadBalancer.

### 4. Backend Tier (Node.js)
- **Container**: Node 18 Alpine multi-stage image.
- **Service**: Exposed via a second AWS LoadBalancer because the React frontend executes in the end user's browser, requiring the backend API to be accessible over the public internet.
- **Security**: Runs as a non-root user (`appuser`) for compliance with Kubernetes security best practices.

### 5. Database Tier (MySQL)
- **Container**: Official MySQL image.
- **State Management**: Uses the **AWS EBS CSI Driver** and dynamic volume provisioning via the `gp2` StorageClass.
- **Service**: Exposed strictly internally via a Headless or `ClusterIP` service (port 3306), ensuring it cannot be accessed directly from the internet.
- **Secrets Management**: Credentials (`MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`) are injected securely into the pod at runtime via Kubernetes Secrets and ConfigMaps.
