### Setup Prmontheus and Grafana on K8S cluster

#### Create monitoring namespace
- kubectl create namespace monitoring

#### Configure Prometheus
- `kubectl apply -f monitoring/cluster-role.yaml -n monitoring`
- `kubectl apply -f monitoring/prometheus-config-map.yaml -n monitoring`
- `kubectl apply -f monitoring/prometheus-deployment.yaml -n monitoring`
- `kubectl get svc -n monitoring`
- Access Prometheus http://ip:port (you may need to use kubectl port-forward)
- Verify that node, pods metrics are getting scrape. 

#### Configure Grafana
- `kubectl apply -f grafana-deployment.yaml -n monitoring`
- `kubectl apply -f grafana-service.yaml -n montoring`
- `kubectl get svc -n monitoring`
- Access Grafana http://ip:port 
- Add above prometheus as datasource
- Import Kubernetes (https://grafana.com/dashboards/2115) dashboard

**Note: Some services are deployed as a `LoadBalancer` to access it from outside cluster. In production, we generally use nginx ingress controller to make service accessible outside cluster.**
