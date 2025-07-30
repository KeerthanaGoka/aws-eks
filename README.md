
# EKS Shopping Platform with Microservices, Observability & Service Mesh

This repository provisions a robust AWS EKS infrastructure and deploys a production-like shopping application with 11 microservices. The platform includes:

- A secure, highly-available **EKS cluster** with managed node groups
- Networking resources (VPC, subnets, security groups) configured for best practices
- Cluster autoscaling with **Karpenter**
- Istio **service mesh** for traffic management and observability
- Centralized monitoring with **Prometheus & Grafana**
- **GitOps workflow** with Argo CD for continuous delivery

All infrastructure components are automated using **Terraform**, enabling reproducible and scalable deployments.

This setup provides a complete Kubernetes platform for running microservices with end-to-end observability, security, and CI/CD capabilities.

## EKS Platform Deployment Guide

### Prerequisites

- **Terraform** >= 1.3.0  
- **kubectl**  
- **helm**  
- **AWS CLI** (with credentials configured)  

---

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/KeerthanaGoka/aws-eks.git
cd aws-eks/terraform
```

### Provision AWS Infrastructure

```bash
cd terraform
terraform init
terraform apply
kubectl rollout restart deployment istio-ingress -n istio-ingress
kubectl apply -f manifests/karpenter.yaml
```
### Terraform Deployment Summary

From the `main.tf`, the following key components are provisioned:

- **EKS Cluster (v1.30)** with a managed node group running Bottlerocket OS for Karpenter
- **Karpenter autoscaler** for dynamic, just-in-time compute provisioning
- **Istio Service Mesh** deployed via the EKS Blueprints Addons module:
  - Istio Ingress Gateway  
  - Istiod  
  - Istio base components
- **Argo CD** for GitOps-based continuous deployment
- **Helm releases** for `kube-state-metrics` and `karpenter`
- **Network rules** to support Istio sidecar injection via ports `15012` and `15017`
- **Modular and production-ready infrastructure** following AWS best practices

This setup enables a scalable, observable, and secure Kubernetes platform with automated provisioning and continuous delivery.



### Networking & IAM Overview

This project provisions a secure and scalable AWS infrastructure using Terraform:

- **VPC**  
  A single VPC with CIDR `10.0.0.0/16` is created to isolate all Kubernetes resources.

- **Subnets**  
  A total of **6 subnets** are created across **3 Availability Zones**:
  -  3 private subnets for EKS worker nodes  
  -  3 public subnets for exposing internet-facing services (e.g., Istio ingress)

- **Security Groups**
  - One security group for the EKS control plane
  - One for managed node groups, with custom ingress rules to allow Istio webhook communication on ports `15012` and `15017`

- **IAM Roles**
  - EKS cluster IAM role for control plane operations
  - Managed node group role for EKS worker nodes
  - Karpenter node IAM role for dynamic autoscaling support

These components collectively ensure secure communication, workload isolation, and scalable compute provisioning across the EKS cluster.


### Istio sidecar injection

```bash
kubectl label namespace default istio-injection=enabled
```

### Observability Addons - Prometheus & Grafana

```bash
for ADDON in prometheus grafana
do
    ADDON_URL="https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/$ADDON.yaml"
    kubectl apply --server-side -f $ADDON_URL
done
```

### Deploy Argo CD

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n argocd --create-namespace

```

### Deploy Your Website

```bash
kubectl apply -f manifests/argocd/shopping-application.yaml # Creates ArgoCD Application
kubectl apply -f manifests/shopping/frontendvs.yaml
```
Verify Istio Gateway and VirtualService are correctly configured to route external traffic to frontend service.

![Website Dashboard](./pictures/website.png)

**Table: Ports exposed by all microservices in the demo application**

| Service                | Service Port | Pod Port | Protocol     |
|------------------------|--------------|----------|--------------|
| emailservice           | 5000         | 8080     | gRPC         |
| checkoutservice        | 5050         | 5050     | gRPC         |
| recommendationservice  | 8080         | 8080     | gRPC         |
| frontend               | 80           | 8080     | HTTP         |
| paymentservice         | 50051        | 50051    | gRPC         |
| productcatalogservice  | 3550         | 3550     | gRPC         |
| cartservice            | 7070         | 7070     | gRPC         |
| redis-cart             | 6379         | 6379     | TCP (Redis)  |
| currencyservice        | 7000         | 7000     | gRPC         |
| shippingservice        | 50051        | 50051    | gRPC         |
| adservice              | 9555         | 9555     | gRPC         |
 
---

## Observability Dashboards

## ArgoCD (GitOps Dashboard)

Apps will automatically sync via Argo CD from the main branch.
You can monitor health/status from the Argo CD UI.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d # for password
```
Access ArgoCD at: [http://localhost:8080](http://localhost:8080)

![ArgoCD Dashboard](./pictures/argocd.png)

### Grafana (Metrics Dashboard)

```bash
kubectl -n monitoring port-forward svc/prometheus-grafana 3000:80
```

Access Grafana at: [http://localhost:3000](http://localhost:3000)

![Grafana Dashboard](./pictures/Grafana.png)
---

## Contributing

Contributions are welcome! Feel free to open issues or submit pull requests to improve this project.

---

## License

This project is licensed under the **MIT License**.

---

## Author

**Keerthana Goka**  
[GitHub Profile](https://github.com/KeerthanaGoka)
