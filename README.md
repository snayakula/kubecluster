# Creating a Kubernetes cluster with a Master plane and Worker Node.

#### Steps Common on both Master and Worker
- Install DOCKER ENGINE
- Install CRI-dockered
- Install kubeadm, kubelet and kubectl

#### Step only on Master
- Initiate the Kubeadm with Network

#### Step only on Worker
- sudo kubeadm join **yourip:yourport** --token **yourtokenid** --cri-socket unix:///var/run/cri-dockerd.sock --discovery-token-ca-cert-hash sha256:**yourvalue** --v=5
- Note: The above kudeadm join string is provided by master node when you perform the "step only on Master"

## [Install DOCKER ENGINE using the apt repository](https://docs.docker.com/engine/install/ubuntu/)

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \

  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```


#### Install the Docker packages

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Test Docker installation
```
sudo docker run hello-world
```

## [Installing CRI-dockered Manual](https://github.com/Mirantis/cri-dockerd)

```
sudo apt-get install git-all
git clone https://github.com/Mirantis/cri-dockerd.git

sudo apt-get install wget    

install go lang : https://go.dev/doc/install
wget https://go.dev/dl/go1.22.1.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.1.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin' >>~/.profile
source ~/.profile
go version
--------------------------------------------------
cd cri-dockerd/
mkdir -p /usr/local/bin
go build -o bin/cri-dockerd

sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd

sudo install packaging/systemd/* /etc/systemd/system

sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

systemctl daemon-reload
systemctl enable --now cri-docker.socket
```


## [Installing kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
-------------------------------------------------------------------------------------
## Initiate the Kubeadm with Network - Perform this step only on Master plane / Master Node
```
sudo systemctl enable --now kubelet
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock

mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### [calico -manifest](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises)
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml -O

kubectl apply -f calico.yaml

kubectl get pods -A
```
## ADDITIONAL INFO:
#### Generating new join token:
kubeadm token generate

## Tips:
- use sudo before commands when you face permission access issue
- create Inbound rule to open the ports on Worker node

