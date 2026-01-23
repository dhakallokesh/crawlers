
**Proxy Observability in Kubernetes Crawlers Overview
**This project demonstrates an observability stack for a large Kubernetes cluster running thousands of crawler pods in the crawlers namespace. Each crawler uses third‑party proxy vendors (vendor-a, vendor-b, vendor-c) with many proxy IPs.
The solution attributes outbound traffic to the correct vendor and provides accurate metrics for:

Requests sent via each proxy
Destination host/IP
Bandwidth sent per proxy per pod (outgoing bytes)
Bandwidth received per proxy per pod (incoming bytes)


 Architecture
<img width="911" height="401" alt="Untitled Diagram drawio (1)" src="https://github.com/user-attachments/assets/3d0f2657-74e4-418f-8efb-7c7434ac264b" />

Envoy sidecar intercepts outbound traffic, exports metrics (/stats/prometheus), and logs destinations.

Prometheus scrapes Envoy metrics from crawler pods.

Grafana visualizes proxy usage, bandwidth, and destinations.


Metrics Schema
Metric Name	Labels	Value Type
crawler_requests_total	vendor, proxy_ip, pod, namespace, destination	Counter
crawler_bytes_sent_total	same	Counter
crawler_bytes_received_total	same	Counter
Requests per proxy → envoy_cluster_upstream_rq_total

Bandwidth sent → envoy_cluster_upstream_cx_tx_bytes_total

Bandwidth received → envoy_cluster_upstream_cx_rx_bytes_total

Destination host/IP → from Envoy access logs (via Loki or Promtail).


1. Setup K3s  cluster
sudo dnf update -y
sudo dnf install -y curl wget vim
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.31.0+k3s1 sh -
Check status:
sudo systemctl status k3s
kubectl get nodes

3. Create namespace

kubectl create ns crawlers
kubectl create ns monitoring

3. Install Prometheus + Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack -n crawlers -f values.yml

4. Deploy crawler simulation

kubectl apply -f config-map.yml
kubectl apply -f vendor-a.yml
kubectl apply -f vendor-b.yml
kubectl apply -f vendor-c.yml


5. Access Grafana
kubectl port-forward svc/monitoring-grafana 3000:80 -n crawlers
Open http://localhost:3000

Validation
Logs:
kubectl logs -f crawler-sim-xxx -c envoy -n crawlers

Prometheus targets:
Check that crawler pods are scraped.
Grafana dashboards:
Requests per proxy vendor.
Bandwidth sent/received per pod.




