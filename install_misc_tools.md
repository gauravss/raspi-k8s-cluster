# Install essential tools in the k8s cluster

## MetalLB 

MetalLB is a load-balancer implementation for bare metal Kubernetes clusters, using standard routing protocols.

Kubernetes does not offer an implementation of network load-balancers (Services of type LoadBalancer) for bare metal clusters. The implementations of Network LB that Kubernetes does ship with are all glue code that calls out to various IaaS platforms (GCP, AWS, Azure…). If you’re not running on a supported IaaS platform (GCP, AWS, Azure…), LoadBalancers will remain in the “pending” state indefinitely when created.

### Installation 

Refer to <https://metallb.universe.tf/installation/> for install instructions. Apply the following manifests

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
```

### Define a ConfigMap

Create a new manifest file like [metallb-config](scripts/metallb-config.yaml) and apply it

```bash
kubectl apply -f scripts/metallb-config.yaml
```

NOTE - Make sure to add the addresses that are outside the DHCP allocation pool of your network.

### References
- <https://levelup.gitconnected.com/step-by-step-slow-guide-kubernetes-cluster-on-raspberry-pi-4b-part-3-899fc270600e>
## Metrics Server

Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

Metrics Server collects resource metrics from Kubelets and exposes them in Kubernetes apiserver through Metrics API for use by Horizontal Pod Autoscaler and Vertical Pod Autoscaler.

Refer to [Github](https://github.com/kubernetes-sigs/metrics-server/) for original documentation.

To customize the script as follows

```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
```

Add the following changes to the `metrics-server` deployment

```yaml
    spec:
      hostNetwork: true
```

```yaml
      containers:
      - args:
        - --kubelet-insecure-tls=true
```

```yaml
    spec:
      hostNetwork: true
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls=true
```

```bash
kubectl apply -f components.yaml
```

To get metrics for the cluster

```bash
kubectl top nodes
```

```bash
NAME               CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
pik8s-controller   474m         11%    1074Mi          13%
pik8s-node-01      156m         3%     546Mi           7%
pik8s-node-02      142m         3%     742Mi           9%
pik8s-node-03      176m         4%     584Mi           7%
```

To get pod statistics in a given namespace run

```bash
kubectl top pods -n kube-system
```

```bash
NAME                                       CPU(cores)   MEMORY(bytes)
coredns-558bd4d5db-mc6rs                   8m           11Mi
coredns-558bd4d5db-sdjp6                   8m           11Mi
etcd-pik8s-controller                      44m          64Mi
kube-apiserver-pik8s-controller            153m         283Mi
kube-controller-manager-pik8s-controller   38m          50Mi
kube-flannel-ds-45fch                      14m          12Mi
kube-flannel-ds-8z2gk                      14m          11Mi
kube-flannel-ds-rkjsz                      16m          12Mi
kube-flannel-ds-x4d8v                      16m          12Mi
kube-proxy-7k89t                           1m           14Mi
kube-proxy-dn7wp                           2m           19Mi
kube-proxy-lch5g                           1m           14Mi
kube-proxy-s6xw4                           2m           19Mi
kube-scheduler-pik8s-controller            10m          19Mi
metrics-server-6d8d759c46-jd7bc            10m          17Mi
```

### References

- <https://levelup.gitconnected.com/setting-up-and-using-metrics-server-for-raspberrypi-kubernetes-cluster-de2e10bb9459>
- <https://github.com/kubernetes-sigs/metrics-server/>

## Kubernetes Dashboard

Dashboard is a web-based Kubernetes user interface. Refer to [official documentations here](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

After deploying the dashboard, create a user that we can use to login. Follow these steps as mentioned in [this guide](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)

### Getting a Bearer Token

```bash
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

## Helm

[Helm](https://helm.sh/) is a package manager for kubernetes. Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

Based on [installation guide here](https://helm.sh/docs/intro/install/), run following commands

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Add charts repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add infobloxopen https://infobloxopen.github.io/cert-manager/
```

## Ingress-controller

Refer to [bare-metal section](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal) on the NGINX ingress controller and run the following command

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml
```

You should see the result as follows 

```bash
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
```

To verify installation run the following command

```bash
kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx --watch
```

NOTE - Please also refer to [other bare-metal considerations](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)

## Prometheus

Refer to <https://github.com/carlosedp/cluster-monitoring#cluster-monitoring-stack-for-arm--x86-64-platforms> for install prometheus for ARM based platforms


## Grafana

## cert-manager
