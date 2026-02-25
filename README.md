# 🚀 Three-Tier Application Deployment on AWS EKS

This project demonstrates how to set up an Amazon EKS cluster and deploy a three-tier application using AWS Load Balancer Controller.

---

## 📌 Prerequisites

- AWS Account (Free Tier recommended)
- IAM User with AdministratorAccess
- Ubuntu EC2 Instance
- SSH Key Pair

---

# 🟢 Step 1: IAM Configuration

1. Create an IAM user named:
   ```
   eks-admin
   ```

2. Attach Policy:
   ```
   AdministratorAccess
   ```

3. Generate:
   - Access Key
   - Secret Access Key

Save credentials securely.

---

# 🟢 Step 2: EC2 Setup

1. Launch an Ubuntu EC2 instance  
   - Region: `us-west-2`
   - Instance Type: `t2.medium`

2. SSH into instance:

```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

---

# 🟢 Step 3: Install AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

Provide:
- Access Key
- Secret Key
- Region: us-west-2
- Output: json

---

# 🟢 Step 4: Install Docker

```bash
sudo apt-get update
sudo apt install docker.io -y
docker ps
sudo chown $USER /var/run/docker.sock
```

---

# 🟢 Step 5: Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

---

# 🟢 Step 6: Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

# 🟢 Step 7: Setup EKS Cluster

Create cluster:

```bash
eksctl create cluster \
--name three-tier-cluster \
--region us-west-2 \
--node-type t2.medium \
--nodes-min 2 \
--nodes-max 2
```

Update kubeconfig:

```bash
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
```

Verify nodes:

```bash
kubectl get nodes
```

---

# 🟢 Step 8: Run Kubernetes Manifests

```bash
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
```

---

# 🟢 Step 9: Install AWS Load Balancer IAM Policy

Download policy:

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create policy:

```bash
aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json
```

Associate OIDC provider:

```bash
eksctl utils associate-iam-oidc-provider \
--region=us-west-2 \
--cluster=three-tier-cluster \
--approve
```

Create IAM Service Account:

```bash
eksctl create iamserviceaccount \
--cluster=three-tier-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--approve \
--region=us-west-2
```

Replace `<YOUR_ACCOUNT_ID>` with your actual AWS account ID.

---

# 🟢 Step 10: Deploy AWS Load Balancer Controller

Install Helm:

```bash
sudo snap install helm --classic
```

Add repo:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Install controller:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=three-tier-cluster \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller
```

Verify:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

Deploy Load Balancer:

```bash
kubectl apply -f full_stack_lb.yaml
```

---

# 🧹 Cleanup (Important to Avoid Charges)

Delete cluster:

```bash
eksctl delete cluster --name three-tier-cluster --region us-west-2
```

Then:

- Stop or terminate EC2 instance
- Delete Load Balancer from EC2 console
- Delete security groups created
- Delete IAM policy (optional)

---

# 🏗 Architecture

```
User → AWS Load Balancer → EKS Cluster → Pods
```

Three-Tier:
- Frontend
- Backend
- Database (MySQL with PVC)

---

# 👨‍💻 Author

Rahul Shukla  
DevOps | Kubernetes | AWS | GitOps
