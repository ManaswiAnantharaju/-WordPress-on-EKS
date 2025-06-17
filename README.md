> **Focus:** Container orchestration, IaC, CI/CD & cost-aware scaling on AWS

## 🗺️ Architecture Overview

```mermaid
flowchart LR
    subgraph AWS
        direction TB
        EKS[EKS Control Plane]:::control
        ALB((AWS<br/>Network/ALB)):::lb
        style ALB fill:#fef9ef,stroke:#e4ba55,stroke-width:2px
        subgraph "Managed Node Group (t3.medium × 3)"
            Node1((Node 1)):::node
            Node2((Node 2)):::node
            Node3((Node 3)):::node
        end
        HPA[Horizontal Pod<br/>Autoscaler]:::hpa
    end
    classDef control fill:#e1effe,stroke:#60a5fa
    classDef node fill:#f0fdf4,stroke:#4ade80
    classDef hpa fill:#ecfeff,stroke:#06b6d4
    classDef lb  fill:#fefce8,stroke:#eab308
    ALB -->|HTTP 80| Node1 & Node2 & Node3
    Node1 & Node2 & Node3 -->|WordPress Pods| HPA

##Quick-Start (re-deploy in any AWS account)
-Requires: macOS/Linux, Docker, eksctl ≥ 0.210, kubectl, Helm ≥ 3.
-Replace <region> with your preferred region.

# 1. Create the cluster (≈15 min)
eksctl create cluster \
  --name enterprise-cluster \
  --version 1.27 \
  --region <region> \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 --nodes-min 1 --nodes-max 4 --managed

# 2. Deploy WordPress via Helm
helm dependency update my-microservice
helm install my-microservice ./my-microservice

# 3. Apply auto-scaling
kubectl apply -f hpa.yaml

# 4. Grab the Load Balancer address
kubectl get svc my-microservice -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

Open the hostname in a browser → WordPress welcome page.
Run a quick load test if you want to watch HPA in action:

kubectl run siege --rm -i --tty \
  --image yokogawa/siege --restart=Never -- \
  siege -c 20 -t 2m http://<load-balancer-DNS>



