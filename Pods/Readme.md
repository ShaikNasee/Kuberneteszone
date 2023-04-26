Installing Kubernetes 
----------------------
* Take 3 machines in any cloud with the configuration of 2Gi of RAM and 2VCPU's 
* Install docker in all the 3 machines 
* Command to install docker 
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker ubuntu
```
* Once docker installation is done in all the 3 machines, install cri-dockerd in all the 3 machines 
become a root user and perform all these activities because some of these options require root options 
refer here[https://github.com/Mirantis/cri-dockerd] for the cri-dockerd installation steps in all the 3 machines
* steps:  
```
sudo -i 
git clone https://github.com/Mirantis/cri-dockerd.git
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile
go version 
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```
* do the above all steps in all the machines 
* install kubeadmin 
refer here[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl]
* take according to your linux distirbution mine is ubuntu so taking debian based package commands
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
Note: follow the official documentation whenever your installing the k8s link given above for all the steps
* become root user 

`sudo -i `
`kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock `
* Add other nodes to the master nodes by following the below commands 
```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.41.119:6443 --token xcgb6b.6t4izavcmgmvyssk \
	--discovery-token-ca-cert-hash sha256:6d579713b780b85d8bbf52510db0d99aa75b531cb6a354037987ab0a6355b360 --cri-socket "unix:///var/run/cri-dockerd.sock"
```
* Once the bove insallation done check th pods if all are running or not , if codedns service pods are not runing then install `flannel plugin`to run the networing 
install this on master node 
* For flannel refer here[https://github.com/flannel-io/flannel#deploying-flannel-manually]
`kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml`

KUBERNETES NETWORKING MODEL
---------------------------

* All the communication with in the pods should happend without NAT (netwokr address translater)
* all the nodes sould not be doing NAT 
* pod knows the ip address, the other pod should also recognise the ip address as same 
* Net Filters and IP ranges 
* every pods gets a unique IP address 


LABELS: 

* To Query the pods we will be using pod lables 