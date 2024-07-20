# Helm Installation and Getting Started Guide

## Introduction to Helm

Helm is a package manager for Kubernetes that simplifies the deployment and management of applications on Kubernetes clusters. It allows you to define, install, and upgrade even the most complex Kubernetes applications using charts, which are collections of pre-configured Kubernetes resources.

## Prerequisites

- A running Kubernetes cluster
- `kubectl` installed and configured to communicate with your cluster

## Installing Helm

### Step 1: Download Helm Binary

For Windows, you can download the Helm binary from the official Helm releases page:

```sh
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Step 2: Extract and Move Binary

Extract the downloaded file and move it to a directory in your system's PATH. For example:

```sh
tar -zxvf helm-v3.7.0-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```

### Step 3: Verify Installation

Check if Helm is installed correctly by running:

```sh
helm version
```

## Getting Started with Helm

### Step 4: Add a Chart Repository

Add the official Helm eks charts repository:

```sh
helm repo add eks https://aws.github.io/eks-charts

```

### Step 5: Update Repositories

Update the repositories to get the latest charts:

```sh
helm repo update eks
```

### Step 6: Create a Domain in AWS Route 53

1. **Open Route 53 Console:**
   - Navigate to the [Route 53 Console](https://console.aws.amazon.com/route53/home).
   
2. **Create a Hosted Zone:**
   - Click on "Create hosted zone".
   - Enter the domain name you want to use (e.g., `hilltop.com`).
   - Choose the type as "Public hosted zone".
   - Click on "Create".

3. **Note the Nameservers:**
   - After creating the hosted zone, note the nameservers provided by Route 53.
   - Update your domain registrar with these nameservers to point your domain to Route 53.

### Step 7: Request an SSL Certificate using AWS Certificate Manager (ACM)

1. **Open ACM Console:**
   - Navigate to the [ACM Console](https://console.aws.amazon.com/acm/home).
   
2. **Request a Certificate:**
   - Click on "Request a certificate".
   - Choose "Request a public certificate".
   - Enter your domain name (e.g., `hilltop.com`).
   - Optionally, add additional names like `www.hilltop.com`.
   - Click "Next".

3. **Validate the Domain:**
   - Choose DNS validation (recommended).
   - ACM will provide DNS records that you need to add to your Route 53 hosted zone.
   - After adding the DNS records, ACM will validate your domain and issue the certificate.

### Step 8: Set Up the IAM Role and Policy for ALB

1. **Create IAM Policy:**
   ```bash
   curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
   aws iam create-policy \
     --policy-name AWSLoadBalancerControllerIAMPolicy \
     --policy-document file://iam_policy.json
   ```

2. **Create IAM Role:**
   ```bash
   eksctl create iamserviceaccount \
     --cluster=eks-hilltop-dev \
     --namespace=kube-system \
     --name=aws-load-balancer-controller \
     --role-name AmazonEKSLoadBalancerController \
     --attach-policy-arn=arn:aws:iam::YOUR_ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
     --approve
   ```

### Step 9: Install the AWS Load Balancer Controller

1. **Add the EKS Helm Repository:**
   ```bash
   helm repo add eks https://aws.github.io/eks-charts
   helm repo update
   ```

2. **Install the AWS Load Balancer Controller:**
   ```bash
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
     -n kube-system \
     --set clusterName=eks-hilltop-dev \
     --set serviceAccount.create=false \
     --set serviceAccount.name=aws-load-balancer-controller
   ```

3. **Apply the CRDs:**
   ```bash
   kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"
   ```

### Step 10: Deploy Your Application with Ingress using Helm Chart

Create a Helm chart named `hilltop`:

```bash
helm create hilltop
```

#### 9 Modify the Deployment YAML

### `values.yaml`

### `templates/ingress.yaml`

### `templates/service.yaml`

### Deploy the Application:
```bash
helm upgrade --install hilltop . -f values.yaml
```

### Notes:

- Ensure you replace placeholders like `YOUR_ACCOUNT_ID`, `REGION`, and `CERTIFICATE_ID` with actual values.
- The AWS Load Balancer Controller will automatically create an ALB when the Ingress resource is applied. You don't need to create the ALB manually.


