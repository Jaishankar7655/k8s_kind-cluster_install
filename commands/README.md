# KIND Cluster Setup with Example Voting App, Argo CD, and Monitoring

This guide walks through setting up a **KIND Kubernetes cluster**, deploying the **Example Voting App**, configuring **Argo CD** for GitOps, and installing the **Kube Prometheus Stack** for monitoring.

---

## 1. Managing Docker and Kubernetes Pods

Check running Docker containers:
```bash
docker ps
```

List all Kubernetes pods across namespaces:
```bash
kubectl get pods -A
```

---


## 2. Managing Files in Example Voting App

Navigate into seed data:
```bash
cd .. && cd seed-data/
ls
cat Dockerfile
cat generate-votes.sh
```

---

## 3. Installing Argo CD

Create namespace and install Argo CD:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check services:
```bash
kubectl get svc -n argocd
```

Expose Argo CD via NodePort:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

Forward ports:
```bash
kubectl port-forward -n argocd service/argocd-server 8443:443 &
```

Retrieve initial Argo CD password:
```bash
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

---

## 4. Kubernetes Dashboard

Deploy dashboard:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Create a token:
```bash
kubectl -n kubernetes-dashboard create token admin-user
```

---

## 5. Delete KIND Cluster

Delete the cluster:
```bash
kind delete cluster --name=kind
```

---

## 6. Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

---

## 7. Install Kube Prometheus Stack

Add repos and update:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

Create namespace:
```bash
kubectl create namespace monitoring
```

Install Prometheus & Grafana:
```bash
helm install kind-prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort \
  --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort \
  --set alertmanager.service.nodePort=32000 --set alertmanager.service.type=NodePort \
  --set prometheus-node-exporter.service.nodePort=32001 --set prometheus-node-exporter.service.type=NodePort
```

Check services:
```bash
kubectl get svc -n monitoring
kubectl get namespace
```

Forward ports:
```bash
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
```

---

## 8. Prometheus Queries

### CPU Usage:
```promql
sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100
```

### Memory Usage:
```promql
sum (container_memory_usage_bytes{namespace="default"}) by (pod)
```

### Network Traffic:
```promql
sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)
```

---

## Summary
This setup provides:
- A **KIND Kubernetes cluster**  
- **Example Voting App** deployed on Kubernetes  
- **Argo CD** for GitOps deployments  
- **Kubernetes Dashboard** for cluster management  
- **Prometheus & Grafana** for monitoring and observability  
