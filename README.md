 # Setup-kubernetes-cluster
 ## Launch 3 EC2 servers.
 ## Configure first as a master node and the other two as a slave.
 ## Deploy one WordPress website 
 
 # Addition
``` Attach persistent volume over the DB ``` 
 
## Step 1 - Launch 3 ec2 Instance - one for master and 2 for slave 
for master , choose the instance type t2xlarge and for the two slave , choose the t2small type

## Step 2 - Configuring the master and slave nodes

### On All EC2, the master and worker nodes:

Install and enable the docker service so that the docker service starts on system restarts.

```
yum install docker -y
systemctl enable docker --now
```

Create proper yum repo files so that we can use yum commands to install the components of Kubernetes.

```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

vim /etc/docker/daemon.json

{
"exec-opts": ["native.cgroupdriver=systemd"]
}

sysctl command is used to modify kernel parameters at runtime. Kubernetes needs to have access to kernel’s IP6 table and so we need to do some more modifications

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

Install kubelet, kubeadm and kubectl; start kubelet daemon

```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
yum install tc
systemctl restart docker
sudo sysctl --system
```

### Only on the Master Node:


```
kubeadm config images pull
kubeadm init  --pod-network-cidr=10.244.0.0/16{network plugin cidr range}

```
Wait for few minutes. Following commands will appear on the output screen of the master node

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

This will also generate a kubeadm join command with some tokens.
save the token at a place as it will be going to used for joining the worker with the master

### On the Worker Node


```
kubeadm join 172.31.45.77:6443 --token ajrjbr.ovvaz6kohgyb0jr0 \
        --discovery-token-ca-cert-hash sha256:XYZXYZXYZXYZ
```   

Now, wait for few minutes to let the pods in ready state 

![image](https://user-images.githubusercontent.com/67600604/178671012-511d5fa2-2f2c-49ed-b180-eb0d15b0286d.png)

```
kubectl get nodes
```

![image](https://user-images.githubusercontent.com/67600604/178670808-10a16798-cef5-45cc-a5c2-f38b269818d0.png)


## Step 3 - Deploy one WordPress website

Now, for the wordpress, we will need a database , so here we are using mysql database

for this, we will create a file , I named it as mysql-deploy

For, Volume Claim , I created a file pv.yml and pvc.yml and pass.yml for secrets and then one file named as  wordpress.yml for the wordpress deployment on the cluster

all these files are given in the same repo

Now, after creating these files 

![image](https://user-images.githubusercontent.com/67600604/178674800-a5bc43ea-7520-4ddf-9a38-1c167c79383f.png)

Now, will create these with kubectl command

```
kubectl create -f pass.yml
kubectl create -f pv.yml
kubectl create -f pvc.yml
kubectl create -f mysql-deploy.yml
kubectl create -f wordpress-deploy.yml
```

Now, will check the deployments, pods and nodes from the master 


kubectl get pods

![image](https://user-images.githubusercontent.com/67600604/178676285-a861d16a-19ee-42f4-a4c3-be9aacc4cf5e.png)

kubectl get svc

![image](https://user-images.githubusercontent.com/67600604/178676417-b43ff887-8c97-4a70-9a8e-0b1b48d02725.png)

kubectl get pv

![image](https://user-images.githubusercontent.com/67600604/178676488-5ecd9902-b20f-4d0f-a36b-d2453db31034.png)
