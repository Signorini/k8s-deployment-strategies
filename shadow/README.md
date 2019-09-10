---
# Shadow release [Testing high critical services without impact/risk]:
Traffic shadowing is a deployment pattern where production traffic is asynchronously copied to a non-production service for testing. It can be fit for situations to test critical apps, or apps hard to revert like payments gateways is good for persistent services, performance testing and/or behaviour without swift data or control the flow without impact.

![Shadow](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/shadown.png)

---

##  Ways

**Istio** - Using Istio virtual gateway

##  Setup the env

To start, we need a kubernetes cluster with prometheus and grafana installed and running.

**Minikube** - The easiest way to get a kubernetes cluster for develop. [Install minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

**Helm** - It's package manager for Kubernetes, is the easy way to install service on kubernetes. [Install Helm](https://helm.sh/docs/using_helm/#quickstart)

**Prometheus** - Install prometheus using helm template. [Install prometheus](https://github.com/helm/charts/tree/master/stable/prometheus)
```
helm install --name prometheus stable/prometheus
```

**Grafana** - Grafana dashboard. [Install grafana](https://github.com/helm/charts/tree/master/stable/grafana)
```
helm install --name grafana stable/grafana
```

**Istio** - Install default Istio deployment. [Install Istio](https://istio.io/docs/setup/)

### Connected prometheus into grafana

Add prometheus as source on grafana.

### Checking if everything is ok

```
> kubectl get pods

NAME                                             READY   STATUS    RESTARTS   AGE
grafana-58cdc7db6c-46srl                         1/1     Running   3          26d
prometheus-alertmanager-6db559d695-42zf8         2/2     Running   6          26d
prometheus-kube-state-metrics-5db7466b57-5brsq   1/1     Running   6          26d
prometheus-node-exporter-mfwps                   1/1     Running   3          26d
prometheus-pushgateway-59bd88779d-2h9cn          1/1     Running   7          36d
prometheus-server-5f74c4749-bcjnk                2/2     Running   21         36d
```

--- 

### Create a grafana dashboard

You can import grafana dashboard using my template, /grafana/templates.json

To test each deployment I use simple querie to aggregate all nginx requests.

```
sum(irate(nginx_http_requests[5m])) by (version)
```

To count all containers running on kubernetes, use the querie below.

```
sum(kube_pod_container_info{image=~"signorini/nginx-openrest:.*"}) by (image)
```

---

# Istio

## Create first deployment

Create a deployment v1 and v2

```
kubectl apply -f istio/app1.yml

kubectl apply -f istio/app2.yml
```

Expose each application using ClusterIP service.
```
kubectl apply -f istio/expose.yml
```

We have two version running side by side

```
kubectl get deploy
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
cn-v1                           2/2     2            2           27d
cn-v2                           2/2     2            2           27d

kubectl get svc
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                        AGE
my-cn1                          ClusterIP   10.109.115.2     <none>        80/TCP,32111/TCP               1h
my-cn2                          ClusterIP   10.109.115.2     <none>        80/TCP,32111/TCP               1h
```

## Install istio

Istio it's a complete service mesh and proxy system, there are many flavors to install istio, the easiest way is using a helm in a minimal setup.

```
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
    --values install/kubernetes/helm/istio/values-istio-minimal.yaml | kubectl apply -f -
```
[More about Istio installation](https://istio.io/docs/setup/kubernetes/install/helm/)

Checking if everything is work.
```
Verify that the Kubernetes services corresponding to your selected profile have been deployed.

$ kubectl get svc -n istio-system

Ensure the corresponding Kubernetes pods are deployed and have a STATUS of Running:

$ kubectl get pods -n istio-system
```

Now, time to deploy our gateway and virtualservice and point out to version 1.

```
kubectl apply -f istio/gateway.yml
```

This gateway set a proxy to be served using host header value and open port 80.
```
servers:
- port:
    number: 80
    name: http
    protocol: HTTP
    hosts:
    - app.com
```

To set the virtual service to use the previous gateway and point out to my-cn1 clusterip service.
```
kubectl apply -f istio/virtualservice.yml
```

Checking the gateway config we can see the destination parameter.
```
gateways:
    - my-app
  http:
    - route:
        - destination:
            host: my-cn1
```

It's all good. Let's test an application.
```
curl -H "Host: app.com" http://localhost:80/;
```

## Shadowing traffic

To start to mirror the traffic we use a istio feature called mirror, 

```
kubectl apply -f istio/virtualservice-mirror.yml
```

Just add a new config on virtual service, istio will start to mirror the traffic.
```
http:
- route:
    - destination:
        host: my-cn1
    mirror:
        host: my-cn2
```