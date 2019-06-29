### Canary Deployment

Canary deployment always goes well with Istio, in which you can mention the percentage of trafic which should divert to the new deployment. 

One of the way to accomplish Canary deployment without Istio is by using Kubernetes labels. 

Here we will see how to do it with labels. I assume that you have alrready installed `guestbook` app in your cluster. 
- Update manifest files to change `release` label to canary and version numberr to `2.0` (both label and image). 
- Updated files are at `canary` folder
- `kubectl apply -R -f canary/ -n development`
- Now try to access `guestbook` url, sometime you will see `Submit` button and sometime `ClickMe` button. `ClickMe` button is part of version `2.0`.
