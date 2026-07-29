# GKE Autopilot MultiKueue DWS

This repository contains the configuration (Infrastructure as Code) to set up a multi-cluster AI scheduling engine using **GKE Autopilot** and **MultiKueue**.

The architecture consists of a central **Manager** cluster that orchestrates and routes GPU batch workloads to a fleet of **Worker** clusters. It leverages Kueue's **MultiKueue** feature and GKE Autopilot's **ComputeClasses** to provide a hardware-agnostic interface for users (e.g., data scientists), who can submit generic jobs requesting `ai-compute` without needing to target specific GPU types.

## Key Features

*   **Dynamic Workload Routing**: Automatically dispatches jobs to available worker clusters (e.g., failing over between T4 and L4 GPU clusters) based on resource availability, maximizing throughput without external load balancers.
*   **Cost-Optimized GPU Provisioning**: Instructs GKE Autopilot to provision resources dynamically, prioritizing procurement in this order: **Reservations -> Dynamic Workload Scheduler (FlexStart) -> Spot VMs**.
*   **Targeted Priority Queues**: Supports dedicated queues (e.g., `l4-premium-queue`) to restrict high-priority workloads to specific, high-performance hardware.
*   **Decoupled Policy Enforcement**: Allows the Manager cluster to accept job submissions without enforcing strict workload-level security policies (like Autopilot's Warden webhooks), offloading enforcement to the Worker clusters where the workloads actually run.

---

## Folder Structure

This repository is organized for **GitOps (e.g., GKE Config Sync)**. Infrastructure state is separated into directory-based RootSyncs to ensure each cluster only pulls its specific configuration, while workloads are kept separate for CI/CD deployment.

```text
.
├── clusters/
│   ├── manager/
│   │   ├── compute-class.yaml            # Template compute class
│   │   ├── resource-flavors.yaml         # Reference flavors for workers
│   │   ├── kueue-config.yaml             # Local Kueue controller manager configuration
│   │   ├── multikueue-setup.yaml         # Connection setup to worker clusters
│   │   └── queues.yaml                   # Global dispatcher queues and routing logic
│   ├── worker1/
│   │   ├── compute-class.yaml            # Translates 'ai-compute' to T4 GPU procurement logic
│   │   ├── resource-flavor.yaml          # Maps local T4 GPU flavors
│   │   └── queues.yaml                   # Defines worker-specific queues and quotas
│   └── worker2/
│       ├── compute-class.yaml            # Translates 'ai-compute' to L4 GPU procurement logic
│       ├── resource-flavor.yaml          # Maps local L4 GPU flavors
│       └── queues.yaml                   # Defines worker-specific queues (standard and premium) and quotas
└── workloads/
    ├── sample-gpu-job.yaml               # Standard polymorphic job (allows T4/L4 failover)
    └── premium-gpu-job.yaml              # Strict priority job (locked to L4 premium queue)
```
