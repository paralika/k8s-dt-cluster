### Deploy guestbook application

Below steps will demostrate how to deploy application in K8S cluster using manifest files.

#### Create namespace and ResourceQuota
On production environment it is recommanded to create `PodDisruptionBudget`, so that minimum Pods will be available to serve the user request in case of node disruption/maintainance. 
- `kubectl  apply -f deploy/namespace.yaml`

#### Deploy guest-book
- `kubectl  apply -f deploy/redis-master-deployment.yaml -n development` 
- `kubectl  apply -f deploy/redis-master-service.yaml -n development` 
- `kubectl  apply -f deploy/redis-slave-deployment.yaml -n development`
- `kubectl  apply -f deploy/redis-slave-deployment.yaml -n development` 
- `kubectl  apply -f deploy/frontend-deployment.yaml -n development`
- `kubectl  apply -f deploy/frontend-service.yaml  -n development`

Application is available on http://ip:port 
