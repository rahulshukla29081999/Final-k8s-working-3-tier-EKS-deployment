🚀 Three-Tier Application Deployment on AWS EKS

This guide walks you through setting up an Amazon EKS cluster on AWS and deploying a three-tier application with an AWS Load Balancer Controller.

📌 Prerequisites

AWS Account (Free Tier recommended)

Basic understanding of Linux & Kubernetes

SSH key pair

Ubuntu EC2 instance

🟢 Step 1: IAM Configuration

Create an IAM user named:

eks-admin

Attach policy:

AdministratorAccess

Generate:

Access Key

Secret Access Key

Save credentials securely.

🟢 Step 2: EC2 Setup

Launch an Ubuntu EC2 instance

Region: us-west-2

Instance Type: t2.medium (recommended)

SSH into the instance:

ssh -i your-key.pem ubuntu@your-ec2-public-ip
🟢 Step 3: Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure

Provide:

Access Key

Secret Key

Region: us-west-2

Output: json

🟢 Step 4: Install Docker
sudo apt-get update
sudo apt install docker.io -y
docker ps
sudo chown $USER /var/run/docker.sock
🟢 Step 5: Install kubectl
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
🟢 Step 6: Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
🟢 Step 7: Setup EKS Cluster

Create cluster:

eksctl create cluster \
--name three-tier-cluster \
--region us-west-2 \
--node-type t2.medium \
--nodes-min 2 \
--nodes-max 2

Update kubeconfig:

aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster

Verify:

kubectl get nodes
🟢 Step 8: Run Kubernetes Manifests
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
🟢 Step 9: Install AWS Load Balancer IAM Policy

Download policy:

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

Create policy:

aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json

Associate IAM OIDC provider:

eksctl utils associate-iam-oidc-provider \
--region=us-west-2 \
--cluster=three-tier-cluster \
--approve

Create IAM Service Account:

eksctl create iamserviceaccount \
--cluster=three-tier-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--approve \
--region=us-west-2
🟢 Step 10: Deploy AWS Load Balancer Controller

Install Helm:

sudo snap install helm --classic

Add repo:

helm repo add eks https://aws.github.io/eks-charts
helm repo update eks

Install controller:

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=three-tier-cluster \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller

Verify:

kubectl get deployment -n kube-system aws-load-balancer-controller

Apply Load Balancer manifest:

kubectl apply -f full_stack_lb.yaml
🧹 Cleanup (Important to Avoid Charges)
Delete EKS Cluster
eksctl delete cluster --name three-tier-cluster --region us-west-2
Additional Cleanup

Stop or Terminate EC2 instance

Delete Load Balancer from EC2 Console

Delete Security Groups created

Delete IAM policy (optional)

🎯 Architecture Overview
User → AWS Load Balancer → EKS Cluster → Pods

Frontend

Backend

Database (MySQL with PVC)

💡 Notes

Ensure your EC2 instance has sufficient permissions.

Use AWS Free Tier carefully.

Monitor AWS billing dashboard.

🔥 Author

Rahul Shukla
DevOps | Kubernetes | AWS | GitOps
