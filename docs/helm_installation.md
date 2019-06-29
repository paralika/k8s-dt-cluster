### Install Helm (Latest-  v2.14.1) and deploy app on K8S cluster using Helm

#### Install and configure Helm in Kubernetes

- Helm comes with the tiller, it is a agent installed in kube-system namespaces. 
- Tiller users the default service account which has admin access, hence it is not safe in production environment. 
- Helm sugegsted below points to secure tiller. 
    - Create a cluster with RBAC enable. 
    - Configure ea ch Tiller gRPC endpoint to use separate TLS certificate. 
    - Release information should be a Kubernetes secrets. 
    - Install one Tiller per user, team, or other orgnisational entity with the --service-account flag, Roles, RoleBindings. 
    - Use the --tiller-tls-verify option with helm init and the --tls flag with other Helm commands to enforce verification.

However, for this tutorial, we are going to use insecure tiller which should not be used in production environment. 


##### Install Helm

$ `curl -L https://git.io/get_helm.sh | bash`  
$ `helm version`
$ `helm init --history-max 200`

Setting `--history-max` on helm init is recommended as configmaps and other objects in helm history can grow large in number if not purged by max limit. Without a max history set the history is kept indefinitely, leaving a large number of records for helm and tiller to maintain.

Create a service account which has admin rights on the cluser and associate it with tiller pod. 
- `kubectl apply -f helm/tiller_rbac.yaml`

- `kubectl edit tiller-deploy -n kube-system` and add above service account. 
 
You are ready to deploy application using Helm in K8S cluster.