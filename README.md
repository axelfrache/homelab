# homelab - cluster firelink

Kubernetes cluster running on Proxmox VMs, managed with GitOps.
Inspired by [onedr0p/cluster-template](https://github.com/onedr0p/cluster-template).

## Infrastructure

3 Talos Linux VMs on Proxmox — HA control-plane.

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