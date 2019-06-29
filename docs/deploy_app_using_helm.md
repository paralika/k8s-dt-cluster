
### Create Helm chart for guestbook

Creating Helm charts for your applicattion is really easy. You need to create helm template and update your deployment and services in that template. 

- `helm create guestbook` - it will create template in current deployment. 
- Create `deployment.yaml` for your application
- Create `service.yaml` for your application 
- Copy those in guestbook/template/

This is quick way to create Helm chart. However, you should use as much as possible variable in `deployment.yaml` and `service.yaml`.

#### Deploy application using helm

- `helm install --namespace development --name guestbook ./guestbook` 

> helm  list   
NAME     	REVISION	UPDATED                 	STATUS   	CHART          	APP VERSION	NAMESPACE   
guestbook	1       	Fri Jun 28 19:50:48 2019	DEPLOYED  	guestbook-0.1.0	1.0        	development 

Now you can access `guestbook` app using http://ip:port.