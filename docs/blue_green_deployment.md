### Blue Green deployment 
In this deployment we creates new set of deployments called Green which is based on new version of `guestbook` app, Blue deployment is the current deployment (version). 

I assumed that you already have `guestbook` deployed on K8S cluster. 

Follow the below steps for upgrading `guestbook` to `2.0`. 

- Update manifest files to change image version to 2.0 and deployment name, manifast files are available in `blue-green` folder.
- `kubectl apply -f blue-green/redis-master-deployment.yaml -n development`
- `kubectl apply -f blue-green/redis-slave-deployment.yaml -n development`
- `kubectl apply -f blue-green/frontend-deployment.yaml -n development`
- Apply `blue-green/test-app.yaml` which will create services for above/Green deployment with different names than the actual (in Blue), so that you can test it. 
- `kubectl get svc -n development` 
- Access `guestbook` of Green deployment. 
- If everything working as expected patch actual services. 
- Delete Blue deployments (excluding services).
- Delete test services created in test deployment. 