# Ansible EKS Automation Suite (CentOS)

This repository demonstrates a production-style Ansible automation project that:
- Baselines CentOS hosts (packages, SSH hardening, firewall)
- Installs **Prometheus Node Exporter** as a **systemd** service on CentOS nodes
- Deploys microservices to **EKS/Kubernetes** using `kubernetes.core`
- Optionally installs **kube-prometheus-stack** (Prometheus + Grafana) on the cluster
- Includes GitHub Actions CI (`ansible-lint`, `yamllint`, syntax check)

## Prerequisites
- An Ansible control node (CentOS/RHEL) with network access to your managed nodes
- SSH access from control node → managed nodes
- A working Kubernetes kubeconfig on the control node (default: `/root/.kube/config`)
- Docker images already pushed (example):
  - `DOCKERHUB_USER/user-service:1.0`
  - `DOCKERHUB_USER/order-service:1.0`

## Quick Start

### 1) Install Ansible collections
```bash
ansible-galaxy collection install -r requirements.yml
```

### 2) Edit inventory
Update placeholders in:
- `inventories/dev/hosts.ini`
- `inventories/dev/group_vars/all.yml`

### 3) Connectivity test
```bash
ansible -i inventories/dev/hosts.ini centos_nodes -m ping
```

### 4) Run Linux baseline + Node Exporter
```bash
ansible-playbook -i inventories/dev/hosts.ini playbooks/linux.yml
```

### 5) Deploy to Kubernetes (EKS)
```bash
ansible-playbook -i inventories/dev/hosts.ini playbooks/k8s.yml
```

### 6) Full run (baseline + deploy)
```bash
ansible-playbook -i inventories/dev/hosts.ini playbooks/site.yml
```

## What you get

### Linux Baseline
- Installs common tools
- Hardens SSH configuration using a template
- Enables `firewalld` and opens ports (SSH + Node Exporter)

### Monitoring
- Installs **Prometheus Node Exporter**
- Creates a **systemd** unit and enables the service
- Exposes metrics on `:9100`

### Kubernetes Deployment
- Namespace: `micro`
- `user-service` → ClusterIP on port 5000
- `order-service` → LoadBalancer on port 80 → targetPort 5000
- Liveness/readiness probes: `/healthz` and `/readyz`

## Notes
- If your Docker Hub repositories are private, create an `imagePullSecret` in the namespace and reference it in your deployments.
- This project is designed to be extended (Ingress + ALB, HPA, ExternalDNS, etc.).

## License
MIT
