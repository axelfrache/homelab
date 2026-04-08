# homelab

Kubernetes cluster running on Proxmox VMs, managed with GitOps.
Based on [onedr0p/cluster-template](https://github.com/onedr0p/cluster-template).

## Infrastructure

### Hardware

Single node running Proxmox:

| Component | Spec |
|---|---|
| CPU | Intel Core i5-14400 (10c, iGPU QSV) |
| Motherboard | ASRock B760M-ITX/D4 WiFi (Mini-ITX) |
| RAM | 32 GB DDR4 3200 MHz (2x16 GB Kingston Fury Beast) |
| NVMe | Samsung 970 EVO Plus 1 TB (Proxmox + VMs) |
| HDD | WD Red Plus 4 TB (future NFS storage) |
| PSU | be quiet! Pure Power 12M 550W 80+ Gold |
| Case | Fractal Design Node 304 (Mini-ITX) |

### VMs

3 Talos Linux VMs on Proxmox : HA control-plane, all nodes run workloads.

| VM | Role | CPU | RAM | Disk |
|---|---|---|---|---|
| firelink-01 | control-plane | 4 vCPU | 8 GB | 50 GB |
| firelink-02 | control-plane | 4 vCPU | 8 GB | 50 GB |
| firelink-03 | control-plane | 4 vCPU | 8 GB | 50 GB |

## Stack

- **OS**: Talos Linux
- **Kubernetes**: v1.35+
- **CNI**: Cilium
- **GitOps**: Flux v2
- **Secrets**: SOPS + age
- **Ingress**: Envoy Gateway
- **DNS**: k8s-gateway + external-dns (Cloudflare)
- **TLS**: cert-manager + Let's Encrypt
- **Tunnel**: Cloudflare Tunnel
- **Storage**: Longhorn

## Usage

```sh
# Cluster status
kubectl get nodes
flux get ks -A
flux get hr -A

# Force Flux sync
task reconcile

# Apply Talos config
task talos:generate-config
task talos:apply-node IP=<ip> MODE=auto
```

## Adding an application

Create the structure under `kubernetes/apps/<namespace>/<app>/` following existing apps as a reference, then push.

See [docs/operations.md](./docs/operations.md) for details.