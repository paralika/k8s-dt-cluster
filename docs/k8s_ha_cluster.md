## Create HA K8S cluster on GCE using kubeadm

This cluster is going to have three master nodes, one load balancer node (nginx), and one worker node. With the stacked control plane nodes approach, the etcd members and control plane nodes are co-located. This approach require less infrastructure.

This is design for the development purpose, should not be used for production. 

#### Prepare your desktop by installing/configuring required tools
- Install `gcloud` https://cloud.google.com/sdk/
- gcloud init -> login -> configure
- Install `kubectl` as per your os.  
For mac - `curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/darwin/amd64/kubectl`
- Verify - `kubectl version --client`

#### Create VPC
- `gcloud compute networks create k8s-cluster --subnet-mode custom`
- `gcloud compute networks subnets create kubernetes --network k8s-cluster --range 10.240.0.0/24`

##### Create internal and external firewall rules
- `gcloud compute firewall-rules create k8s-cluster --allow tcp,udp,icmp --network k8s-cluster --source-ranges 10.240.0.0/24,10.200.0.0/16`
- `gcloud compute firewall-rules create k8s-cluster-external --allow tcp:22,tcp:6443,icmp --network k8s-cluster --source-ranges 0.0.0.0/0`
- `gcloud compute firewall-rules list --filter="network:k8s-cluster"`

#### Create compute resources for master
**Note: Make sure that master nodes has 2 vCPU, it is prerequisite for `kubeadm`.** 

>for i in 0 1 2; do
  gcloud compute instances create master-${i} \
    --async \
    --boot-disk-size 20GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-2 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags k8s-cluster,controller  
done

#### Create compute resources for worker and Load Balancer

**Node: Because of resource limitation, I am going to create only one worker node. Worker node and Load balance will have 1vCPU.**

>for i in 0 1; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 40GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags k8s-cluster,worker   
done  

#### Verify that instances are getting created
- `gcloud compute instances list`

#### Setup Load Balancer
- `gcloud compute ssh worker-1`
- `sudo apt-get install -y nginx`
- `sudo systemctl enable nginx`
- `sudo mkdir -p /etc/nginx/tcpconf.d`
- `sudo vi /etc/nginx/nginx.conf` 
>   include /etc/nginx/tcpconf.d/*;
- Add kuberneretes.conf file in nginx
>cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf  
>stream {  
    upstream kubernetes {  
        server 10.240.0.10:6443;  
        server 10.240.0.11:6443;  
        server 10.240.0.12:6443;  
    }  
    server {  
        listen 6443;  
        listen 443;  
        proxy_pass kubernetes;  
    }  
}  
EOF  

- `sudo nginx -s reload`
- `curl -k https://localhost:6443/version`

Right now it will not return K8S details as we haven't setup master node yet. 

Now you have network, instances and load balancer ready for configuring K8S HA cluster. 

#### Provision K8S nodes 
SSH to masters and worker-1 node, one by one, to install `kubelet`, `kubeadm`, `kubectl` and `docker` on each node. Or use `tmux` to to do it simultaniously. 

- `gcloud compute ssh <node>`
- `sudo su -`
- `apt-get update && apt-get install -y apt-transport-https curl`  
- `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -`  

>cat <<EOF >/etc/apt/sources.list.d/kubernetes.list  
deb https://apt.kubernetes.io/ kubernetes-xenial main  
EOF  

- `apt-get update`
- `apt-get install -y kubelet kubeadm kubectl`
- `apt-mark hold kubelet kubeadm kubectl`
- Install docker, follow steps given at https://docs.docker.com/install/linux/docker-ce/ubuntu/

- Verify docker - `systemctl status docker`
- Kubelet should be still in activating mode as we haven't cluster yet. `systemctl status kubelet`.

#### Bootstrap master nodes
- `gcloud compute ssh master-0`
- `sudo su -`
- Create a configuration file called kubeadm-config.yaml:

> apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "LOAD_BALANCER_PUBLIC_IP:6443"

- `sudo kubeadm init --config=kubeadm-config.yaml --upload-certs`

You will get output looks like, copy it somewhere. You will need it to add nodes in cluster:

>To start using your cluster, you need to run the following as a regular user:

>>  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

>You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

>You can now join any number of the control-plane node running the following command on each as root:

>>  kubeadm join xx.xx.xx.xx:6443 --token t76307.x1amtoh6wxve8gp5 --discovery-token-ca-cert-hash sha256:82413de041f735f22febea7b668acfb55b8cb9e07eba288237f43a6de418d541 --experimental-control-plane --certificate-key 8fe8277efb42a884718009cfd8cfb04b478c1d2cad1a517e1728125d7b8fef6d

>Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use 
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

>Then you can join any number of worker nodes by running the following on each as root:

>>kubeadm join xx.xx.xx.xx:6443 --token t76307.x1amtoh6wxve8gp5 \
    --discovery-token-ca-cert-hash sha256:82413de041f735f22febea7b668acfb55b8cb9e07eba288237f43a6de418d541 

- Add kubeconfig file 
> mkdir -p $HOME/.kube  
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

- `kubectl get nodes` This will show the node is in `NotReady` state. Install the CNI plugin now. 
> kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

- Node should be in `Ready` state now. Verify that - `kubectl gete nodes`.

- SSH to master-1 and then master-2 and run below command, to add them in control-plane. 

>  kubeadm join xx.xx.xx.xx:6443 --token t76307.x1amtoh6wxve8gp5 --discovery-token-ca-cert-hash sha256:82413de041f735f22febea7b668acfb55b8cb9e07eba288237f43a6de418d541 --experimental-control-plane --certificate-key 8fe8277efb42a884718009cfd8cfb04b478c1d2cad1a517e1728125d7b8fef6d

#### Add worker node
- gcloud compute ssh worker-0
>kubeadm join xx.xx.xx.xx:6443 --token t76307.x1amtoh6wxve8gp5 \
    --discovery-token-ca-cert-hash sha256:82413de041f735f22febea7b668acfb55b8cb9e07eba288237f43a6de418d541 

##### Copy kubeconfig file on your local machine
- SSH to master node and copy config time to temp
- On your local machine `gcloud compute scp master-0:/tmp/config config`
- Move this file to `~/.kube/`
- Remove /tmp/config from `master-0`

#### Optionally, label worker node
- From your local machine, `
kubectl label node worker-1 node-role.kubernetes.io/worker=worker`

`kubectl get nodes`
>NAME         STATUS     ROLES      AGE     VERSION  
master-0     Ready      master     30m     v1.15.0  
master-1     Ready      master     27m     v1.15.0  
master-2     Ready      master     25m     v1.15.0  
worker-1     Ready      worker     20m     v1.15.0  

Your development cluster is ready now. 