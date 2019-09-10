---
# A/B release [The best to test new features]:
A/B testing is very similar to a canary release, normally used for business decision-based in statistics and data rather than deployment itself. Technically point of view the implementation is the same as a canary release, a/b release consist of mark a pool of users based on some parameters, tracking using cookies/headers etc and send a specific version, can be two or more per group.

![ab](https://raw.githubusercontent.com/Signorini/k8s-deployment-strategies/master/images/ab.png)

---

##  Ways

**Ingress** - Using nginx ingress

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

**Ingress** - Using Default Ingress setup.

To install a default ingress into kubernetes, simple, run the code below.
```
kubectl apply -f ./ingress/ingress.yml
```

Will deploy all rbac authentication, configs maps, service, pods and ingress rules.

```
kubectl get ingress
NAME         HOSTS     ADDRESS   PORTS   AGE
ingress-ab   app.com             80      26d

kubectl get pods
NAME                                             READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-64ff8db85f-k5x44        1/1     Running   2          27d

kubectl get svc
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                        AGE
exp-nginx                       NodePort    10.97.154.169    <none>        80:31028/TCP,32111:32111/TCP   30d
```


### Connected prometheus into grafana

Add prometheus as source on grafana.

### Checking if everything is ok

```
> kubectl get pods

NAME                                             READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-64ff8db85f-k5x44        1/1     Running   2          27d
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

# Ingress

## Create first deployment

Create a deployment v1

```
kubectl apply -f ingress/app1.yml
```

Create the ingress and point out to app1 deployment.

```
// Setup the ingress rbac and config maps
kubectl apply -f ingress/ingress.yml

// Create a ingress rules to point out to service version 1
kubectl apply -f ingress/ingress-v1.yml

// Expose ingress externaly
kubectl apply -f ingress/expose-ingress.yml
```

Checking ingress rules
```
paths:
- backend:
    serviceName: my-cn1 // version 1
    servicePort: 80 // expose port 80
- backend:
    serviceName: my-cn1
    servicePort: 32111 // expose prometheus metrics.
```

## Do A/B release

To start to switch the traffic, you can deploy one or more rules to split the traffic, in this example we use a custom header
```
kubectl apply -f ingress/ingress-v2.yml
```

Ingress have a built-in a/b test funcationality, can be activate using two annotations
```
nginx.ingress.kubernetes.io/canary-by-header: "X-V2" # canary by custom header
nginx.ingress.kubernetes.io/canary: "true" # enabling canary
```

To test 
```
// hit version 1
curl -H "Host: app.com" http://localhost:31028/;

// hit version 2
curl -H "Host: app.com" -H "X-V2: always" http://localhost:31028/;
```