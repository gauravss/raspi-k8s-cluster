# Install k8s on Raspberry Pi

## Steps

1. Install Docker

    ```bash
    curl -sSL get.docker.com | sh
    sudo usermod -aG docker pik8su
    ```

1. Set Docker daemon options

    ```bash
    sudo nano /etc/docker/daemon.json
    ```

    ```json
    {
        "exec-opts": ["native.cgroupdriver=systemd"],
        "log-driver": "json-file",
        "log-opts": {
            "max-size": "100m"
        },
        "storage-driver": "overlay2"
    }
    ```

1. Enable packet forwarding for IPv6

    ```bash
    sudo nano /etc/sysctl.conf
    ```

    Uncomment the following line

    ```bash
    # Uncomment the next line to enable packet forwarding for IPv4
    net.ipv4.ip_forward=1
    ```

1. Reboot the RPis

    ```bash
    sudo reboot
    ```

1. Ensure docker is running correctly

    ```bash
    sudo systemctl status docker
    ```

    It should display something like:

    ```bash
    ● docker.service - Docker Application Container Engine
        Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
        Active: active (running) since Sun 2021-07-04 04:26:50 UTC; 4min 37s ago
    TriggeredBy: ● docker.socket
        Docs: https://docs.docker.com
    Main PID: 1812 (dockerd)
        Tasks: 12
        CGroup: /system.slice/docker.service
                └─1812 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

    Jul 04 04:26:47 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:47.735744495Z" level=warning msg="Your kernel does not support CPU realtime scheduler"
    Jul 04 04:26:47 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:47.735770254Z" level=warning msg="Your kernel does not support cgroup blkio weight"
    Jul 04 04:26:47 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:47.735794606Z" level=warning msg="Your kernel does not support cgroup blkio weight_device"
    Jul 04 04:26:47 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:47.737330106Z" level=info msg="Loading containers: start."
    Jul 04 04:26:48 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:48.705165642Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
    Jul 04 04:26:49 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:49.056832142Z" level=info msg="Loading containers: done."
    Jul 04 04:26:50 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:50.212572623Z" level=info msg="Docker daemon" commit=b0f5bc3 graphdriver(s)=overlay2 version=20.10.7
    Jul 04 04:26:50 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:50.214158178Z" level=info msg="Daemon has completed initialization"
    Jul 04 04:26:50 pik8s-controller systemd[1]: Started Docker Application Container Engine.
    Jul 04 04:26:50 pik8s-controller dockerd[1812]: time="2021-07-04T04:26:50.377840271Z" level=info msg="API listen on /run/docker.sock"    
    ```

1. Run a test container

    ```bash
    docker run hello-world
    ```

### Add Kubenetes repository

1. Open the following file

    ```bash
    sudo nano /etc/apt/sources.list.d/kubernetes.list
    ```

    ```bash
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    ```

1. Add the GPG key to the Pi

    ```bash
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    ```

1. Install k8s packages

    ```bash
    sudo apt update
    sudo apt install kubeadm kubectl kubelet
    ```

## Setup the Master node / Control Plane

### Initialize Kubernetes

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Make a note of the kubeadm command which will be used to join nodes to the k8s cluster.

### Setup config directory

```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install Flannel network driver

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

[Flannel Reference](https://github.com/flannel-io/flannel#flannel)

You can install any. Please refer to [List of available implementations](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)

e.g. Calico

```bash
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

### Make sure pods are up

```bash
kubectl get pods --all-namespaces
```

```bash
pik8su@pik8s-controller:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-mc6rs                   1/1     Running   0          12m
kube-system   coredns-558bd4d5db-sdjp6                   1/1     Running   0          12m
kube-system   etcd-pik8s-controller                      1/1     Running   0          12m
kube-system   kube-apiserver-pik8s-controller            1/1     Running   0          12m
kube-system   kube-controller-manager-pik8s-controller   1/1     Running   0          12m
kube-system   kube-flannel-ds-x4d8v                      1/1     Running   0          74s
kube-system   kube-proxy-dn7wp                           1/1     Running   0          12m
kube-system   kube-scheduler-pik8s-controller            1/1     Running   0          12m
```

## Join worker nodes to the cluster

Run the join command on each worker node. This command was provided in the earlier step.

### If the token is expired or removed

Run the following command on the controller node

```bash
kubeadm token create --print-join-command
```

Use the output as join command.

### Check status of nodes

To check if the nodes have joined successfully, run the following command

```bash
watch kubectl get nodes
```
