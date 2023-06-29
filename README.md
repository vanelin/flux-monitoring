# flux-monitoring

# Source
- [Flux monitoring with Prometheus](https://fluxcd.io/flux/guides/monitoring/)
- [OpenTelemetry Operator for Kubernetes](https://github.com/open-telemetry/opentelemetry-operator)

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

# To install the opentelemetry-operator in an existing cluster, make sure you have cert-manager installed and run
flux create kustomization cert-manager \
  --interval=1h \
  --prune=true \
  --source=flux-monitoring \
  --path="./monitoring/cert-manager" \
  --health-check-timeout=1m \
  --wait

# Install opentelemetry-operator
flux create kustomization open-telemetry \
  --interval=1h \
  --prune=true \
  --source=flux-monitoring \
  --path="./monitoring/open-telemetry" \
  --health-check-timeout=1m \
  --wait

# You can access Grafana using port forwarding:

kubectl -n monitoring port-forward svc/kube-prometheus-stack-grafana 3000:80

kubectl --namespace monitoring port-forward --address 0.0.0.0  svc/kube-prometheus-stack-grafana 3000:80

# Get password Grafana
kubectl get secret --namespace monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

```