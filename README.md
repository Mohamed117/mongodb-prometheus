# MongoDB Monitoring with Prometheus and Grafana on Kubernetes

This guide demonstrates how to monitor a MongoDB deployment within a Kubernetes cluster using Prometheus and Grafana. It covers deploying Prometheus, Grafana, MongoDB, and the MongoDB exporter, along with configuring them to collect and visualize MongoDB metrics.

## Prerequisites

* A running Kubernetes cluster.
* Helm installed.
* `kubectl` configured to connect to your cluster.

## 1. Deploy Prometheus and Grafana

We'll use the `kube-prometheus-stack` Helm chart for easy deployment.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable  # Consider migrating to prometheus-community charts if possible
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
```

### Accessing Grafana

#### Expose Grafana:

```bash
kubectl expose deployment prometheus-grafana --type=LoadBalancer --name=exposed-grafana
```

#### Retrieve Credentials:

```bash
kubectl get secret prometheus-grafana -o yaml | grep -E 'admin-user|admin-password'
```

Decode the `admin-user` and `admin-password` values (they are base64 encoded). You can use a tool like `base64 -d` or an online decoder.

**Default Credentials (may be outdated, check the secret):**
- User: `admin`
- Password: `prom-operator`

#### Access Grafana:
Once the LoadBalancer has an external IP (check with `kubectl get service exposed-grafana`), you can access Grafana in your browser using that IP.

### Accessing Prometheus

#### Expose Prometheus:

```bash
kubectl expose service prometheus-kube-prometheus-prometheus --type=LoadBalancer --name=exposed-prometheus
```

#### Access Prometheus:
Access the Prometheus UI via the external IP of the `exposed-prometheus` service.

## 2. Deploy MongoDB

Apply your MongoDB deployment and service configuration. Replace `mongodb.yml` with the actual file name if different.

```bash
kubectl apply -f mongodb.yml
```

## 3. Deploy the MongoDB Exporter

The MongoDB exporter collects metrics from MongoDB and exposes them to Prometheus.

### Prepare `values.yaml`:

```bash
helm show values prometheus-community/prometheus-mongodb-exporter > values.yaml
vim values.yaml
```

### Configure `values.yaml`:
Modify the `values.yaml` file with your MongoDB connection details. Crucially, ensure the `mongodb.uri` points to your MongoDB service.

```yaml
mongodb:
  uri: "mongodb://mongodb-service:27017" # Replace with your MongoDB service name and port

serviceMonitor:
  interval: 90s
  scrapeTimeout: 60s
  enabled: true
  additionalLabels:
    release: prometheus
```

### Install the Exporter:

```bash
helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f values.yaml
```

## 4. Verify the Exporter

#### Expose the Exporter (Optional - For direct access):
If you want to directly access the exporter's metrics endpoint, you can expose it. This is generally not required as Prometheus will scrape it automatically.

```bash
kubectl expose service mongodb-exporter-prometheus-mongodb-exporter --type=LoadBalancer --name=exposed-mongodb-exporter
```

#### Check Metrics (Optional):
Access the exposed service's external IP to see the metrics.

## 5. Configure Prometheus to Scrape the Exporter (Usually Automatic)

The `serviceMonitor` configuration in the `values.yaml` file should automatically configure Prometheus to discover and scrape the exporter. Verify this by checking the Prometheus targets page (accessible via the exposed Prometheus service). You should see the MongoDB exporter listed as a target.

## 6. Import Grafana Dashboards (Recommended)

Find and import pre-built Grafana dashboards for MongoDB monitoring. This will give you a head start in visualizing your MongoDB metrics. Search the Grafana dashboard library for "MongoDB" and import a relevant dashboard.

## Troubleshooting

- **Exporter Not Scraping:** Check Prometheus targets and logs for errors. Ensure the `mongodb.uri` in the `values.yaml` is correct and that the MongoDB service is accessible from the exporter pod.
- **Connection Issues:** Verify network connectivity between the exporter, Prometheus, and MongoDB. Check DNS resolution and network policies.
- **Credentials:** Double-check the MongoDB connection credentials.
