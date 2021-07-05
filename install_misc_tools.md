# Install essential tools in the k8s cluster

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
