### Setup EFK stack (for logging) on K8S cluster
Below steps shows how to create single node setup for Elastcsearch, and Fluentd as well as Kibana. 

#### Configure Elasticsearch

We are using local volumes here hence data is not persistent. 

- `kubectl create ns logging`
- `kubectl apply -f logging/elasticsearch_svc.yaml.yaml -n logging`
- `kubectl apply -f logging/elasticsearch_statefulset.yaml -n logging`
- `kubectl get pods -n logging`

#### Configure Kibana
- `kubectl apply -f logging/kibana.yaml  -n logging`
- Kibana will be accessible on http://ip:port (you may need to use kubectl port-forward as service type is ClusterIP)
- Verify that by curl http://<node-ip>:<node-port>

#### Configure Fluentd
- `kubectl apply -f logging/fluentd.yaml -n logging`

- Access Kibana using http://ip:port
- Click on Discover at left panel 
- You should should see the recent log entries 

At this point we have successfully configured EFK stack on your kubernetes cluster.