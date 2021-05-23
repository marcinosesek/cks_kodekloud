# Prepare SD cards
1. Download `balenaEtcher` to burn images intu SD cards
1. Download Ubuntu 20.04 x64 bit for raspberry pi if using raspberrypi 4
1. Burn Image on SD cards. In my case 3 SD cards for:
    * k8s-master
    * k8s-worker-01
    * k8s-worker-02
1. Boot Raspberry pi 
1. Change hostname: `vi /etc/hostname`
1. Update /etc/hosts
    ```
    128.0.0.1 localhost
    <> k8s-master
    ```
1. Configure static ip for eth0 interface
    1. Make sure that static ip will be disabled from DHCP pool
    ```
    sudo vi /etc/netplan/50-cloud-init.yaml
    network:
    ethernets:
        eth0:
            dhcp4: no
            dhcp6: no
            addresses: [192.168.1.58/24]
            gateway4: 192.168.1.1
            nameservers:
                    addresses: [8.8.8.8,8.8.4.4]
    version: 2
    ```
    ```
    sudo netplan apply
    ```
1. Open `sudo vi /boot/firmware/cmdline.txt` and at the end add this values:
    ```
     cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1
    ```
1. Install all updates:
    ```
    sudo apt update && sudo apt dist-upgrade
    ```
 1. Reboot node
 1. Create user and make him a member of sudo group
    ```
    sudo adduser mo
    sudo usermod -aG sudo mo
    ```
1. Install docker
    ```
    curl -sSL get.docker.com | sh
    ```
1. Add new created user to the docker group
    ```
    sudo usermod -aG docker mo
    ```
1. Edit `sudo vi /etc/docker/daemon.json`
    ```
    {
     "exec-opts": ["native.cgroupdriver=systemd"],
     "log-driver": "json-file",
     "log-opts": {
        "max-size": "100m"
     },
     "storage-driver": "overlay2"
    }
    ```
1. Edit `sudo vi /etc/sysctl.conf` and uncomment
    ```
    net.ipv4.ip_forward=1
    ```
1. Reboot node
1. Check if cocker is up and running
    ```
    sudo systemctl status docker
    ```
1. Check if docker can run container
    ```
    docker run hello-world
    ```
1. Add K8s repository 
    ```
    sudo vi /etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    ```
1. Download gpg key
    ```
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    ```
1. Update repository
    ```
    sudo apt update
    ```
1. Instal K8s packages
    ```
    sudo apt install kubeadm kubectl kubelet
    ```
1. Initialize K8s Cluster on k8s-master node only
    ```
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ...
    Your Kubernetes control-plane has initialized successfully!

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

kubeadm join 192.168.1.10:6443 --token oq22kq.6l0oas0xshbu9jyz --discovery-token-ca-cert-hash sha256:d00d3cba36014bbedc4a7ab63ef8e5bb3b6d77a52f2b6da31b0a010dec08119a 
    ```
1. Run commands as regular user:
    ```
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
1. Install K8s network driver
    ```
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ```
1. Check running pods:
    ```
    kubectl get pods --all-namespaces
    NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
    kube-system   coredns-558bd4d5db-7wrw6             1/1     Running   0          5m31s
    kube-system   coredns-558bd4d5db-mn9np             1/1     Running   0          5m31s
    kube-system   etcd-k8s-master                      1/1     Running   0          5m32s
    kube-system   kube-apiserver-k8s-master            1/1     Running   0          5m43s
    kube-system   kube-controller-manager-k8s-master   1/1     Running   0          5m43s
    kube-system   kube-flannel-ds-djtkp                1/1     Running   0          48s
    kube-system   kube-proxy-4nz7f                     1/1     Running   0          5m30s
    kube-system   kube-scheduler-k8s-master            1/1     Running   0          5m43s
    ```