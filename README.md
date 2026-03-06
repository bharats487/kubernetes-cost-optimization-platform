# Kubernetes Cost Optimization Platform

[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.29+-blue?logo=kubernetes)](https://kubernetes.io/)
[![Kubecost](https://img.shields.io/badge/Cost-Kubecost-green)](https://kubecost.com/)
[![KEDA](https://img.shields.io/badge/Autoscaling-KEDA-blue)](https://keda.sh/)
[![Goldilocks](https://img.shields.io/badge/VPA-Goldilocks-yellow)](https://goldilocks.docs.fairwinds.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> Production Kubernetes FinOps and cost optimization platform using Kubecost, Goldilocks (VPA), KEDA event-driven autoscaling, Karpenter, and Spot instance automation. Provides multi-cluster cost visibility, rightsizing recommendations, namespace budget enforcement, and automated cost-saving actions for AWS EKS and Azure AKS.

---

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                 KUBERNETES COST OPTIMIZATION PLATFORM                │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                   COST VISIBILITY LAYER                        │ │
│  │                                                                │ │
│  │  Kubecost (Multi-Cluster)                                      │ │
│  │  ├── Namespace cost allocation                                 │ │
│  │  ├── Workload-level cost breakdown                             │ │
│  │  ├── Shared infrastructure cost distribution                  │ │
│  │  └── Savings insights & rightsizing                           │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                OPTIMIZATION LAYER                              │ │
│  │                                                                │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │ │
│  │  │  Goldilocks  │  │    KEDA      │  │     Karpenter        │ │ │
│  │  │  VPA Recs    │  │  Event-driven│  │  Node provisioning   │ │ │
│  │  │  CPU/Mem     │  │  Autoscaling │  │  Spot + On-demand    │ │ │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                ENFORCEMENT & REPORTING                         │ │
│  │                                                                │ │
│  │  Budget Policies (Kyverno) │ Slack Alerts │ Grafana Dashboard │ │
│  │  Automated Rightsizing     │ Cost Reports │ Chargeback CSV    │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Features

- **Multi-Cluster Cost Visibility** – Kubecost aggregated across EKS and AKS with shared cost allocation
- **Namespace Budget Enforcement** – Kyverno policies blocking deployments that exceed namespace cost budget
- **VPA Rightsizing** – Goldilocks VPA recommendations with automated resource request/limit updates
- **KEDA Autoscaling** – Event-driven scaling based on SQS, Service Bus, Kafka, and custom Prometheus metrics
- **Karpenter Node Provisioning** – Intelligent node provisioning with EC2 Spot interruption handling
- **Spot Instance Automation** – Multi-AZ Spot workloads with automatic fallback to On-Demand
- **Cost Anomaly Detection** – ML-based cost anomaly alerts via Kubecost + CloudWatch Anomaly Detection
- **Chargeback & Showback** – CSV/API-based team cost reports with label-based allocation
- **Idle Resource Cleanup** – Automated cleanup of unused PVCs, LoadBalancers, and stopped workloads
- **Grafana FinOps Dashboards** – Real-time cost dashboards showing spend by team, namespace, and service

---

## Repository Structure

```
kubernetes-cost-optimization-platform/
├── README.md
├── .gitignore
│
├── platform/
│   ├── kubecost/
│   │   ├── values.yaml             # Kubecost Helm values
│   │   ├── multi-cluster-config.yaml
│   │   └── cost-allocation-config.yaml
│   │
│   ├── goldilocks/
│   │   ├── values.yaml             # Goldilocks Helm values
│   │   └── vpa-crds.yaml
│   │
│   ├── keda/
│   │   ├── values.yaml             # KEDA Helm values
│   │   └── scaled-objects/
│   │       ├── sqs-scaler.yaml     # AWS SQS ScaledObject
│   │       ├── kafka-scaler.yaml   # Kafka ScaledObject
│   │       └── servicebus-scaler.yaml
│   │
│   └── karpenter/
│       ├── values.yaml             # Karpenter Helm values
│       ├── nodepool.yaml           # NodePool definition
│       ├── nodeclass-spot.yaml     # EC2NodeClass for Spot
│       └── nodeclass-ondemand.yaml
│
├── policies/
│   ├── namespace-budget.yaml       # Kyverno: namespace cost limits
│   ├── require-resource-limits.yaml # Kyverno: mandatory resource limits
│   └── label-enforcement.yaml      # Kyverno: cost allocation labels
│
├── dashboards/
│   ├── finops-overview.json        # Grafana FinOps dashboard
│   ├── namespace-cost.json         # Per-namespace cost trends
│   └── savings-insights.json       # Optimization opportunities
│
├── automation/
│   ├── rightsizing/
│   │   └── apply-vpa-recommendations.py
│   ├── cleanup/
│   │   └── remove-idle-resources.py
│   └── reports/
│       └── generate-chargeback.py  # Weekly team cost reports
│
├── .github/
│   └── workflows/
│       ├── validate-manifests.yml
│       └── cost-report.yml         # Weekly cost reporting
│
└── docs/
    ├── getting-started.md
    ├── karpenter-guide.md
    ├── chargeback-setup.md
    └── runbooks/
        ├── cost-spike.md
        └── rightsizing.md
```

---

## Quick Start

```bash
# Install Kubecost
helm repo add kubecost https://kubecost.github.io/cost-analyzer
helm install kubecost kubecost/cost-analyzer \
  --namespace kubecost --create-namespace \
  --values platform/kubecost/values.yaml

# Install Goldilocks (VPA)
helm repo add fairwinds-stable https://charts.fairwinds.com/stable
helm install goldilocks fairwinds-stable/goldilocks \
  --namespace goldilocks --create-namespace

# Enable VPA recommendations for a namespace
kubectl label namespace production goldilocks.fairwinds.com/enabled=true

# Install KEDA
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda --namespace keda --create-namespace

# Install Karpenter (EKS)
helm install karpenter oci://public.ecr.aws/karpenter/karpenter \
  --version "0.35.0" \
  --namespace karpenter --create-namespace \
  --values platform/karpenter/values.yaml

# Apply policies
kubectl apply -f policies/

# Generate monthly cost report
python automation/reports/generate-chargeback.py \
  --period 2026-02 --output chargeback-feb-2026.csv

# Access Kubecost UI
kubectl port-forward -n kubecost svc/kubecost-cost-analyzer 9090:9090
```

---

## Cost Savings Targets

| Optimization | Estimated Savings |
|-------------|------------------|
| Right-size over-provisioned pods | 20-40% |
| Spot instances for non-critical workloads | 60-80% |
| KEDA scale-to-zero for batch jobs | 100% idle time |
| Remove unused PVCs and Load Balancers | $50-500/month |
| Reserved/Savings Plans via recommendations | 30-50% |

---

## GitHub CLI — Create & Upload

```bash
gh repo create kubernetes-cost-optimization-platform \
  --public \
  --description "Kubernetes FinOps platform with Kubecost, Goldilocks VPA, KEDA, Karpenter, Spot automation, and namespace budget enforcement for EKS and AKS"

gh repo edit bharats487/kubernetes-cost-optimization-platform \
  --add-topic kubernetes \
  --add-topic finops \
  --add-topic kubecost \
  --add-topic keda \
  --add-topic karpenter \
  --add-topic aws \
  --add-topic cost-optimization \
  --add-topic devops

git init && git checkout -b main
git add . && git commit -m "feat: Kubernetes cost optimization platform"
git remote add origin https://github.com/bharats487/kubernetes-cost-optimization-platform.git
git push -u origin main
```

---

## License

MIT License — see [LICENSE](LICENSE)
