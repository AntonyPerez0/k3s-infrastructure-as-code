# Bare-Metal Kubernetes: Enterprise GitOps Architecture

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-%23EF7B4D.svg?style=for-the-badge&logo=argo&logoColor=white)
![Tailscale](https://img.shields.io/badge/Tailscale-%23FFFFFF.svg?style=for-the-badge&logo=tailscale&logoColor=black)
![Rocky Linux](https://img.shields.io/badge/Rocky%20Linux-%2310B981.svg?style=for-the-badge&logo=rocky-linux&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)

## 1. Executive Summary

This repository contains the declarative infrastructure and deployment manifests for a production-grade, bare-metal Kubernetes cluster. Designed with a strict Code-First GitOps philosophy, this environment serves as a practical implementation of modern datacenter operations, continuous delivery pipelines, and automated infrastructure validation. All cluster states are driven by code, ensuring high reproducibility and eliminating configuration drift.
<img width="1680" height="710" alt="Screenshot 2026-04-08 at 5 33 32 PM" src="https://github.com/user-attachments/assets/1d7881cf-4bfc-4cee-8334-a457676cbaa8" />
<img width="1680" height="644" alt="Screenshot 2026-04-08 at 5 41 46 PM" src="https://github.com/user-attachments/assets/48ff8cea-16b3-4b5e-ab4c-37451ff3fdbf" />
<img width="1680" height="939" alt="Screenshot 2026-04-08 at 5 42 18 PM" src="https://github.com/user-attachments/assets/9962128e-4e2e-4109-8cef-8ff3cdd2f204" />

## 2. Core Technology Stack

- **Kubernetes Engine:** K3s, optimized for single-node edge computing environments.
- **Operating System:** Rocky Linux x86_64 running on dedicated Dell Optiplex hardware.
- **Automation and CI/CD:** ArgoCD for continuous delivery, Forgejo for version control, and containerized Forgejo Runners for automated pipeline execution.
- **Networking and Security:** Tailscale Kubernetes Operator for Zero-Trust overlay networking and automated TLS management.
- **Observability:** Promtail, Loki, and Grafana for centralized log aggregation, supplemented by InfluxDB for time-series data ingestion.

## 3. Continuous Delivery and Automated Reconciliation

By leveraging ArgoCD, the cluster maintains continuous synchronization with the main branch of this repository.

- **Automated Sync Policy:** The ArgoCD Root Application autonomously detects manifest changes and applies them to the cluster with self-healing capabilities enabled.
- **State Enforcement:** Resources modified manually via the command line are actively overwritten to match their declared state in version control, preventing unauthorized deviations.
- **Pipeline Execution:** Dedicated Forgejo Runners are deployed within the cluster to execute automated testing, linting, and build pipelines directly against committed code.

## 4. Tiered Storage Architecture

The cluster employs a tiered storage strategy to balance performance requirements with capacity constraints.

- **Performance Tier:** High-IOPS NVMe solid-state drives utilizing the `local-path` provisioner. This tier is strictly reserved for latency-sensitive workloads including Kubernetes system pods, databases like InfluxDB, and application state caches.
- **Capacity Tier:** High-capacity mechanical disk drives mounted via a custom `bulk-hdd` StorageClass. This tier utilizes a `Retain` reclaim policy to safely store long-term backups, metric archives, and bulk datasets without risk of accidental deletion during pod lifecycle events.

## 5. Enterprise Disaster Recovery

A fully automated, GitOps-managed disaster recovery pipeline guarantees data persistence and rapid cluster restoration.

- **Automated Snapshots:** Velero utilizes a specialized Node Agent to execute deep, block-level backups of Persistent Volume Claims within the `apps` namespace.
- **Scheduled Execution:** Complete namespace backups are scheduled via a Kubernetes cron syntax to execute daily at 3:00 AM.
- **S3-Compatible Archival:** Backup data is securely routed via internal cluster DNS to an internal MinIO S3 bucket hosted on the Capacity Tier.
- **Lifecycle Management:** Backups strictly adhere to a 168-hour Time-To-Live policy, allowing Velero to autonomously prune stale data and manage storage consumption.

## 6. Zero-Trust Security and Networking

The architecture prioritizes a secure-by-default network perimeter, mitigating traditional external attack vectors.

- **Ingress Control:** The Tailscale Kubernetes Operator acts as the sole Ingress Controller, intercepting and routing all external traffic.
- **Access Management:** Standard port forwarding configurations for HTTP and HTTPS are disabled at the edge router. System access strictly requires an authenticated Tailscale client connection.
- **Certificate Automation:** The operator automatically negotiates and provisions secure TLS certificates for all exposed services, such as the unified Homelab Dashboard, Forgejo, and n8n.

## 7. Centralized Observability and Telemetry

To support deep technical analysis and system auditing, the cluster implements comprehensive telemetry pipelines.

- **Log Aggregation:** Promtail daemonsets capture standard output and error streams from all containerized workloads, routing them to Loki for indexed storage with a 336-hour retention period.
- **Visualization:** Grafana provides unified dashboards for real-time log querying and infrastructure monitoring, removing the need for manual `kubectl logs` commands.
- **Extensible Pipelines:** The n8n workflow engine and custom Kubernetes CronJobs autonomously execute Extract, Transform, and Load operations. These pipelines utilize persistent volumes to cache session tokens, elegantly bypassing external API rate limits during data ingestion.
