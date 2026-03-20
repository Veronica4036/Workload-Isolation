# EKS Auto Mode — Workload Isolation with Karpenter NodePools

Custom Karpenter NodePool architecture for EKS Auto Mode that isolates platform infrastructure from application workloads using taints and tolerations, managed via the EKS Capability for Argo CD.

## Architecture

```
┌─────────────────────────────────────────────────┐
│                 EKS Auto Mode                    │
│                                                  │
│  ┌─────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ System   │  │ Platform │  │ Application   │  │
│  │ Pool     │  │ Pool     │  │ Pool          │  │
│  │(managed) │  │(tainted) │  │(open)         │  │
│  │          │  │          │  │               │  │
│  │ CoreDNS  │  │ Kyverno  │  │ Tenant apps   │  │
│  │ kube-    │  │ CertMgr  │  │ schedule here │  │
│  │ proxy    │  │ ExtDNS   │  │ by default    │  │
│  │          │  │ ExtSec   │  │               │  │
│  └─────────┘  └──────────┘  └───────────────┘  │
│                                                  │
│         Managed Argo CD (AWS control plane)      │
│         syncs from this Git repo                 │
└─────────────────────────────────────────────────┘
```

## Repository Structure

```
nodepools/
  platform-nodepool.yaml    # Tainted NodePool for infrastructure add-ons
  application-nodepool.yaml # Open NodePool for tenant workloads
argocd/
  application.yaml          # Argo CD Application resource
examples/
  platform-placement.yaml   # Reference: nodeSelector + toleration for platform pods
```

## Quick Start

1. Update `argocd/application.yaml` with your Git repo URL.
2. Push this repo to GitHub.
3. Apply the Argo CD Application:
   ```bash
   kubectl apply -f argocd/application.yaml
   ```
4. Argo CD syncs the NodePools to your cluster automatically.

## Platform Add-On Placement

Any pod that should run on the platform pool needs:

```yaml
nodeSelector:
  workload-type: platform
tolerations:
  - key: workload-type
    value: platform
    effect: NoSchedule
```

See `examples/platform-placement.yaml` for a full reference.
