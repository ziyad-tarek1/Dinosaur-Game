
#                                                  *The Unlimated CI/CD DevOps Project*

![unlimated-cicd](https://github.com/user-attachments/assets/4eeed70b-e860-4494-80ed-d050e680a10c)


## setting-up-monitoring-with-prometheus-and-grafana
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm repo list
kubectl create namespace monitoring
helm install prometheus prometheus-community/prometheus --namespace monitoring
helm install grafana grafana/grafana --namespace monitoring
```

### we can check the status by running:
 ```bash
kubectl get pods -l app.kubernetes.io/instance=prometheus
```

### To expose the prometheus service
```bash
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-np
```

### This will open a browser window with the Prometheus web interface.
```bash
minikube service prometheus-server-np
```

### To expose the Grafana service
```bash
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-np
```

### This will open a browser window with the Grafana web interface.
```bash
minikube service grafana-np
```

### To get the Grafana admin password, run the following:
```bash
echo "The Grafana uusername is admin & admin password is:" && kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

### kube-prometheus-stack
the kubernetse exporter
  ```sh
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

Expose the kubernetse-exporter

```bash
# to find the exporter port
kubectl get svc -n monitoring
```

### Importing Dashboards
Grafana provides pre-built dashboards for Kubernetes monitoring. You can import dashboards like:

Kubernetes Cluster Monitoring: ID 6417
Node Exporter Full: ID 1860

