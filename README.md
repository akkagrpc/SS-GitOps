# Introduction 
SS-GitOps project helps in quick setup and destroy  kubernetes cluster.
It is organized into two modules
1. SS-Core
2. SS-App

## SS-Core
Core contains the foundation technologies needed for the cluster.
It has a) Storage b) API Gateway (Ingress) c) Service Mesh b) Secrets d) Docker registry authorization.
A demo application deployed for testing.

## SS-App
App contains the micorservices needed for Sock shop.

## Techstack
1. Storage - local-path-storage. This can be replaced by any storage in production. local-path-storage is used for single node deployments and development.
2. API Gateway - Ambassador is used for gateway and ingress the external traffic into the cluster. MetalLB is used for loadbalancing. LoadBalancer is optional. Usually it is inbuilt in cloud providers. 
3. Service mesh - Linkered 
4. Database - MongoDB and MySQL
5. Messaging - RabbitMQ


# Getting Started
Here are the instructions for working with k0s cluster. If you already have a cluster u can safely skip to stop 9.

## Pre-requisites
### **Operating system** : Ubuntu is preffered. For Windows users install WSL2 with Ubuntu 20.04.2 LTS

### **Docker** : Docker version 20. WSL2 users have to enable WSL2 integration with Ubuntu

### **Podman** : Podman is preffered. Following instructions will work with docker

### **Install Kubectl** : **Version v1.21.1** WSL Users have to install kubectl for windows and also for Ubuntu

### **kontena-lens** : for graphical administation
### **Broadband** : Numerous images will be downloaded. If you are on a metered network it is recomemded to switch over to broadband

## Links to software tools
1. [WSL2 installation https://docs.microsoft.com/en-us/windows/wsl/install-win10 ](https://docs.microsoft.com/en-us/windows/wsl/install-win10 "WSL2 Ubuntu")
2. [Docker for windows https://docs.docker.com/docker-for-windows/install/ ](https://docs.docker.com/docker-for-windows/install/ "Docker for windows 10")
3. [Docker WSL2 setup  https://docs.docker.com/docker-for-windows/wsl/ ](https://docs.docker.com/docker-for-windows/wsl/ "Docker for WSL2")
3. [Kubectl v1.21.1 for windows https://dl.k8s.io/release/v1.21.0/bin/windows/amd64/kubectl.exe ](https://dl.k8s.io/release/v1.21.0/bin/windows/amd64/kubectl.exe "Kubectl for windows")
4. [Kubectl for Linux. Choose v1.21.1 https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ ](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ "Kubectl for Linux")
5. [kontena-lens https://k8slens.dev ](https://k8slens.dev "kontena-lens")


## Working with k0s 

1. Checkout SS-GitOps 
```
mkdir -p  ~/workspace/SockShop
cd ~/workspace/SockShop
git clone http://tfsdev1:8080/tfs/DataD/DevOps/_git/SS-GitOps
```

2. Start k0s
```
docker run -d --name k0s --ip 172.17.0.2 --hostname k0s --privileged -v k0s_vol:/var/lib/k0s -v k0s_storage:/opt/local-path-provisioner -p 6443:6443 -p 443:443 -p 80:80 k0sproject/k0s:v1.21.0-k0s.0
```
3. Get all configurations
```
wsl users : mkdir -p "$(wslvar USERPROFILE)"/.kube
all users : mkdir -p ~/.kube

docker exec k0s cat /var/lib/k0s/pki/admin.conf > k0s-cluster.conf && \
docker exec k0s k0s default-config > k0s.yaml && \
docker exec k0s cat /var/lib/k0s/pki/admin.conf > ~/.kube/config && \
docker exec k0s cat /var/lib/k0s/pki/admin.conf > `wslpath "$(wslvar USERPROFILE)"`/.kube/config 
```
4. Check cluster status
```
kubectl cluster-info

    Kubernetes control plane is running at https://localhost:6443
    CoreDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    Metrics-server is running at https://localhost:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

```
5. Stop cluster 
```
docker stop k0s 
```
6. Start the cluster
```
docker start k0s 
```
7. Create foundation
```
kubectl create  -k ./overlays/local-crds/
kubectl -n ss-core wait --for condition=established --timeout=90s crd -lproduct=aes
```

8. Deploy SockShop 
```
kubectl apply -k ./overlays/local/
```
9. Check pod status. Wait till all the pods move to running state
```
kubectl get pods -o wide --all-namespaces -w
```

10. Test access to the demo application
```
curl -Lk https://localhost/backend/
{
    "server": "perky-apricot-y5cpah3e",
    "quote": "A late night does not make any sense.",
    "time": "2021-05-06T08:20:44.5536636Z"
}
```

11. Link to access ss
```
https://ss.<IP ADDRESS>.nip.io
```
12. Destroy all deployments
```
kubectl delete -k ./overlays/local/
```

## Debug tips
### Check if ambassador is ready
### AMBASSADOR_IP is the value used in ambassador-ip.yaml. It has to be same as the one defined in custom-ip.yaml.
```
kubectl -n ss-core wait --for condition=established --timeout=90s crd -lproduct=aes
curl -Lk "https://localhost/backend/"
```

### Check docker version
```
Client: Docker Engine - Community
 Cloud integration: 1.0.14
 Version:           20.10.6
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        370c289
 Built:             Fri Apr  9 22:46:45 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

```

## Kubectl version
```
kubectl version

Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T16:31:21Z", GoVersion:"go1.16.1", Compiler:"gc", Platform:"linux/amd64"}
```
13. ## Cleanup
```

docker stop k0s

docker rm k0s && \
docker volume ls && \
docker volume rm  k0s_storage && \
docker volume rm  k0s_vol

```

### Application design
1. [SockShop API](https://microservices-demo.github.io/api/index?url=https://raw.githubusercontent.com/microservices-demo/payment/master/api-spec/payment.json "SockShop Payments api")
2. [SockShop polyglot microservices design](https://github.com/microservices-demo/microservices-demo/blob/master/internal-docs/design.md "SockShop design")
3. [SockShop source code](https://github.com/microservices-demo "SockShop github link")