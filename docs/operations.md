# Operations

## Talos

### Update node configuration

```sh
task talos:generate-config
task talos:apply-node IP=<ip> MODE=auto
# Use MODE=reboot to force a restart
```

### Upgrade Talos or Kubernetes

Update versions in `talos/talenv.yaml`, then:

```sh
# Upgrade Talos on a node
task talos:upgrade-node IP=<ip>

# Upgrade Kubernetes
task talos:upgrade-k8s
```

### Add a node

1. Boot the node in maintenance mode
2. Retrieve disk and MAC address:
   ```sh
   talosctl get disks -n <ip> --insecure
   talosctl get links -n <ip> --insecure
   ```
3. Add the node in `talos/talconfig.yaml`
4. Generate and apply:
   ```sh
   task talos:generate-config
   task talos:apply-node IP=<ip>
   ```

### Reset the cluster

```sh
task talos:reset
```

---

## Flux

### Force a sync

```sh
task reconcile
```

### Diagnose resources

```sh
flux get ks -A
flux get hr -A
flux get sources git -A
```

### Debug a pod

```sh
kubectl -n <namespace> get pods
kubectl -n <namespace> logs <pod>
kubectl -n <namespace> describe <resource> <name>
kubectl -n <namespace> get events --sort-by='.metadata.creationTimestamp'
```

---

## Adding an application

1. Create `kubernetes/apps/<namespace>/<app>/app/` with:
   - `ocirepository.yaml` — Helm chart source
   - `helmrelease.yaml` — Helm configuration
   - `kustomization.yaml` — resource list
2. Create `kubernetes/apps/<namespace>/<app>/ks.yaml` — Flux Kustomization
3. Reference it in `kubernetes/apps/<namespace>/kustomization.yaml`
4. Push → Flux deploys automatically

### Expose an app internally

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: <app>
  namespace: <namespace>
spec:
  parentRefs:
    - name: envoy-internal
      namespace: network
  hostnames:
    - <app>.<your-domain>
  rules:
    - backendRefs:
        - name: <service>
          port: <port>
```

### Expose an app externally (internet)

Replace `envoy-internal` with `envoy-external` in `parentRefs`.

---

## Certificates

```sh
kubectl get certificates -A
kubectl get challenges -A
kubectl describe order -n <namespace> <name>
```

Force a retry if stuck:
```sh
kubectl delete order -n <namespace> <order-name>
```