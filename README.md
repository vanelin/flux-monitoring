# flux-monitoring

```bash
# First register the Git repository on your cluster:
flux create source git flux-monitoring \
  --interval=30m \
  --url=https://github.com/vanelin/flux-monitoring \
  --branch=main

# Install the Prometheus stack
flux create kustomization kube-prometheus-stack \
  --interval=1h \
  --prune \
  --source=flux-monitoring \
  --path="./monitoring/kube-prometheus-stack" \
  --health-check-timeout=5m \
  --wait

# Install the Loki stack
flux create kustomization loki-stack \
  --depends-on=kube-prometheus-stack \
  --interval=1h \
  --prune \
  --source=flux-monitoring \
  --path="./monitoring/loki-stack" \
  --health-check-timeout=5m \
  --wait

# Install Flux Grafana dashboards
flux create kustomization monitoring-config \
  --depends-on=kube-prometheus-stack \
  --interval=1h \
  --prune=true \
  --source=flux-monitoring \
  --path="./monitoring/monitoring-config" \
  --health-check-timeout=1m \
  --wait

# You can access Grafana using port forwarding:

kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80

```